上次介绍了Azure Application Insights，实现了.net core程序的监控功能。这次让我们来看看Azure DevOps Pipeline功能。Azure DevOps Pipeline 是Azure DevOps里面的一个组件，对于12个月试用账号同样永久免费。
    
![Uyf6xI.png](https://s1.ax1x.com/2020/07/17/Uyf6xI.png)
## 持续集成CI
持续集成指的是，频繁地（一天多次）将代码集成到主干。
它的好处主要有两个。
```
（1）快速发现错误。每完成一点更新，就集成到主干，可以快速发现错误，定位错误也比较容易。

（2）防止分支大幅偏离主干。如果不是经常集成，主干又在不断更新，会导致以后集成的难度变大，甚至难以集成。
```
持续集成的目的，就是让产品可以快速迭代，同时还能保持高质量。它的核心措施是，代码集成到主干之前，必须通过自动化测试。只要有一个测试用例失败，就不能集成。
Martin Fowler说过，"持续集成并不能消除Bug，而是让它们非常容易发现和改正。"    
*[摘自阮一峰大神的blog](http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html)*
     
DevOps跟CI就不多介绍了。这里我们订个目标：当我们提交代码后，服务器自动编译代码，自动运行单元测试，自动发送成功失败的邮件。
## 创建组织
![Uy0G8S.png](https://s1.ax1x.com/2020/07/17/Uy0G8S.png)
    
开通Azure DevOps功能，第一步需要创建一个组织。
![Uy0gKJ.png](https://s1.ax1x.com/2020/07/17/Uy0gKJ.png)
    
随便取个组织名称，区域还是那个套路，选近的，这里选东亚。
![UyB9xg.png](https://s1.ax1x.com/2020/07/17/UyB9xg.png)
     
## 创建仓库
点击继续之后页面会跳转到正式的Azure DevOps界面。首先需要创建一个项目。这里跟Github一样，需要选择私有还有公开，估计Azure DevOps后端其实就是使用了Github的服务。这里选一个私有的吧，取个项目名称：devop_test ，还可以设置用户名密码等信息。
![UyBuzF.png](https://s1.ax1x.com/2020/07/17/UyBuzF.png)
    
## 创建ASP.NET MVC项目
新建一个ASP.NET MVC项目，就默认的示例项目就行。

![UyBXy4.png](https://s1.ax1x.com/2020/07/17/UyBXy4.png)
    
为了让pipeline执行单元测试，所以我们新建一个单元测试功能，然后写一个最简单的单元测试方法。
```
  [TestClass()]
    public class WeatherForecastControllerTests
    {
        [TestMethod()]
        public void GetTest()
        {
            var ctrl = new WeatherForecastController(null);
            var result = ctrl.Get();

            Assert.IsNotNull(result);
        }
    }
```
## 上传代码到仓库
有了代码之后我们要把代码传到仓库里去。就是使用上面的仓库的地址、用户名、密码。这是git的问题了，不多说了。那么上面是一些准备工作，下面开始正式使用pipeline。
## 配置Pipeline
点击左侧的pipeline菜单，开始胚子pipeline的导航。    
第一步：需要配置代码仓库，选择刚才的Azure Repos Git。当然它还支持从Github或者别的地方拉代码。
![UyBUzD.png](https://s1.ax1x.com/2020/07/17/UyBUzD.png)
    
第二步：选择刚才的devop_test仓库。
![UyDukt.png](https://s1.ax1x.com/2020/07/17/UyDukt.png)
    
第三步：开始配置yml。这个yml呢其实跟docker-compose的配置啊，dockerfile啊一样，就是配置了一些列的任务（task）。
```
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
```
大概讲下这个yml配置了啥。   
trigger:表示代码的分支   
vmImage：表示虚拟机的环境，是win还是linux。   
variables：定义了一些参数，后面的设置可以直接使用。   
steps：步骤，里面每一个task就是一个步骤。    
task：    
    command: 'restore' nuget包还原。   
    command: 'build' 编译代码。   
    command: 'test' 运行单元测试。   
配置好yml之后点击“SAVE AND RUN”就会执行第一次pipeline的任务。运行之后任务会先进入队列，等待一会就能看到任务是否执行成功了。
![UyDt7n.png](https://s1.ax1x.com/2020/07/17/UyDt7n.png)
    
下面这图就表示任务执行成功了。每一步绿色的勾勾，还有执行了几秒都会显示出来。还可以看更加详细的日志。

![UyDWh6.png](https://s1.ax1x.com/2020/07/17/UyDWh6.png)
     
这个界面表示运行的单元测试的结果。成功了几个，是吧了几个，表示的都很清楚。
![UyD6B9.png](https://s1.ax1x.com/2020/07/17/UyD6B9.png)
    
成功之后你的账户邮箱还会收到邮件通知，成功是绿色的。
![Uy6B5R.png](https://s1.ax1x.com/2020/07/17/Uy6B5R.png)
    
前面都是成功的，我们故意把代码写个错误，然后直接提交代码，看看build能不能过。
```

        [HttpGet]
        public IEnumerable<WeatherForecast> Get()
        {
            var rng = new Random();
            return Enumerable.Range(1, 5).Select(index => new WeatherForecast
            {
                Date = DateTime.Now.AddDays(index),
                TemperatureC = rng.Next(-20, 55) //error ,去掉了一个逗号
                Summary = Summaries[rng.Next(Summaries.Length)]
            })
            .ToArray();
        }
```
改完代码后提交上去。可以看到任务会自己执行，然后过一会出现了一个红色的X。果然pipeline报错了。点击任务可以看到更加详细的错误列表。
[![Uy67xf.md.png](https://s1.ax1x.com/2020/07/17/Uy67xf.md.png)](https://imgchr.com/i/Uy67xf)
    
同时也受到了失败的邮件通知。
![Uy6qsS.png](https://s1.ax1x.com/2020/07/17/Uy6qsS.png)
    
## 总结
这次我们通过Azure DevOps Pipeline简单演示了CI的整个过程。我们成功实现了一开始订的小目标：写代码>提交代码>编译>运行测试>发送通知。除了yml配置有点麻烦，整过过程也都是很简单，而且是这个功能都是免费的。Azure DevOps pipeline除了CI，显然还能实现DI，如何编译docker镜像，如果推送镜像，如果部署镜像，那么请看下篇吧。
    
关注我的公众号一起玩转技术   
![](https://s1.ax1x.com/2020/06/29/NfQjds.jpg)