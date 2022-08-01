# Logic Apps e Funções no VS Code executados em Containers

Continuando nossa [jornada](https://techcommunity.microsoft.com/t5/desenvolvedores-br/introdu%C3%A7%C3%A3o-aos-logic-apps/ba-p/3263990) de [Logic Apps](https://azure.microsoft.com/services/logic-apps/), hoje iremos criar um novo projeto usando [Visual Studio Code](https://code.visualstudio.com/), executá-lo em um container usando [Docker](https://www.docker.com/).

Isso se torna possível com o pacote de extensões do Azure Functions, que nos permite adicionar e executar workflows de logic apps em conjunto com funções, desde que o runtime selecionado seja o node.

Lembrando que também é possivel utilizar o [Visual Studio](https://visualstudio.microsoft.com/downloads/), porém existe uma limitação em relação a versão do produto.

> A extensão Aplicativos Lógicos do Azure não está disponível no momento para Visual Studio 2022.

Segundo a [documentação](https://docs.microsoft.com/azure/logic-apps/manage-logic-apps-with-visual-studio) esta opção é disponível nas versões 2019, 2017 ou 2015 – Community Edition ou superior.

Por esta extensão não estar disponível no momento neste exemplo vamos seguir apenas com o VS Code.

Depois de acompanhar este artigo, você aprenderá a criar um novo projeto de logic apps e/ou funções, executá-lo em containers mantendo as chaves de acesso (para as funções) e o token SAS (para os logic apps).

## Setup do ambiente

1. No menu de extensões do VS Code, procure por Logic Apps. Você encontrará duas opções [Azure Logic Apps (Consumption)](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-logicapps) e [Azure Logic Apps (Standard)](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurelogicapps). Para este passo a passo vamos usar apenas o Standard pois ele nos permite criar e rodar nossos fluxos de forma local;

2. Instale também o [Azure Functions Core Tool](https://docs.microsoft.com/azure/azure-functions/functions-run-local), rodando o comando ```winget install Microsoft.AzureFunctionsCoreTools -v 3.0.3904``` no terminal. Quando a instalação estiver finalizada, será necessário fechar e reabrir o VS Code;

3. Certifique-se de que você tenha o Docker instalado e rodando em sua máquina. Caso precise instalar, siga este [passo-a-passo](https://docs.docker.com/desktop/install/windows-install/).

Opcionais:

1. Para desenvolvimento local, podemos usar o [Azure Storage Emulator](https://go.microsoft.com/fwlink/?linkid=717179&clcid=0x409). Quando ela estiver instalado, um atalho terá sido adicionado ao seu menu de início do Windows, execute o atalho e o setup estará concluído. Lembrando que este item só é necessário se na execução local você optar por usar a opção de storage de desenvolvimento. Em seu **local.settings.json** configurar ```"AzureWebJobsStorage": "UseDevelopmentStorage=true"```;

2. Se você quiser rodar seus fluxos localmente, também será necessário o [Node.js](https://nodejs.org/download/release/v14.20.0/) versão 12 ou 14.

## Setup do projeto

Inicamos clonando o código deste [repositório](https://github.com/jdipolit/LogicApps).

Este fluxo expõe uma API que quando chamada insere o corpo da requisição em um banco cosmos e depois seleciona os 10 primeiros registros da base.

Para que este fluxo funcione, vamos criar alguns recursos no Azure.

Vamos abrir o CLI do Azure e iniciamos criando um novo grupo de recursos.

```azurecli
az group create --name rg-lab --location eastus2
```

Agora vamos criar nossa storage account e obter sua string de conexão

```azurecli
az storage account create --name stmeulab --resource-group rg-lab --location eastus2 --sku Standard_LRS 

az storage account show-connection-string --resource-group rg-lab --name stmeulab
```

Guarde essa string de conexão, pois vamos utilizar ela daqui a pouco.

Por último vamos criar nosso banco, e obter a string de conexão.

```azurecli
az cosmosdb create --name dbmeulab --resource-group rg-lab

az cosmosdb sql database create --name coisas --resource-group rg-lab --account-name dbmeulab

az cosmosdb sql container create --resource-group rg-lab --account-name dbmeulab --database-name coisas --name itens --partition-key-path "/id"

az cosmosdb keys list --name dbmeulab --resource-group rg-lab --type connection-strings
```

Agora vamos alterar nosso projeto para usar nossas strings de conexão do banco e do storage account.

Em **local.settings.json** vamos alterar os valores das chaves para algo assim:

```json
{
  "IsEncrypted": false,
  "Values": {
    "FUNCTIONS_WORKER_RUNTIME": "node",
    "AzureWebJobsStorage": "string de conexão da storage account",
    "AzureCosmosDB_connectionString": "string de conexão do banco"
  }
}
```

Agora vamos alterar as mesmas configurações no arquivo **Dockerfile**.

```dockerfile
ENV AzureWebJobsStorage="string de conexão da storage account"
ENV AzureCosmosDB_connectionString="string de conexão do banco"
```

## Navegando e editando os fluxos

Olhano nos arquivos do projeto vamos encontrar três pastas, uma função chamada 'ola' e dois logic apps chamados ***inserir*** e ***listar***.

<!-- Abra a paleta de comandos (View > Command Palette...) ou pressione ```ctrl+shift+p``` e digite "logic apps new" e selecione a opção "Azure Logic Apps: Create New Project..."

Selecione uma pasta para armazenar seu projeto.

Selecione a opção "Stateful Workflow" e depois "Open in current windows".
(Adicionar diferenças)

Dê um nome ao fluxo criado, para este exemplo usarei ***MeuPrimeiroFluxo***.
 -->

No explorador de arquivos, em seu projeto expanda a pasta ***inserir*** ou ***listar*** e clique com o botão direito em ```workflow.json```. Selecione "Open in Designer".

Na primeria vez que você faz isso, o VS Code vai solicitar algumas informações. Se estivessemos trabalhando com um logic app hospedado no Azure, precisaríamos configurar algumas coisas, mas neste exemplo vamos fazer tudo de forma local então selecione "Skip for now".

Nesta interface gráfica você pode alterar seu fluxo exatamente como no portal do Azure.

## Executando seu fluxo localmente

Agora é só apertar F5 e Azure Function Core Tools vai executar seu fluxo.

Para podermos obter a url de chamada local, assim como analizar as chamadas, devemos ir até o explorador de arquivos, na pasta ***inserir*** ou ***listar*** e clique com o botão direito em ```workflow.json```. Selecione "Overivew" e copie a url apresentada.

Para testar, vamos abrir o [Postman](https://www.postman.com/) e fazer uma chamada, para este exemplo use o método POST, com a url que você acabou de copiar. Envie a mensagem.

Se quiser, você pode analizar o processamento do Logic App na página overview clicando no identificador da chamada.

Você também pode debugar a execução adicionando break points no arquivo ```workflow.json``` e acompanhar o valor das variaveis na aba "Run and Debug" do VS Code.

## Usando a API de gerenciamento

Você iniciar seu projeto a qualquer instante sem o VS Code, basta navegar até a pasta do projeto no terminal e executar o comando ```func host start```. Este comando inicia o host do Functions em seu ambiente local e toma como base a pasta atual.

Enquanto o host do Functions estiver rodando você pode acessar alguns métodos da API de gerenciamento do seu logic app. Aqui estão alguns exemplos:

- Histórico de execuções

    ```GET http://localhost:7071/runtime/webhooks/workflow/api/management/workflows/inserir/runs```

- Histórico de versões publicadas

    ```GET http://localhost:7071/runtime/webhooks/workflow/api/management/workflows/listar/versions```

- Código fonte de uma versão específica

    ```GET http://localhost:7071/runtime/webhooks/workflow/api/management/workflows/inserir/versions/08585447452426391644```

- Nomes dos gatilhos dispoíveis para o fluxo

    ```GET http://localhost:7071/runtime/webhooks/workflow/api/management/workflows/listar/triggers```

- Obter a Url de chamada por gatilho

    ```POST http://localhost:7071/runtime/webhooks/workflow/api/management/workflows/inserir/triggers/manual/listCallbackUrl```

## Usando Docker

Um ponto muito importante para conseguirmos rodar nossos logic apps de forma eficiente em containers é entender que na verdade estamos rodando um projeto de funções, com seu próprio runtime e que a definição dos workflows de logic apps estão sendo executadas através do pacote de extesão criado para o runtime das funções.

Então nosso projeto e as APIs de gerenciamento serão as mesmas que de um projeto exclusivo de funções.

Observe que neste projeto temos uma função e um logic app que vão rodar dentro da mesma API, usando as mesmas chaves de acesso (pois elas se aplicam ao projeto).

Vale resaltar que só é possivel rodar funções e logic apps em conjunto quando o runtime do projeto definido na configuração **FUNCTIONS_WORKER_RUNTIME** for igual a **node**. Sem essa definição, seus logic apps não serão executados.

Normalmente a chave mestra de acesso as APIs seria criada e armazenada dentro da storage account definida pelo projeto. Porém podemos manipular esta mecânica alterando o valor da configuração ```AzureWebJobsScriptRoot``` dentro do nosso **Dockerfile**.

Note que nosso projeto contém um arquivo chamado host_secrets.json e que nosso Dockerfile manipula este arquivo. Desta maneira conseguimos fixar e gerenciar nossos segredos.

Outro ponto importante é entender como podemos manter nosso token SAS que dará acesso a cada logic app quando estamos rodando um container. A geração do SAS está diretamente ligada a chave mestra do projeto e o hostname da máquina. Então se nossa chave ou nossa máquina tiver um nome variável, toda vez que um novo container for executado, o SAS terá sido alterado e isso se torna um grande problema para os clientes que consomem nossa API.

Como já resolvemos o problema de gerenciamento da chave mestra para nosso projeto, agora só precisamos resolver o problema do hostname do container. Isso será feito com um argumento na hora de executar o container.

Vamos iniciar o cmd e navegar até a pasta do projeto. Dentro dela vamos rodar os seguintes comandos para compilação e execução do container.

```cli
docker build . --tag fn-node:latest

docker run --name logicapp -h logicapp -it --rm -p 7071:80 fn-node
```

Estes comandos criam uma imagem chamada fn-node a partir da definição do Dockerfile do projeto e depois executam esta imagem em um novo container, expondo a porta 7071 e configurando um hostname fixo de 'logicapp' para  o container.

Agora conseguimos acessar nossas APIs através de chamadas REST.

Para chamar a função dentro do projeto vamos fazer a seguite chamada:

```
GET 
http://localhost:7071/api/ola?code=asGmO6TCW/t42krL9CljNod3uG9aji4mJsQ7==

{
    "name": "julio"
}
```

Observe o parâmetro code, onde passo a chave mestra do projeto definida no arquivo host_secrets.json. Se o nível de acesso da função criada for "function" você também pode usar a chave "functionKeys" definida no mesmo arquivo.

Para chamar seu logic app primeiro vamos conferir o token SAS criado para o método ***inserir*** no seu container.

Vamos fazer a chamada:

```POST
POST 
http://localhost:7071/runtime/webhooks/workflow/api/management/workflows/inserir/triggers/manual/listCallbackUrl?code=asGmO6TCW/t42krL9CljNod3uG9aji4mJsQ7==
```

Ao contrário da execução local, rodando no container sempre é necessário passar a chave mestra da API para acessar todos os métodos de gerenciamento do seu projeto.

A resposta desta chamada vai te dar a url completa para chamada do seu workflow.

Como só expusemos a porta 80 em nosso projeto será necessário alterar a url de https para http e a porta para 7071.

Teste o fluxo ***inserir***. Este workflow aceita qualquer corpo passado na mensagem e o insere no seu banco cosmos.

Exemplo de corpo usando Postman:

```JSON
{
    "id": "{{$guid}}",
    "test": "your random number is {{$randomInt}}"
}
```

Teste também o fluxo ***listar***, que vai te retornar os 10 últimos registros da base, ordenados por id (não se esqueça de obter o token SAS antes de chamar o método pois ele será diferente do primeiro).

E pronto! Agora você pode containerizar seus projetos de logic apps e funções.

E se não for mais usar os recuros no Azure, não se esqueça de deletar o grupo de recursos

```azurecli
az group delete --name rg-lab
```

## Conclusão

Conseguimos criar um novo projeto utilizando o VS Code, entendemos suas extensões, dependências e como podemos testar nossos fluxos. Além disso, conseguimos compilar e executar nossa aplicação em container, executando com Docker e utilizando a API de gerenciamento das funções.

## Próximos passos

Em artigos futuros vamos abordar:

- Escolhendo entre os ambientes de hospedagem dos Logic Apps;
- Conexão de fluxos no Azure com endpoints on-premisses;
- Construção de conectores personalizados;
- Logic Apps com APIM;
- Estratégias de troubleshooting para Logic Apps;
- Integration Service Environment (ISE).
