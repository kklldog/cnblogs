AgileConfig轻量级配置中心自第一个版本发布不知不觉已经半年了。在并未进行什么推广的情况下收到了250个star，对我有很大的鼓舞，并且也有不少同学试用，并且给出了宝贵的意见，非常感谢他们。其中有一些意见非常好，但是一直没有开发。主要是一来下半年比较忙（懒），二来我不想把AgileConfig搞的过于复杂。但其中有个需求被很多同学提及过，就是希望能支持应用间的继承（关联），类似Apollo的公共namespace的概念。比如微服务应用之间有不少公共配置项，可以配置在一个应用内，然后其他应用继承它，这样每个应用就不用重复的配置公共配置。我思考了一下，这个配置确实是个非常有用的功能，于是花了点时间实现了它。    
Github地址：[https://github.com/kklldog/AgileConfig](https://github.com/kklldog/AgileConfig) 求star 。    
下面的示例简单演示下如何使用AgileConfig读取配置并且使用继承功能
## 使用docker启动一个AgileConfig实例
```
sudo docker run --name agile_config -e adminConsole=true -e db:provider=sqlserver -e db:conn="Persist Security Info = False; User ID =dev; Password =dev@123,; Initial Catalog =agile_config_test; Server =." -p 5000:5000 kklldog/agile_config:latest

```
使用docker命令运行一个AgileConfig实例，这是最简单的方法。当然你也可以拉源码下来编译发布使用IIS来运行它。     
配置环境变量：    
adminConsole=true 开启控制台功能    
db:provider=sqlserver 数据库为SqlServer    
db:conn="Persist Security Info = False; User ID =dev; Password =dev@123,; Initial Catalog =agile_config_test; Server =." 配置数据库连接   
-p 5000:5000 容器的5000口映射本地的5000口   
## 配置AgileConfig
第一次运行需要配置管理密码    
![](https://camo.githubusercontent.com/f147bfbc04551c69d4d33039fe60943dbede4b9db6756704aa395871acc29c49/68747470733a2f2f73312e617831782e636f6d2f323032302f30362f30392f74344467494a2e706e67)
密码配置完成后重新登录进系统，开始配置节点    
![D66God.png](https://s3.ax1x.com/2020/11/29/D66God.png)   
在设计的时候节点跟控制台是分开的，但是为了部署简单最后节点跟控制台被实现在一起了。所以采用单节点部署的时候，该实例既是节点又是控制台，所以也需要把本节点的地址加入到节点列表里，以便控制台能管理到。
## 添加应用
AgileConfig的初始化完成了，现在我们开始添加应用。   
添加“公共应用”   
![D66oTJ.png](https://s3.ax1x.com/2020/11/29/D66oTJ.png)   
添加应用名称，应用id，勾选“可被继承”。点击确定完成公共的创建。系统只支持一层的继承，可被继承的应用不能再继承其它应用。    
创建完成后为公共应用添加配置项   
![D66jOO.png](https://s3.ax1x.com/2020/11/29/D66jOO.png)    
为公共应用添加一个配置项：键为public_key_01 值为0001 。    
    
添加“私有应用”
![D6cKts.png](https://s3.ax1x.com/2020/11/29/D6cKts.png)
添加一个私有应用，不要选“可被继承”。点击继承应用栏的加号会弹出可以被的继承应用列表，选择“公共应用”。点击“确定”完成创建。   
    
为私有应用创建配置项    
![D6c091.png](https://s3.ax1x.com/2020/11/29/D6c091.png)    
为私有应用添加一个配置项：键为private_key_01 值为0002 。    
    
> 注意：把所有的配置都上线，否则客户端读不到配置。

## 客户端读取配置
### 创建Asp.net Core WebApi项目
我们创建一个WebApi项目做为客户端来演示如何读取配置   
![D6giE4.png](https://s3.ax1x.com/2020/11/29/D6giE4.png)
### 使用nuget引用AgileConfig.Client
```
Install-Package AgileConfig.Client -Version 1.1.0
```
### 集成AgileConfig.Client
使用nuget安装成功后，切换到Program.cs开始集成AgileConfigClient。   
```
 public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((ctx,cfg)=> {
                var appId = "private_01";
                var secret = "";
                var nodes = "http://localhost:5000";
                //new一个client实例
                var configClient = new ConfigClient(appId, secret, nodes);
                //使用AddAgileConfig配置一个新的IConfigurationSource
                cfg.AddAgileConfig(configClient);
            })
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                });
```
使用IHostBuilder的ConfigureAppConfiguration把我们的ConfigClient注入进去。这里我们的ConfigClient配置的是私有应用的id：private_01 。
### 读取配置
前期工作都完成了，现在我们可以开始编写读取配置的代码了。   
新建一个ReadConfigController：
```
   [ApiController]
    [Route("[controller]")]
    public class ReadConfigController : ControllerBase
    {
        private readonly IConfiguration _IConfiguration;
        public ReadConfigController(IConfiguration configuration)
        {
            _IConfiguration = configuration;
        }

        [HttpGet]
        public String Get()
        {
            var publicConfig = _IConfiguration["public_key_01"];
            var privateConfig = _IConfiguration["private_key_01"];


            return $"publicConfig:{publicConfig} , privateConfig:{privateConfig}";
        }
    }
```
通过构造函数注入IConfiguration，然后通过它直接读取公共配置，私有配置，并且直接把字符串返回回去。
> 注意修改一下客户端程序的启动端口，默认5000跟上面的AgileConfig实例占用的端口冲突。

### 运行一下
运行客户端项目，然后在浏览器里输入http://localhost:51605/readconfig   
![D6WrPH.png](https://s3.ax1x.com/2020/11/29/D6WrPH.png)   
可以看到我们的公共配置跟私有配置都准确的读取到了。

## 总结
通过以上一个简单的示例，演示了如何使用AgileConfig读取配置以及如何在应用间继承配置。以上示例并未展示所有内容，使用继承的时候需要注意一下几点：   
1. 当私有应用的配置跟被继承应用重复时，私有应用的配置会覆盖被继承应用的配置
2. 如果一个应用被标记为"可被继承"后，这个应用自己不能继续继承其它应用
3. 一个私有应用可以继承多个“可被继承”的应用，如果多个继承的应用间出现重复的配置，那么将按照继承的顺序，后面的应用会覆盖前面的应用。   

如果喜欢这个项目的，请给我star吧。谢谢。

## 关注我的公众号一起玩转技术   
![](https://s1.ax1x.com/2020/06/29/NfQjds.jpg)