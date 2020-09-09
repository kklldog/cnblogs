```
  FROM mcr.microsoft.com/dotnet/core/sdk:3.1-bionic AS build
WORKDIR /app
COPY /. /app
RUN dotnet restore
WORKDIR /app/devops_test
RUN dotnet publish -o ./out -c Release
EXPOSE 5000
ENTRYPOINT ["dotnet", "out/devops_test.dll"]
```
![w8E7a8.png](https://s1.ax1x.com/2020/09/09/w8E7a8.png)    
![w8E5rt.png](https://s1.ax1x.com/2020/09/09/w8E5rt.png)    
![w85NRA.png](https://s1.ax1x.com/2020/09/10/w85NRA.png)    
![w85tGd.png](https://s1.ax1x.com/2020/09/10/w85tGd.png)    
![w85YPH.png](https://s1.ax1x.com/2020/09/10/w85YPH.png)    
```
# ASP.NET Core (.NET Framework)
# Build and test ASP.NET Core projects targeting the full .NET Framework.
# Add steps that publish symbols, save build artifacts, and more:
# https://docs.microsoft.com/azure/devops/pipelines/languages/dotnet-core

trigger:
- master

pool:
  vmImage: 'ubuntu-18.04'

variables:
  solution: '**/*.sln'
  buildPlatform: 'Any CPU'
  buildConfiguration: 'Release'

steps:
- task: DotNetCoreCLI@2
  inputs:
    command: 'restore'
- task: DotNetCoreCLI@2
  displayName: Build
  inputs:
    command: build
    projects: '**/*.csproj'
    arguments: '--configuration $(buildConfiguration)'

- task: DotNetCoreCLI@2
  inputs:
    command: test
    projects: '**/*Tests/*.csproj'
    arguments: '--configuration $(buildConfiguration)'
- task: Docker@2
  inputs:
    containerRegistry: 'docker-hub'
    repository: 'kklldog/az_devop_test'
    command: 'buildAndPush'
    Dockerfile: '**/Dockerfile'
    tags: latest
- task: SSH@0
  inputs:
    sshEndpoint: 'azvm-ssh'
    runOptions: 'commands'
    commands: |
      docker rm -f az_devop_test
      docker rmi -f kklldog/az_devop_test
      docker pull kklldog/az_devop_test
      docker run -d -p 5000:5000 --name az_devop_test kklldog/az_devop_test
    readyTimeout: '20000'
```
![w85n2R.png](https://s1.ax1x.com/2020/09/10/w85n2R.png)    
