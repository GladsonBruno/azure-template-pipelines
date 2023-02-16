## Manual template Node
O template Python cobre os seguintes pontos:

* Versionamento automático

* Qualidade de código com Sonar Cloud

* Testes unitários

* Code Coverage

* Build do binário da aplicação e geração de container Docker

* Scan de vulnerabilidades no container Docker gerado

Um exemplo de utilização pode ser visto neste [link](https://github.com/GladsonBruno/Python-AzureDevOps-CI-Example)

## Pré requisitos
* Possuir o Azure DevOps devidamente configurado conforme recomendado neste [manual](../README.md).

* O build da aplicação deve ocorrer dentro do Dockerfile.

* Possuir pelo menos um teste unitário implementado. ( Caso queira ter as informações de Code Coverage no Sonar e Azure. )

* Utilizar o [Pytest](https://docs.pytest.org/en/7.2.x/) como framework de testes.

* Criar as classes de teste com o seguinte padrão de nomenclatura: **test_\*.py** ou **\*_test.py**

* Utilizar os pacotes **pytest-cov** e **pytest-azurepipelines** em seu arquivo **requirements.txt** para que o CI gere o report de cobertura de testes adequadamente.


## Configuração da aplicação prévia a utilização do template Python
### Pacotes Python que recomendados inserir em seu arquivo requirements.txt:
Insira os seguintes pacotes em seu arquivo **requirements.txt**:
```
pytest==7.2.0
pytest-cov==4.0.0
pytest-azurepipelines==1.0.4
```

E em seguida execute o seguinte comando para realizar a instalação destes pacotes:
```
pip install -r requirements.txt
```

### Configurando o arquivo pyproject.toml
Crie um arquivo chamado **pyproject.toml** e o configure com o seguinte conteúdo:
```
[tool.pytest.ini_options]
pythonpath = [
  ".", "app", 
]
testpaths = ["tests"]
python_files = ["test_*.py", "*_test.py"]
```

Esta configuração indica o seguinte:
* Seu código fonte está contido na pasta **app**.
* Seus testes unitários estão contidos na pasta **tests**.
* Os Patterns de arquivos de testes unitário aceitos são: **test_\*.py** e **\*_test.py**

Caso a configuração de sua aplicação seja diferente basta alterar o arquivo **pyproject.toml** de acordo!

### Configurando o arquivo .coveragerc
Crie um arquivo chamado **.coveragerc** e o configure com o seguinte conteúdo:
```
[paths]
source = app

[run]
branch = true
parallel = true
data_file = coverage/.coverage
omit = 
  /usr/lib/**/*
  /tests/*

[report]
show_missing = true
precision = 2

[html]
directory = htmlcov
```

Esta configuração indica o que é necessário para gerar o report de Code Coverage de seu projeto indicando a pasta **app** como sendo a pasta que irá conter seu código fonte.


### Validando a configuração da aplicação
Após realizar as configurações dos tópicos anteriores em sua aplicação execute o seguinte comando para validar se a mesma foi devidamente configurada:
```
pytest --cov=app --cov-report=xml --cov-report=html --nunitxml=reports/nunit/test-output.xml --junitxml=reports/junit/test-output.xml
```

Caso o comando seja executado com sucesso é sinal que sua aplicação foi devidamente configurada e que podemos seguir para os próximos passos.

## Utilização do template

Após configurar seu projeto com os pré-requisitos especificados no tópico anterior crie um arquivo chamado **azure-pipeline.yml** com o conteúdo deste [link](https://github.com/GladsonBruno/Python-AzureDevOps-CI-Example/blob/master/azure-pipeline.yml).

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
  template: /python/azure-pipeline.yml@templates
  parameters:
    CONTAINER_REGISTRY_SERVICE_CONNECTION_NAME: MyContainerRegistryServiceConnection
    DOCKER_REGISTRY: MyRegistry
    SONAR_CLOUD_SERVICE_CONNECTION_NAME: MySonarClodServiceConnection
    SONAR_CLOUD_ORGANIZATION: MySonarCloudOrganization
```


Após isso crie um variable group no Azure DevOps contendo as seguinte variáveis:

* **COVERAGE_REPORT_PATH**: Informa ao CI onde encontrar o arquivo coverage.xml que irá conter as informações de Code Coverage esperadas pela pipeline. O valor que uso normalmente é **\*\*/\*coverage.xml**

* **COVERAGERC_PATH**: Informa ao CI onde encontrar seu arquivo **.coveragerc**. O valor que uso normalmente é **$(Build.SourcesDirectory)/.coveragerc**

* **IMAGE_NAME**: Nome que será atribuído ao seu container Docker. Recomendo nomear o container com letras minúsculas separadas por **-**. Exemplo: **demo-api**

* **PYTHON_VERSION**: Informa a pipeline a versão do Python utilizado pela aplicação. Exemplo: **2**, **3**.

* **REQUIREMENTS_TXT_PATH**: Informa ao CI onde encontrar seu arquivo **requirements.txt**. O valor que uso normalmente é **$(Build.SourcesDirectory)/requirements.txt**

* **SONAR_EXCLUSIONS**: Informa ao Sonar Cloud quais diretórios e arquivos serão excluídos da análise de qualidade. O valor que uso normalmente é: **coverage/\*\*,env/\*\*,htmlcov/\*\*,reports/\*\***

* **SONAR_INCLUSIONS**: Informa ao Sonar Cloud quais diretórios e arquivos serão incluídos da análise de qualidade. O valor que uso normalmente é: **\*\*/\*.py**

* **SONAR_JUNIT_REPORT_PATH**: Informa ao Azure DevOps a localização do report Junit auxiliar a cobertura de código se encontra. O valor que uso normalmente é **reports/junit/test-output.xml**

* **SONAR_PATH_SOURCES**: Informa ao Sonar Cloud o path no qual está contido seu código. Normalmente uso o valor **app**.

* **SONAR_PROJECT_KEY**: Chave do projeto criada no Sonar Cloud.

* **SONAR_PROJECT_NAME**: Nome do projeto criado no Sonar Cloud.

* **SONAR_XUNIT_REPORT_PATH**: Informa ao Sonar a localização do report Xunit auxiliar a cobertura de código se encontra. O valor que uso normalmente é **reports/nunit/test-output.xml**

* **TEST_PATH**: Informa ao CI a pasta que contém seus testes unitários. Normalmente uso o valor **tests**.

Com o variable group criado insira o nome dele na pipeline substituindo o variable group **python-ci-example**.

Exemplo do arquivo de pipeline após finalizar a configuração:
```
trigger:
  branches:
    include:
      - master
      - develop

variables:
  - group: python-ci-example

resources:
  repositories:
  - repository: templates
    type: github
    name: GladsonBruno/azure-template-pipelines
    endpoint: GladsonBruno
    ref: 'refs/heads/main'

extends:
  template: /python/azure-pipeline.yml@templates
  parameters:
    CONTAINER_REGISTRY_SERVICE_CONNECTION_NAME: MyContainerRegistryServiceConnection
    DOCKER_REGISTRY: MyRegistry
    SONAR_CLOUD_SERVICE_CONNECTION_NAME: MySonarClodServiceConnection
    SONAR_CLOUD_ORGANIZATION: MySonarCloudOrganization
```