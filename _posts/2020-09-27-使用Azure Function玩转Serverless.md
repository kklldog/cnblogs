## Serverless&Azure Functions
通过无服务器计算，开发者无需管理基础结构，从而可以更快构建应用程序。通过无服务器应用程序，将由云服务提供商自动预配、缩放和管理运行代码所需的基础结构。

要理解无服务器计算的定义，注意到服务器仍在运行代码很重要。服务器名称来源于这样一个事实：与基础结构预配和管理相关联的任务对开发者不可见。这种方式让开发者能够更多地专注于业务逻辑，向业务核心交付更多价值。无服务器计算可帮助团队提高生产力、更快将产品推向市场，并让组织可以更好地优化资源、保持专注于创新。
> 引用自[Azure文档](https://azure.microsoft.com/zh-cn/overview/serverless-computing/)

以上文字引用自Azure，说的有点官方。说说我个人对Serverless的简单理解。所谓Serverless是相对于传统有应用的部署方案来说的。传统应用不管你是直接部署在主机或者容器化来部署，你的程序总是跑在一个完整的应用进程下。比如你只是想提供一个简单的A+C=C的Web Api，你往往需要启动一个完整的asp.net mvc项目或者sprintmvc项目来承载简单的代码。那么Serverless通过云服务把目标更加细化，通过它你可以只使用相关代码实现某个方法或函数，上传到云端后这个函数就可以跑了。这样子的话资源占用更加少，相对的资源付费也会更加有针对性，因为你无需为没用的资源所付费。Serverless可以说是对微服务的更加细化，使在云端运行的代码从application级别降低到了Function/method级别。    
上面简单介绍了Serverless。而Azure的Serverless方案就是Azure Functions。
## 创建函数应用
Azure Function是个免费服务，在免费服务列表里找到它并点击创建。
![0kfZ3F.png](https://s1.ax1x.com/2020/09/27/0kfZ3F.png)
给函数取个名称，发布选择“代码”。如果打算用.net来开发则运行时堆栈选择.NET Core版本选择3.1。跟其他资源一样区域选择东亚，因为它离你近。   
![0kfV9U.png](https://s1.ax1x.com/2020/09/27/0kfV9U.png)   
因为Azure Function虽然是Serverless但是也些储存空间，所以需要配置存储账户。选择上次我们使用AzureBlob时候创建的存储账号。没有的话可以新建一个。    
操作系统任意选择Linux或者Windows。    
计划类型选择：消耗（无服务器）    
> Azure 函数提供1000000请求/月的免费额度

## 使用VSCode进行本地开发
在函数列表界面点击“本地开发”。会弹出本地开发指导。选择VSCode环境会出现VSCode的开发环境配置说明。    
![0kfec4.png](https://s1.ax1x.com/2020/09/27/0kfec4.png)   
首先本地需要安装node跟npm。使用下面的命令自动安装Core Tools包：
```
npm install -g azure-functions-core-tools@3 --unsafe-perm true
```
> 注意：这个包还是很大的，由于网络的原因有可能拉不下来。如果长时间下不下来也可以直接搜索azure-functions-core-tools直接下载独立安装包。

使用npm安装完core tools后还有安装Azure Functions的VSCode插件。    
![0kfCBn.png](https://s1.ax1x.com/2020/09/27/0kfCBn.png)    
打开VSCode插件菜单，搜索Azure Functions，找到Azure Functions插件后点击Install开始安装。这个插件一会就安装完了。
## 新建Function
我们按照完VSCode的插件后，切换到Azure Function菜单。   
![0kfsC8.png](https://s1.ax1x.com/2020/09/27/0kfsC8.png)   
点击新建按钮会弹出Azure Function支持的触发器。触发器有很多有HttpTrigger，BlobTrigger，CosmosDbTrigger等等。这里选择最简单的HttpTriger触发器。接着会提示输入项目名称，输入名称后回车就可以生成本地项目了。
## Function代码
我们简单演示下Azure Function，使用这个函数实现一个简单的两个数相加返回相加结果。   
```
namespace Company.Function
{
    public static class AzFnTest
    {
        [FunctionName("AzFnTest")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            dynamic data = JsonConvert.DeserializeObject(requestBody);
            
            int a = data.A;
            int b = data.B;

            int c = a + b;

            return new OkObjectResult(c);
        }
    }
}
```
以C#为语言的Azure Function入口就是一个run方法。run方法的入参有2个，一个是HttpRequest，一个Ilogger。其中HttpRequest包含了http请求的信息，QueryString、body、headers等。这个类就是来自Microsoft.AspNetCore.Mvc命名空间。返回值是Task<IActionResult>。那么本质上一个Function其实可以看做是标准MVC方案里的一个Action。只是缺乏了参数自动绑定。我们需要的参数都要从HttpRequest对象上提取。    
上面的代码很简单，就是获取body内容反序列化成一个动态对象，获取参数A、B，然后相加得到C，通过OkObjectResult直接返回出去。
## 本地测试
在VSCode界面按F5启动调试。VSCode会启动一个本地实例，可以接受http请求。我们使用postman往这个地址post一个json数据过去。
```
{
    "A" : 1 ,
    "B" : 2
}
```
![0kfkNV.png](https://s1.ax1x.com/2020/09/27/0kfkNV.png)   
可以看到返回了结果3。   
## 上传到Azure
![0kfP7q.png](https://s1.ax1x.com/2020/09/27/0kfP7q.png)    
在VSCode上点击上传按钮，会提示登录Azure。登录成功后会列出上面我们新建的Azure Function的资源。
![0kfAhT.png](https://s1.ax1x.com/2020/09/27/0kfAhT.png)    
选择azure-fn0，选中之后会开始上传，最后output窗口会提示成功。
![0AKpY4.png](https://s1.ax1x.com/2020/09/27/0AKpY4.png)
回到portal网站刷新下，会看到我们的项目已经上传成功了。
## 运行函数
点击函数名称弹出明细界面。点击“获取函数URL”获取调用这个函数的真实URL。   
![0AKmkD.png](https://s1.ax1x.com/2020/09/27/0AKmkD.png)   
有了这个地址我们就可以在全球范围内使用这个函数啦。让我们使用Postman再测试一下。
![0AKG0f.png](https://s1.ax1x.com/2020/09/27/0AKG0f.png)
可以看到返回了正确的结果。
## 总结
以上我们使用C#代码实现了一个简单的Azure Functions并调用了它。Azure Functions还支持Java、Nodejs、Python等常用的编程语言。Azure Functions跟Azure生态紧密结合，除了支持HttpTrigger，还支持CosmosDb，Azure Blob，甚至是Iot边缘计算等场景。Azure Functions是Azure的Serverless解决方案，它具有无需基础结构管理、动态可伸缩性、加快上市、更高效地使用资源等优点，大家如果想体验Serverless可以尝试一下。
    
## 关注我的公众号一起玩转技术   
![](https://s1.ax1x.com/2020/06/29/NfQjds.jpg)