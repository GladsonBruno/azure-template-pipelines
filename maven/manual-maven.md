## Manual template Maven
O template Maven cobre os seguintes pontos:
* Versionamento automático

* Qualidade de código com Sonar Cloud

* Testes unitários

* Code Coverage

* Build do binário da aplicação e geração de container Docker

* Scan de vulnerabilidades no container Docker gerado

Um exemplo de utilização pode ser visto neste [link](https://github.com/GladsonBruno/SpringBoot-AzureDevOps-CI-Example/blob/master/azure-pipeline.yml)

### Pré requisitos
* Possui o plugin do Jacoco configurado na aplicação.
* Possuir pelo menos um teste unitário implementado ou vazio. ( Caso queira ter as informações de Code Coverage no Sonar e Azure. )

### Utilização do template

Para utilizar o template é necessário possuir um variable group com as seguintes variáveis cadastradas:
* **IMAGE_NAME**: Nome que será atribuído ao seu container Docker.

* **JACOCO_REPORT_PATH** Pasta no qual o report de code coverage do Jacoco será gerado. Por padrão o valor desta variável é: **/jacoco.xml

* **JAVA_JDK_VERSION**: Informa a pipeline a versão do JDK utilizado pela aplicação. Exemplo: 11, 17.

* **SONAR_EXCLUSIONS**: Informa ao Sonar Cloud quais diretórios e arquivos serão excluídos da análise de qualidade. O valor que uso normalmente é: src/test/*.java,src/main/resources/application.properties

* **SONAR_INCLUSIONS**: Informa ao Sonar Cloud quais diretórios e arquivos serão incluídos da análise de qualidade. O valor que uso normalmente é: **/*.java

* **SONAR_PROJECT_KEY**: Chave do projeto criada no Sonar Cloud.

* **SONAR_PROJECT_NAME**: Nome do projeto criado no Sonar Cloud.


Além disso é necessário definir os seguintes parâmetros ao realizar a extensão em cima do template:
* **CONTAINER_REGISTRY_SERVICE_CONNECTION_NAME**: Nome do Service Connection que faz a interface entre Azure e o Registry Docker alvo configurado no Azure DevOps.

* **DOCKER_REGISTRY**: Informações referente ao Registry para correto tageamento da imagem Docker.

* **SONAR_CLOUD_SERVICE_CONNECTION_NAME**: Nome do Service Connection que faz a interface entre Azure e o Sonar Cloud alvo configurado no Azure DevOps.

* **SONAR_CLOUD_ORGANIZATION**: Nome de sua organization dentro do SonarCloud.

Exemplo de uso de parâmetros em uma pipeline extendida de um template:
```
extends:
  template: /maven/azure-pipeline.yml@templates
  parameters:
    CONTAINER_REGISTRY_SERVICE_CONNECTION_NAME: MyContainerRegistryServiceConnection
    DOCKER_REGISTRY: MyRegistry
    SONAR_CLOUD_SERVICE_CONNECTION_NAME: MySonarClodServiceConnection
    SONAR_CLOUD_ORGANIZATION: MySonarCloudOrganization
```