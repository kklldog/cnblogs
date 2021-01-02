下面通过几个示例来演示下如何AgileConfig.Client来读取配置：
## asp.net core mvc下读取配置
mvc项目应该是目前使用最广泛的项目，同样它与AgileConfig.Client的集成最深入。下面来看看如何在mvc项目下使用AgileConfig.Client。
### 安装AgileConfig.Client
```
Install-Package AgileConfig.Client
```
当然第一步是使用nuget命令安装最新版的Client库。
### 修改appsettings.json
```
  "AgileConfig": {
    "appId": "test_app",
    "secret": "",
    "nodes": "http://agileconfig.xbaby.xyz:5000"
  }
```
AgileConfig.Client连接服务端需要一点必要的信息，我们把这些信息配置在appsettings.json文件里。节点的名称叫“AgileConfig”，里面配置了：   
1. appId 应用id
2. secret 应用密钥，没有的话留空
3. nodes 节点地址，如果有多个则使用英文逗号(,)分隔
    
### AddAgileConfig