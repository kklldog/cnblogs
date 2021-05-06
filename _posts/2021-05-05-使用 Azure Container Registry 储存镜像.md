Azure Container Registry（容器注册表）是基于 Docker Registry 2.0规范的托管专用 Docker 注册表服务。 可以创建和维护 Azure 容器注册表来存储与管理专用的 Docker 容器映像和相关项目。   
Azure Container Registry 类似与阿里云的容器镜像服务。提供镜像的私有存储服务器。对于12月试用账户有100G的免费存储额度及10个Webhook的能力。   
依托 Azure 的全球节点可以使你的镜像在全球范围能被访问到并快速拉取。   
以下是 Azure Container Registry 的简单试用。
## 创建资源
![g1VWG9.png](https://z3.ax1x.com/2021/05/06/g1VWG9.png)   
在免费服务列表找到容器注册表，点击“创建”。
![g1Vf2R.png](https://z3.ax1x.com/2021/05/06/g1Vf2R.png)   
在弹出的创建界面填写资源组、注册表名称等信息。   
位置选择离你近的，比如东南亚。   
SKU选择基本。   
点击“查看+创建”按钮。   
![g1VcaF.png](https://z3.ax1x.com/2021/05/06/g1VcaF.png)   
在校验通过后，点击“创建”按钮。   
![g1Vg54.png](https://z3.ax1x.com/2021/05/06/g1Vg54.png)   
在经过几秒钟的等待后我们的资源就被创建好了，点击“转到资源”可以查看Azure Container Registry的概要信息。   
其中比较重要的是右上角的，登录服务器：minjiezhou.azure.io 。后面的操作需要使用到。
## 上传本地镜像
下面演示下如何通过 Azure CLI 命令行来上传镜像到注册表。 
```
az acr login --name minjiezhou
```
使用az acr login 命令登录到 Azure Container Registry 。   
> 请先安装Azure CLI 。
```
docker images 

REPOSITORY                         TAG                 IMAGE ID            CREATED             SIZE
mcr.microsoft.com/dotnet/sdk       3.1                 b4f189e5f593        3 weeks ago         710MB
mcr.microsoft.com/dotnet/runtime   3.1                 e77a510a55f6        3 weeks ago         190MB
kklldog/agile_config               test                68288d3f5669        4 weeks ago         281MB
kklldog/agile_config               latest              6b2b834fa8d4        5 months ago        281MB
```
登录成功后，我们先列一下本地的镜像。如果本地没有镜像那就先去dockerhub上拉一个下来。
```
docker tag kklldog/agile_config minjiezhou.azurecr.io/agile_config:v1
```
我们演示下把agile_config的镜像推送到容器注册表上去。   
使用 docker tag 命令重命名镜像。重命名的格式为 <登录服务器>/agile_config:v1
```
docker push minjiezhou.azurecr.io/agile_config:v1

The push refers to repository [minjiezhou.azurecr.io/agile_config]
f3f098bf4d75: Pushed
3635892d0647: Pushed
d3d8723bb140: Pushed
bbd61b971886: Pushed
dc4a66fc412f: Pushed
b22af9287e60: Pushed
f5600c6330da: Pushed
v1: digest: sha256:15113de4c788ac61aecdb3a676beaff18f09dd8f786b012e5f14274f295e7dc7 size: 1793
```
使用 docker push 命令开始推送。等待命令执行完毕后转到门户查看。   
![g1esNF.png](https://z3.ax1x.com/2021/05/06/g1esNF.png)
点击“储存库”菜单，可以看到我们的agile_config镜像已经存在了。
```
docker rmi minjiezhou.azurecr.io/agile_config:v1
```
为了测试拉取镜像，我们先使用 docker rmi 命令删除本地的镜像。
```
docker pull minjiezhou.azurecr.io/agile_config:v1

v1: Pulling from agile_config
Digest: sha256:15113de4c788ac61aecdb3a676beaff18f09dd8f786b012e5f14274f295e7dc7
Status: Downloaded newer image for minjiezhou.azurecr.io/agile_config:v1
minjiezhou.azurecr.io/agile_config:v1
```
使用 docker pull 命令从Azure容器注册表服务拉取我们的agile_config镜像。

## 总结
通过以上简单的几步操作，我们演示了如何通过门户开通 Azure 容器注册表服务。以及如何通过 Azure CLI 命令上传下载 docker 镜像等操作。通过简单的几步我们就拥有了一个在全球范围内能轻松访问的容器仓库服务。   

## 关注我的公众号一起玩转技术   
![](https://s1.ax1x.com/2020/06/29/NfQjds.jpg)