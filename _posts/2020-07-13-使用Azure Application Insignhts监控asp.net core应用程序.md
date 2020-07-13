Application Insignhts是微软开发的一套监控程序。他可以对线上的应用程序进行全方位的监控，比如监控每秒的请求数，失败的请求，追踪异常，对每个请求进行监控，从http的耗时，到SQL查询的耗时，完完整整的被记录下来。当对程序进行优化跟排错时非常好使。它原来是visualstudio online的一个服务，现在合并进了Azure，作为Azure monitor的一个组件。虽然合并进了Azure，但是Application Insignhts还是免费的。    

## 什么是Application Insignhts
Application Insights 是 Azure Monitor 的一项功能，是面向开发人员和 DevOps 专业人员的可扩展应用程序性能管理 (APM) 服务。 使用它可以监视实时应用程序。 它将自动检测性能异常，并且包含了强大的分析工具来帮助诊断问题，了解用户在应用中实际执行了哪些操作。 它旨在帮助持续提高性能与可用性。 它适用于本地云、混合云或任何公有云中托管的各种平台（包括 .NET、Node.js、Java 和 Python）上的应用。 它与 DevOps 进程集成，并且具有与不同开发工具的连接点。 可以通过与 Visual Studio App Center 集成来监视和分析移动应用的遥测数据。    
摘自微软文档：[app-insights-overview](https://docs.microsoft.com/zh-cn/azure/azure-monitor/app/app-insights-overview)
## 在Azure创建Application Insignhts服务
上一次介绍了如何注册12个月免费订阅账号[如何白嫖微软Azure12个月及避坑指南](https://www.cnblogs.com/kklldog/p/azure-free-12m.html)，使用账号登录管理平台后，找到Application Insignhts服务，点击创建。    
![UYPsET.png](https://s1.ax1x.com/2020/07/13/UYPsET.png)   
在创建界面选择资源组，填写实例名称，选择区域，选择个离你近的。    
![UYiZq0.png](https://s1.ax1x.com/2020/07/13/UYiZq0.png)    
创建一个标记。标记其实就是一组键值对，主要用来统计的时候进行区分跟合并用的。    
![UYkTaV.png](https://s1.ax1x.com/2020/07/13/UYkTaV.png)    
最后点提交，等待一会就会提示部署完成。    
![UYABz4.png](https://s1.ax1x.com/2020/07/13/UYABz4.png)    
部署成功后回到管理控制台主页，找到所有资源，点击刚才填写的实例名就可以查看详情了。   
![UYZbwR.png](https://s1.ax1x.com/2020/07/13/UYZbwR.png)   
这个页面默认会显示几个指标，因为截图的时候是我已经接入过了，所以有数据，第一次进去应该是没有数据的。    
> “检测密钥”比较重要，后面asp.net core程序对接的时候需要用到。
## 在asp.net core程序接入Application Insignhts服务
在asp.net core程序接入Application Insignhts服务非常简单。简单的配置几行代码就可以运行了，对业务代码完全没有侵入。    
找一个asp.net core的程序，在.csproj文件下加入Application Insignhts包的引用。
```
<Project Sdk="Microsoft.NET.Sdk.Web">
    ...
    ...
  <ItemGroup>
    <PackageReference Include="Microsoft.ApplicationInsights.AspNetCore" Version="2.13.1" />
  </ItemGroup>
</Project>

```
在Startup.ConfigureServices下注入Application Insignhts相关的服务。
```
       public void ConfigureServices(IServiceCollection services)
        {
            //register application insights
            services.AddApplicationInsightsTelemetry();
            ...
            ...
        }
```
在配置文件appsettings.json下配置检测密钥。
```
{
  "ApplicationInsights": {
    "InstrumentationKey": "xxxxxxxxxxxxxx"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  },
  ...
  ...
}
```
这样asp.net core程序就配置好了。正常流程发布程序后部署它。
## 查看应用程序监控指标
发布完程序，等程序运行一段时间后就可以去管理界面查看监控指标了。    
![UYMNwQ.png](https://s1.ax1x.com/2020/07/13/UYMNwQ.png)
默认有4个指标：
1. 失败的请求数
2. 服务器响应时间
3. 服务器请求
4. 可用性

其中比较有意思的是服务器响应时间跟服务器请求这2个指标，对于我们调优有非常大的意义。服务器响应时间跟服务器请求点进去其实是进了性能指标的界面。    
![UYl6IJ.png](https://s1.ax1x.com/2020/07/13/UYl6IJ.png)
该界面展示了服务器一段时间内接受到的请求数量及响应速度。同时列出一些慢的请求，点击一个请求可以看到更加明细的信息。    
点击第一个最慢的看看为什么会这么慢。
![UY1nwF.png](https://s1.ax1x.com/2020/07/13/UY1nwF.png)    
可以看到这个请求耗时主要是SQL跟HTTP，其中SQL平均耗时17ms，这个肯定没问题。HTTP平均耗时650ms那么这个接口慢的问题基本被锁定了。    
这还没完，继续点击深入钻取...示例按钮，还有更加详细的信息。
![UY1HmT.png](https://s1.ax1x.com/2020/07/13/UY1HmT.png)    
点击示例按钮，会列出该接口近期的一些调用示例。选一个耗时比较长的进入点击进去，还有更详细的信息。    
![UY3QHS.png](https://s1.ax1x.com/2020/07/13/UY3QHS.png)    
通过这图就很清晰了，这个请求包含了多次SQL请求，跟2次HTTP请求。SQL请求耗时都在1ms左右，其中一次HTTP请求1.7s，那么很明显了，就是这个HTTP请求拖慢了整个请求，所有需要对这个HTTP请求进行优化。    
这还没完，点击其中的SQL请求，还有更详细的信息，能显示执行了什么SQL语句。   
![UYUNi8.png](https://s1.ax1x.com/2020/07/13/UYUNi8.png)    
点击HTTP请求，同样会列出详细信息，包括请求的URL等信息。    
![UYUjQH.png](https://s1.ax1x.com/2020/07/13/UYUjQH.png)
## 其他指标
除了默认列出来的指标，其实还有很多指标能够查看。   
在右侧边栏点击指标菜单，显示指标筛选界面。在该界面可以添加自己想看的指标。比如CPU，内存等信息。    
![UYdBEq.png](https://s1.ax1x.com/2020/07/13/UYdBEq.png)
## 实时指标
实时指标是个很酷炫的功能，可以在一个界面动态实时显示N个指标。   
![UY0iY6.gif](https://s1.ax1x.com/2020/07/13/UY0iY6.gif)
## 总结
asp.net core程序使用Application Insignhts非常简单，通过简单的几行代码就集成完成，并且对业务代码零侵入。Application Insignhts的监控功能非常强大，可以对应用程序、服务器各种指标进行监控。特别是性能指标的请求，对我们进行线上程序的调优具有非常强大指导意义。