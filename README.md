# Sobre o projeto
O projeto tem o objetivo de ser usado como template de pipelines para o Azure DevOps.

# Pré-requisitos para utilização
* Possuir um Service Connection configurado no Azure DevOps para realizar a interface entre o Docker Registry e o Azure DevOps.

* Possuir um Service Connection configurado no Azure DevOps para realizar a interface entre o Sonar Cloud e o Azure DevOps.

* Possuir um Service Connection configurado no Azure DevOps para realizar a interface entre o GitHub e o Azure DevOps.

* Realizar o fork deste projeto de template no GitHub.

* Possuir os seguintes extensões instaladas em sua organization no Azure DevOps:

  * [GitTools](https://marketplace.visualstudio.com/items?itemName=gittools.gittools)

  * [SonarCloud](https://marketplace.visualstudio.com/items?itemName=SonarSource.sonarcloud)

  * [SonarCloud build breaker](https://marketplace.visualstudio.com/items?itemName=SimondeLang.sonarcloud-buildbreaker)

  * [Tag\Branch Git on Release](https://marketplace.visualstudio.com/items?itemName=jabbera.git-tag-on-release-task)


# Utilizando os templates disponíveis
## Template Maven
Para utilizar o template Maven faça a leitura do seguinte manual:
* [Manual Template Maven](./maven/manual-maven.md).

## Template Dotnet
Para utilizar o template Dotnet faça a leitura do seguinte manual:
* [Manual Template Dotnet](./dotnet/manual-dotnet.md).


## Template Node
Para utilizar o template Node faça a leitura do seguinte manual:
* [Manual Template Node](./node/manual-node.md).


## Template Python
Para utilizar o template Python faça a leitura do seguinte manual:
* [Manual Template Python](./python/manual-python.md).
