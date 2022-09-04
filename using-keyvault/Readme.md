# Logic Apps e Funções com Key Vault

Neste artigo vamos explorar mais a fundo alguns conceitos da arquitetura utilizada em projetos de Funções e Logic Apps do tipo standard. Vamos explorar o básico de como funciona identidade gerenciada, para que serve o [Key Vault](https://azure.microsoft.com/services/key-vault/) e como podemos usar isso em nossos projetos.

Para total compreensão deste passo a passo, é recomendada a leitura de um artigo anterior: [Logic Apps e Funções no VS Code executados em Containers](https://techcommunity.microsoft.com/t5/desenvolvedores-br/logic-apps-e-fun%C3%A7%C3%B5es-no-vs-code-executados-em-containers/ba-p/3600411) pois ele descreve como fazer o setup do ambiente e do projeto que vamos utilizar hoje.

## O que é o Key Vault?

Segundo a [documentação oficial](https://docs.microsoft.com/azure/key-vault/general/basic-concepts):

> O Azure Key Vault é serviço de nuvem para armazenar e acessar segredos de maneira segura.

> Um segredo é qualquer coisa a qual você queira controlar rigidamente o acesso, como chaves de API, senhas, certificados ou chaves criptográficas.

Desta maneira podemos entender que ele nos proporciona segurança durente todo o processo desenvolvimento e entrega de software, permitindo que só as pessoas responsáveis por um segredo consigam acesso a ele, sem riscos de vazamento de informações.

## O que é Identidade Gerenciada?

Identidades gerenciadas representam uma maneira segura de acesso a aplicações e recursos do Azure, utilizando o [Azure Active Directory](https://azure.microsoft.com/services/active-directory) e ela terá um papel fundamental em nossa aplicação, que será controlar nosso acesso ao Key Vault através de nossa aplicação.

Quando investigamos a [documentação](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview) vemos que elas podem ser de dois tipos:

- Atribuída pelo sistema
- Atribuída pelo usuário

Neste exemlpo vamos criar uma Identidade Gerenciada que será atribuída à nossa aplicação, depois disso, vamos configurar as políticas de acesso de nosso Key Vault para premitir que nossa recém criada Identidade possa ler os segredos salvos.

## Configurando nossa Aplicação

Se você ainda não deu uma olhada no artigo [Logic Apps e Funções no VS Code executados em Containers](https://techcommunity.microsoft.com/t5/desenvolvedores-br/logic-apps-e-fun%C3%A7%C3%B5es-no-vs-code-executados-em-containers/ba-p/3600411), essa é a hora. Vamos utilizar os mesmos passos descritos nos capítulos `Setup do ambiente` e `Setup do projeto`.

Agora vamos integrar o Key Vault ao nosso projeto!

O primeiro passo e criar um novo Key Vault em nosso grupo de recursos.

```azurecli
az keyvault create --name kv-lab --resource-group rg-lab --location eastus2
```
