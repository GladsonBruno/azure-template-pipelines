## Manual template Node
O template Maven cobre os seguintes pontos:

* Versionamento automático

* Qualidade de código com Sonar Cloud

* Testes unitários

* Code Coverage

* Build do binário da aplicação e geração de container Docker

* Scan de vulnerabilidades no container Docker gerado

Um exemplo de utilização pode ser visto neste [link](https://github.com/GladsonBruno/Node-AzureDevOps-CI-Example)

## Pré requisitos
* Possuir o Azure DevOps devidamente configurado conforme recomendado neste [manual](../README.md).

* Possuir pelo menos um teste unitário implementado ou vazio. ( Caso queira ter as informações de Code Coverage no Sonar e Azure. )

* Utilizar o [Jest](https://jestjs.io/pt-BR/) como framework de testes.

* Criar as classes de teste com a extensão **.spec.ts**

* Utilizar o pacote **jest-sonar-reporter** para gerar os reports esperados pelo Sonar no formato **lcov.info**

* Possuir implementados os script **test** e **test:sonar** no **package.json** da aplicação.

* Possuir o Jest devidamente configurado para gerar os reports **lcov.info**(Report de Code Coverage esperado pelo SonarCloud) e **coverage.xml**(Report de Code Coverage esperado pelo Azure DevOps).

## Configuração da aplicação prévia a utilização do template Node
### Pacotes NPM recomendados para utilizar na aplicação:
```
npm install --save-dev jest
npm install --save-dev ts-jest
npm install --save-dev @types/jest
npm install --save-dev jest-sonar-reporter
```

### Scripts a serem configurados no arquivo package.json:
```
# Configuração esperada do script 'test'
"test": "jest --coverage --coverageReporters=cobertura"

# Configuração esperada do script 'test:sonar'
"test:sonar": "jest --coverage"
```

### Exemplo de configuração esperada do Jest:
```
  "jest": {
    "testResultsProcessor": "jest-sonar-reporter",
    "moduleFileExtensions": [
      "js",
      "json",
      "ts"
    ],
    "rootDir": "src",
    "testRegex": ".*\\.spec\\.ts$",
    "transform": {
      "^.+\\.(t|j)s$": "ts-jest"
    },
    "collectCoverageFrom": [
      "**/*.(t|j)s"
    ],
    "coverageDirectory": "../coverage",
    "testEnvironment": "node"
  },
  "jestSonar": {
    "reportPath": "coverage",
    "reportFile": "test-reporter.xml",
    "indent": 4
  }
```
Para mais detalhes consulte este [link](https://github.com/GladsonBruno/Node-AzureDevOps-CI-Example/blob/master/package.json) para ver um exemplo completo.

### Validando a configuração da aplicação
Após realizar as configurações dos tópicos anteriores em sua aplicação execute os seguintes comandos para validar se a mesma foi devidamente configurada:
```
npm run test
npm run test:sonar
```

Caso os 2 comandos estejam sendo executados com sucesso é sinal que sua aplicação foi devidamente configurada e que podemos seguir para os próximos passos.

## Utilização do template

Após configurar seu projeto com os pré-requisitos especificados no tópico anterior crie um arquivo chamado **azure-pipeline.yml** com o conteúdo deste [link](https://github.com/GladsonBruno/Node-AzureDevOps-CI-Example/blob/master/azure-pipeline.yml).

Em seguida observe o seguinte trecho de seu arquivo de pipeline:
```
resources:
  repositories:
  - repository: templates
    type: github
    name: GladsonBruno/azure-template-pipelines
    endpoint: GladsonBruno
    ref: 'refs/heads/main'
```


Atualize o parâmetro **name** para conter o seu projeto forkeado através deste projeto base. O formato esperado é o seguinte: **GitHub_Username/repository-name**. Exemplo: **GladsonBruno/azure-template-pipelines**.

Em seguida atualize o parâmetro **endpoint** para conter o nome de seu Service Connection que realiza a interface entre o Azure DevOps e o GitHub.


Além disso é necessário definir os seguintes parâmetros ao realizar a extensão em cima do template:
* **CONTAINER_REGISTRY_SERVICE_CONNECTION_NAME**: Nome do Service Connection que faz a interface entre Azure e o Registry Docker alvo configurado no Azure DevOps.

* **DOCKER_REGISTRY**: Informações referente ao Registry para correto tageamento da imagem Docker.

* **SONAR_CLOUD_SERVICE_CONNECTION_NAME**: Nome do Service Connection que faz a interface entre Azure e o Sonar Cloud alvo configurado no Azure DevOps.

* **SONAR_CLOUD_ORGANIZATION**: Nome de sua organization dentro do SonarCloud.

Exemplo de uso de parâmetros em uma pipeline extendida de um template:
```
extends:
  template: /node/azure-pipeline.yml@templates
  parameters:
    CONTAINER_REGISTRY_SERVICE_CONNECTION_NAME: MyContainerRegistryServiceConnection
    DOCKER_REGISTRY: MyRegistry
    SONAR_CLOUD_SERVICE_CONNECTION_NAME: MySonarClodServiceConnection
    SONAR_CLOUD_ORGANIZATION: MySonarCloudOrganization
```


Após isso crie um variable group no Azure DevOps contendo as seguinte variáveis:

* **COVERAGE_REPORT_PATH**: Informa ao CI onde encontrar o arquivo coverage.xml que irá conter as informações de Code Coverage esperadas pela pipeline. O valor que uso normalmente é **\*\*/\*coverage.xml**

* **IMAGE_NAME**: Nome que será atribuído ao seu container Docker. Recomendo nomear o container com letras minúsculas separadas por **-**. Exemplo: **demo-api**

* **NODEJS_VERSION**: Informa a pipeline a versão do Node utilizado pela aplicação. Exemplo: **14**, **16**.

* **SONAR_EXCLUSIONS**: Informa ao Sonar Cloud quais diretórios e arquivos serão excluídos da análise de qualidade. O valor que uso normalmente é: **\*\*/\*.spec.ts**

* **SONAR_INCLUSIONS**: Informa ao Sonar Cloud quais diretórios e arquivos serão incluídos da análise de qualidade. O valor que uso normalmente é: **\*\*/*.ts,\*\*/\*.js**

* **SONAR_LANGUAGE**: Informa ao Sonar Cloud a linguagem que está sendo analidada. Use o valor **js**

* **SONAR_PATH_SOURCES**: Informa ao Sonar Cloud o path no qual está contido seu código. Use o valor **./src**.

* **SONAR_PROJECT_KEY**: Chave do projeto criada no Sonar Cloud.

* **SONAR_PROJECT_NAME**: Nome do projeto criado no Sonar Cloud.

* **SONAR_REPORT_PATH**: Informa ao Sonar Cloud onde encontrar o report **lcov.info** que irá conter as informações de Code Coverage da aplicação. Use o valor **coverage/lcov.info** neste parâmetro.

Com o variable group criado insira o nome dele na pipeline substituindo o variable group **node-ci-example**.

Exemplo do arquivo de pipeline após finalizar a configuração:
```
trigger:
  branches:
    include:
      - master
      - develop

variables:
  - group: node-ci-example

resources:
  repositories:
  - repository: templates
    type: github
    name: GladsonBruno/azure-template-pipelines
    endpoint: GladsonBruno
    ref: 'refs/heads/main'

extends:
  template: /node/azure-pipeline.yml@templates
  parameters:
    CONTAINER_REGISTRY_SERVICE_CONNECTION_NAME: MyContainerRegistryServiceConnection
    DOCKER_REGISTRY: MyRegistry
    SONAR_CLOUD_SERVICE_CONNECTION_NAME: MySonarClodServiceConnection
    SONAR_CLOUD_ORGANIZATION: MySonarCloudOrganization
```