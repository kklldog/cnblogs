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
     
点击继续之后页面会跳转到正式的Azure DevOps界面。首先需要创建一个项目。这里跟Github一样，需要选择私有还有公开，估计Azure DevOps后端其实就是使用了Github的服务。这里选一个私有的吧，取个项目名称：devop_test 。
![UyBuzF.png](https://s1.ax1x.com/2020/07/17/UyBuzF.png)
    

![UyBXy4.png](https://s1.ax1x.com/2020/07/17/UyBXy4.png)
    
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
![UyBUzD.png](https://s1.ax1x.com/2020/07/17/UyBUzD.png)
    
![UyDukt.png](https://s1.ax1x.com/2020/07/17/UyDukt.png)
    
![UyDt7n.png](https://s1.ax1x.com/2020/07/17/UyDt7n.png)
    
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
![UyDWh6.png](https://s1.ax1x.com/2020/07/17/UyDWh6.png)
     
![UyD6B9.png](https://s1.ax1x.com/2020/07/17/UyD6B9.png)
    
![Uy6B5R.png](https://s1.ax1x.com/2020/07/17/Uy6B5R.png)
    
[![Uy67xf.md.png](https://s1.ax1x.com/2020/07/17/Uy67xf.md.png)](https://imgchr.com/i/Uy67xf)
    
![Uy6qsS.png](https://s1.ax1x.com/2020/07/17/Uy6qsS.png)
    