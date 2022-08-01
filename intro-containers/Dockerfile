FROM mcr.microsoft.com/azure-functions/node:4-node16 as base
WORKDIR /home/site/wwwroot
EXPOSE 80

#Adicionando host e connections
COPY host.json .
COPY connections.json .

#Adicionando uma função
ADD /ola/function.json /home/site/wwwroot/ola/function.json
ADD /ola/index.js /home/site/wwwroot/ola/index.js

#Adicionando os Logic Apps
ADD /inserir/workflow.json /home/site/wwwroot/inserir/workflow.json
ADD /listar/workflow.json /home/site/wwwroot/listar/workflow.json

#Garantindo que a chave mestra do projeto se mantenha sempre a mesma
ADD host_secrets.json /home/site/wwwroot/etc/secrets/host.json
ENV FUNCTIONS_SECRETS_PATH=/home/site/wwwroot/etc/secrets

#Configurações básicas
ENV AzureWebJobsSecretStorageType=Files
ENV AzureWebJobsScriptRoot=/home/site/wwwroot
ENV AzureFunctionsJobHost__Logging__Console__IsEnabled=true
ENV AzureWebJobsStorage=""

#Configuração usada para o logic app (referenciada em connections.json)
ENV AzureCosmosDB_connectionString=""

#Configuração obrigatória para rodar um Logic App
ENV WEBSITE_HOSTNAME=localhost