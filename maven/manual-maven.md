## Manual template Maven
O template Maven cobre os seguintes pontos:
* Versionamento automático

* Qualidade de código com Sonar Cloud

* Testes unitários

* Code Coverage

* Build do binário da aplicação e geração de container Docker

* Scan de vulnerabilidades no container Docker gerado

Um exemplo de utilização pode ser visto neste [link](https://github.com/GladsonBruno/SpringBoot-AzureDevOps-CI-Example/blob/master/azure-pipeline.yml)

## Pré requisitos
* Possuir o Azure DevOps devidamente configurado conforme recomendado neste [manual](../README.md).

* Possui o plugin do Jacoco configurado na aplicação.

* Possuir o plugin Sonar Maven Plugin configurado na aplicação.

* Possuir pelo menos um teste unitário implementado ou vazio. ( Caso queira ter as informações de Code Coverage no Sonar e Azure. )

## Configuração do plugins do Maven esperados pelo CI
Exemplo de configuração do **pom.xml** da aplicação:
```
.
.
.
	<build>
		<finalName>${project.artifactId}</finalName>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
			<plugin> <!-- Plugin do SonarQube -->
				<groupId>org.sonarsource.scanner.maven</groupId>
				<artifactId>sonar-maven-plugin</artifactId>
				<version>3.6.0.1398</version>
			</plugin>
			<plugin>
				<groupId>org.jacoco</groupId>
				<artifactId>jacoco-maven-plugin</artifactId>
				<version>0.8.8</version>
				<executions>
					<execution>
						<goals>
							<goal>prepare-agent</goal>
						</goals>
					</execution>
					<!-- attached to Maven test phase -->
					<execution>
						<id>report</id>
						<phase>test</phase>
						<goals>
							<goal>report</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
.
.
.
```

Caso queira conferir um exemplo real desta configuração acesse este [link](https://github.com/GladsonBruno/SpringBoot-AzureDevOps-CI-Example/blob/master/pom.xml).

## Utilização do template

Após configurar seu projeto com os pré-requisitos especificados no tópico anterior crie um arquivo chamado **azure-pipeline.yml** com o conteúdo deste [link](https://github.com/GladsonBruno/SpringBoot-AzureDevOps-CI-Example/blob/master/azure-pipeline.yml).

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
  template: /maven/azure-pipeline.yml@templates
  parameters:
    CONTAINER_REGISTRY_SERVICE_CONNECTION_NAME: MyContainerRegistryServiceConnection
    DOCKER_REGISTRY: MyRegistry
    SONAR_CLOUD_SERVICE_CONNECTION_NAME: MySonarClodServiceConnection
    SONAR_CLOUD_ORGANIZATION: MySonarCloudOrganization
```


Após isso crie um variable group no Azure DevOps contendo as seguinte variáveis:

* **IMAGE_NAME**: Nome que será atribuído ao seu container Docker.

* **JACOCO_REPORT_PATH** Pasta no qual o report de code coverage do Jacoco será gerado. Por padrão o valor desta variável é: **\*\*/jacoco.xml**

* **JAVA_JDK_VERSION**: Informa a pipeline a versão do JDK utilizado pela aplicação. Exemplo: **11**, **17**.

* **SONAR_EXCLUSIONS**: Informa ao Sonar Cloud quais diretórios e arquivos serão excluídos da análise de qualidade. O valor que uso normalmente é: **src/test/*.java,src/main/resources/application.properties**

* **SONAR_INCLUSIONS**: Informa ao Sonar Cloud quais diretórios e arquivos serão incluídos da análise de qualidade. O valor que uso normalmente é: **\*\*/\*.java**

* **SONAR_PROJECT_KEY**: Chave do projeto criada no Sonar Cloud.

* **SONAR_PROJECT_NAME**: Nome do projeto criado no Sonar Cloud.

Com o variable group criado insira o nome dele na pipeline substituindo o variable group **springboot-ci-example**.

Exemplo do arquivo de pipeline após finalizar a configuração:

```
trigger:
  branches:
    include:
      - master
      - develop

variables:
  - group: springboot-ci-example

resources:
  repositories:
  - repository: templates
    type: github
    name: GitUsername/azure-template-pipelines
    endpoint: GitHubServiceConnection
    ref: 'refs/heads/main'

extends:
  template: /maven/azure-pipeline.yml@templates
  parameters:
    CONTAINER_REGISTRY_SERVICE_CONNECTION_NAME: MyContainerRegistryServiceConnection
    DOCKER_REGISTRY: MyRegistry
    SONAR_CLOUD_SERVICE_CONNECTION_NAME: MySonarClodServiceConnection
    SONAR_CLOUD_ORGANIZATION: MySonarCloudOrganization
```