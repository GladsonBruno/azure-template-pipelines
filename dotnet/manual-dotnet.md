## Manual template Dotnet
O template Dotnet cobre os seguintes pontos:

* Versionamento automático

* Qualidade de código com Sonar Cloud

* Testes unitários

* Code Coverage

* Build do binário da aplicação e geração de container Docker

* Scan de vulnerabilidades no container Docker gerado

Um exemplo de utilização pode ser visto neste [link](https://github.com/GladsonBruno/aspnet-azure-devops-ci-example)

## Pré requisitos
* Possuir o Azure DevOps devidamente configurado conforme recomendado neste [manual](../README.md).

* O build da aplicação deve ocorrer fora do Dockerfile.

* Possuir pelo menos um teste unitário implementado ou vazio. ( Caso queira ter as informações de Code Coverage no Sonar e Azure. )

* Possuir um arquivo **coverlet.runsettings.xml** criado na raiz do projeto.

## Configuração da aplicação prévia a utilização do template Dotnet
### Criando o arquivo coverlet.runsettings.xml
Crie o arquivo **coverlet.runsettings.xml** na pasta raiz de seu projeto com o seguinte conteúdo:
```xml
<?xml version="1.0" encoding="utf-8" ?>
<RunSettings>
    <DataCollectionRunSettings>
        <DataCollectors>
            <DataCollector friendlyName="XPlat Code Coverage">
                <Configuration>
                    <Format>opencover</Format>                    
                </Configuration>
            </DataCollector>
        </DataCollectors>
    </DataCollectionRunSettings>
</RunSettings>
```

### Validando a configuração da aplicação
Após realizar as configurações dos tópicos anteriores em sua aplicação execute os seguintes comandos para validar se a mesma foi devidamente configurada:

**OBSERVAÇÃO IMPORTANTE**: Substitua **./aspnet-demo-api.sln** pelo caminho no qual se encontra o arquivo **.sln** de seu projeto.

```
dotnet restore
dotnet test ./aspnet-demo-api.sln --settings ./coverlet.runsettings.xml --logger trx /p:CollectCoverage=true /p:CoverletOutputFormat=opencover
```

Caso o comando de testes seja executado com sucesso é sinal que sua aplicação foi devidamente configurada e que podemos seguir para os próximos passos.

## Utilização do template

Após configurar seu projeto com os pré-requisitos especificados no tópico anterior crie um arquivo chamado **azure-pipeline.yml** com o conteúdo deste [link](https://github.com/GladsonBruno/aspnet-azure-devops-ci-example/blob/master/azure-pipeline.yml).

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
  template: /dotnet/azure-pipeline.yml@templates
  parameters:
    CONTAINER_REGISTRY_SERVICE_CONNECTION_NAME: MyContainerRegistryServiceConnection
    DOCKER_REGISTRY: MyRegistry
    SONAR_CLOUD_SERVICE_CONNECTION_NAME: MySonarClodServiceConnection
    SONAR_CLOUD_ORGANIZATION: MySonarCloudOrganization
```

Após isso crie um variable group no Azure DevOps contendo as seguinte variáveis:

* **BUILD_OPTIONS** Informa ao CI as opções de build a serem utilizas no comando **dotnet build**. Exemplo: **-c Release -o ./app/build**

* **BUILD_PATH**: Informa ao CI o path do arquivo **.csproj** da solução pricipal de seu projeto. Exemplo: **$(Build.SourcesDirectory)/aspnet-demo-api/aspnet-demo-api.csproj**

* **DOTNET_SDK_VERSION**: Informa a pipeline a versão do AspNet/Dotnet utilizado pela aplicação. Exemplo: **3.x**, **6.x**.

* **IMAGE_NAME**: Nome que será atribuído ao seu container Docker. Recomendo nomear o container com letras minúsculas separadas por **-**. Exemplo: **demo-api**

* **OPENCOVER_REPORT_PATH**: Informa ao Sonar Cloud onde encontrar o arquivo de cobertura de testes de sua aplicação. O valor que uso normalmente é: **$(Agent.TempDirectory)/\*\*/coverage.opencover.xml**

* **OPENCOVER_VSTEST_REPORT_PATH**: Informa ao Sonar Cloud onde encontrar o arquivo **.trx** que irá conter os resultados dos testes de sua aplicação. O valor que uso normalmente é: **$(Agent.TempDirectory)/\*\*/*.trx**

* **PUBLISH_OPTIONS**: Informa ao CI as opções de build a serem utilizas no comando **dotnet publish**. Exemplo: **-c Release -o ./app/publish /p:UseAppHost=false**

* **SOLUTIONFILE_PATH**: Informa ao CI o path do arquivo **.sln** da solução pricipal de seu projeto. Exemplo: **$(Build.SourcesDirectory)/aspnet-demo-api.sln**

* **SONAR_EXCLUSIONS**: Informa ao Sonar Cloud quais diretórios e arquivos serão excluídos da análise de qualidade. O valor que uso normalmente é: **\*\*/obj/\*\*,\*\*/\*.dll,\*\*/\*Test.cs**

* **SONAR_PROJECT_KEY**: Chave do projeto criada no Sonar Cloud.

* **SONAR_PROJECT_NAME**: Nome do projeto criado no Sonar Cloud.

* **TEST_OPTIONS**: Informa ao CI as opções de build a serem utilizas no comando **dotnet test**. Exemplo: **--settings ./coverlet.runsettings.xml --logger trx /p:CollectCoverage=true /p:CoverletOutputFormat=opencover**

Com o variable group criado insira o nome dele na pipeline substituindo o variable group **node-ci-example**.

Exemplo do arquivo de pipeline após finalizar a configuração:
```
trigger:
  branches:
    include:
      - master
      - develop

variables:
  - group: dotnet-ci-example

resources:
  repositories:
  - repository: templates
    type: github
    name: GladsonBruno/azure-template-pipelines
    endpoint: GladsonBruno
    ref: 'refs/heads/main'

extends:
  template: /dotnet/azure-pipeline.yml@templates
  parameters:
    CONTAINER_REGISTRY_SERVICE_CONNECTION_NAME: MyContainerRegistryServiceConnection
    DOCKER_REGISTRY: MyRegistry
    SONAR_CLOUD_SERVICE_CONNECTION_NAME: MySonarClodServiceConnection
    SONAR_CLOUD_ORGANIZATION: MySonarCloudOrganization
```