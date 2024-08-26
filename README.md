
### Estrutura Geral

Este exemplo de AWS Step Function usa um estado do tipo `Map` para processar múltiplos itens, fazendo requisições POST para uma API e tratando erros com retry e catch. Aqui está um detalhamento das principais seções:

1. **Comment**: Uma descrição geral sobre o que o Step Function faz.

2. **StartAt**: Define o estado inicial da máquina de estados, que é o `MapState`.

### Estados

1. **MapState**:
   - **Type**: `Map` - Indica que este estado é um estado `Map`, que é usado para processar uma coleção de itens em paralelo.
   - **ItemProcessor**: Configura como os itens são processados.
     - **ProcessorConfig**:
       - **Mode**: `INLINE` - Indica que a configuração do processador é feita inline (diretamente no estado).
     - **StartAt**: `Invoke API` - Define o estado inicial para o processador de itens dentro do `MapState`.
     - **States**:
       - **Invoke API**:
         - **Type**: `Task` - Indica que este estado realiza uma tarefa, que no caso é invocar uma API.
         - **Resource**: `arn:aws:states:::apigateway:invoke` - Identifica o recurso a ser chamado, que é o serviço API Gateway.
         - **Parameters**:
           - **ApiEndpoint**: `<ID>.execute-api.us-east-2.amazonaws.com` - Endpoint da API Gateway onde a requisição POST será enviada.
           - **Path**: `/dev/pets` - Caminho da API para a requisição.
           - **Method**: `POST` - Método HTTP para a requisição.
           - **RequestBody.$**: `$.item` - O corpo da requisição é baseado no item atual do `MapState`.
           - **AuthType**: `NO_AUTH` - Indica que não há autenticação necessária para esta chamada de API.
         - **Retry**:
           - Configurações de retry para lidar com falhas:
             - **ErrorEquals**: `States.ALL` - Aplica o retry para todos os erros.
             - **IntervalSeconds**: 2 - Intervalo entre tentativas de retry.
             - **MaxAttempts**: 3 - Número máximo de tentativas de retry.
             - **BackoffRate**: 2 - Taxa de aumento do intervalo entre tentativas de retry.
         - **End**: true - Indica que este é o estado final no processador de itens.

   - **ItemsPath**: `$.itens` - Define o caminho para a lista de itens a ser processada pelo estado `MapState`.
   - **ResultPath**: `$.resultados` - Define onde armazenar os resultados do processamento dos itens.
   - **MaxConcurrency**: 3 - Define o número máximo de itens a serem processados em paralelo.
   - **Catch**:
     - Configurações para tratar erros que não foram resolvidos pelos retries:
       - **ErrorEquals**: `States.ALL` - Captura todos os erros.
       - **Next**: `FailState` - Redireciona para o estado `FailState` em caso de erro.

2. **FailState**:
   - **Type**: `Fail` - Indica que este estado termina a execução com falha.
   - **Error**: `MapStateError` - Nome do erro que ocorreu.
   - **Cause**: `Ocorreu um erro durante o processamento do MapState.` - Mensagem de causa do erro.

### Resumo

Este Step Function configura uma execução que processa itens em paralelo, fazendo requisições POST para uma API. Ele trata falhas de forma robusta com tentativas de retry e captura qualquer erro que não tenha sido resolvido pelos retries, redirecionando para um estado de falha apropriado.

```json
{
  "itens": [
    {
      "item": {
        "id": 1,
        "nome": "Produto A",
        "preco": 100,
        "quantidade": 2
      }
    },
    {
      "item": {
        "id": 2,
        "nome": "Produto B",
        "preco": 200,
        "quantidade": 1
      }
    },
    {
      "item": {
        "id": 3,
        "nome": "Produto C",
        "preco": 150,
        "quantidade": 3
      }
    }
  ]
}
```

