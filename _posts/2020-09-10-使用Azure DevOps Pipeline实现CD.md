
上一次我们讲了使用Azure DevOps Pipeline实现.Net Core程序的CI。这次我们来演示下如何使用Azure DevOps实现.Net Core程序的CD。
    
实现本次目标我们除了Azure DevOps外还需要：   
1. 一台安装了Docker的主机
2. 一个 Docker Hub 账号

上一次我们的CI实现了：   
**发布**>**编译**>**单元测试**    
这次我们要实现剩下的几个步骤：   
**生成镜像**>**推送镜像**>**部署**
## 创建Docker镜像仓库
我们生成的镜像需要有个存放的地方。各大云厂商其实都有这种服务，这次直接使用Docker Hub提供的公共仓库服务。
![w8IEOP.png](https://s1.ax1x.com/2020/09/10/w8IEOP.png)    
创建一个仓库名叫：az_devop_test。
## 创建Dockerfile
我们的代码创建为镜像需要一个Dockerfile来描述如何构建这个镜像。在项目根目录下新建一个文件命名为Dockerfile注意不带任何后缀名。以下为Dockerfile的内容：
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
## 配置Servic Connections
选择ProjectSetting菜单，选择Service connections。Service connections用来存储跟外部服务相关的账号密码等信息，这里我们需要配置2个service。    
1. Docker Hub的信息
2. 主机SSH的信息

![w8E7a8.png](https://s1.ax1x.com/2020/09/09/w8E7a8.png)    
### 配置Docker Registry service
我们的pipeline需要给Docker hub推送镜像，所以需要一些必要的信息，比如账号密码等信息。
![wG8uE4.png](https://s1.ax1x.com/2020/09/10/wG8uE4.png)   
点击"New Service"找到Docker Register项目点击下一步    
    
![w8E5rt.png](https://s1.ax1x.com/2020/09/09/w8E5rt.png)    
选择DockerHub，填写对应的账号密码   
### 配置SSH service
我们的pipeline在完成镜像推送后需要通过SSH登录到主机运行命令把新的镜像跑起来。   
![w85NRA.png](https://s1.ax1x.com/2020/09/10/w85NRA.png)    
填写主机IP端口等信息。
## 修改pipeline
上次我们的pipeline已经定义好了CI的步骤，这次需要在上次的基础上继续完善CD的功能。
### BuildAndPush Task
找到上次的pipeline选择编辑功能，在右边的Task列表里找到DockerTask，点击出现配置界面    
![wGJsHS.png](https://s1.ax1x.com/2020/09/10/wGJsHS.png)    
在Container register里选择前面在service connections配置的docker-hub服务。
    
repository填写我们在docker hub上新建的仓库：kklldog/az_devop_test 。注意：仓库名称要把用户名写全了不然推不上去。
    
tags填写：latest 。
    
command：选择buildAndPush 。build跟push本是两步操作，这里直接合并为一步。

![w85tGd.png](https://s1.ax1x.com/2020/09/10/w85tGd.png)    
### SSH Task
添加完BuildAndPush Task后同样的方法再次添加SSH Task。    
![w85YPH.png](https://s1.ax1x.com/2020/09/10/w85YPH.png)    
SSH service connection里选择前面我们配置好的ssh service。    
Run 这里选择Commands。   
Commands 填写需要执行的命令：
```
docker rm -f az_devop_test
      docker rmi -f kklldog/az_devop_test
      docker pull kklldog/az_devop_test
      docker run -d -p 5000:5000 --name az_devop_test kklldog/az_devop_test
```
简单解释下这个命令：   
删除运行的容器；删除镜像；拉取最新的镜像；使用新的镜像运行一个容器。
## 完整的pipeline
到此我们的pipeline已经配置好了，以下是完整的pipeline.yml代码。
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
## 运行一下
手动运行一个这个pipeline，点击pipeline可以看到实时的日志，等到最后可以看到每一步都成功了，说明我们的pipeline配置成功了。
![w85n2R.png](https://s1.ax1x.com/2020/09/10/w85n2R.png)    
访问一下容器对应的端口，我们的网站已经可以访问了。修改一下代码，然后提交，每次都会自动部署最新的代码到主机。   
![wGUSbV.png](https://s1.ax1x.com/2020/09/10/wGUSbV.png)
## 总结
以上通过2篇文章简单的介绍了Azure DevOps Pipeline如何实现CICD功能。Azure DevOps Pipeline给我的感觉是比较易用的，配置yml其实都是图形化的上手难度会比较小。另外它跟Github有深入的集成，可以使用Github的账号直接登录。况且它还是个免费服务，大家可以试试。
    
## 关注我的公众号一起玩转技术   
![](https://s1.ax1x.com/2020/06/29/NfQjds.jpg)