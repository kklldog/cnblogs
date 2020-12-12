上次我们介绍了如果使用Azure应用服务。我们通过Visual studio新建一个项目后手动编译发布代码。然后通过FTP上传我们的发布文件。整个过程跟我们手动发布项目到IIS上其实没啥差别。
这么操作有点繁琐，显然在这年头也有点过时了。这次我们来玩一下azure应用比较高级的持续部署。说高级其实也很简单，Azure现在跟github有比较深入的集成，只有通过鼠标点几下，立马就完成了。
话不多说，下面就演示下吧。
## 配置部署中心
如何新建应用服务因为前面讲过了就不在赘述了。直接从配置部署中心开始吧。
![rZ2jUI.png](https://s3.ax1x.com/2020/12/13/rZ2jUI.png)
点击侧边“部署中心”，在弹出的页面上选择“Github”。    
点击“继续”弹出github授权界面。
![rZ2XVA.png](https://s3.ax1x.com/2020/12/13/rZ2XVA.png)
点击“Authorize AzureAppService”同意授权。
![rZ2Lbd.png](https://s3.ax1x.com/2020/12/13/rZ2Lbd.png)
点击“下一步”配置生成提供程序，选择“Github Actions”。
![rZ2qDH.png](https://s3.ax1x.com/2020/12/13/rZ2qDH.png)
点击“下一步”弹出配置界面，这个页面可以选择github上的仓库。
我随便选一个以前提交上去的RazorpageCrudDemo吧。分支选择master。运行时堆栈选择：.net core，版本 .net core 3.1 lts 。
![rZ2bKe.png](https://s3.ax1x.com/2020/12/13/rZ2bKe.png)
点击“完成”会自动生成一个yml文件。这个yml配置的就是github的action workflow。我们的azure应用服务跟github就是通过它串起来的。
## 验证部署
![rZ2TgO.png](https://s3.ax1x.com/2020/12/13/rZ2TgO.png)
切换到github的actions页面。会发有一个build and deploy的job正在运行。
![rZ244x.png](https://s3.ax1x.com/2020/12/13/rZ244x.png)
等待这个job运行成功后，我们就可以访问azure应用服务的url地址了。
![rZ2fER.png](https://s3.ax1x.com/2020/12/13/rZ2fER.png)
访问一下azure应用服务对应的url，出现了asp.net core的默认页面。说明我们的部署成功了。
![rZ2hU1.png](https://s3.ax1x.com/2020/12/13/rZ2hU1.png)
在访问下里面的页面，也成功渲染出来了。
## 持续部署（CD）
```
@page
@model IndexModel
@{
    ViewData["Title"] = "Home page";
}

<div class="text-center">
    <h1 class="display-4">Azure App deploy with Github</h1>
    <p>Learn about <a href="https://docs.microsoft.com/aspnet/core">building Web apps with ASP.NET Core</a>.</p>
</div>
```
既然是持续部署（CD），那么我们尝试下修改项目的首页，然后提交代码，看会不会自动部署代码。
![rZ2o8K.png](https://s3.ax1x.com/2020/12/13/rZ2o8K.png)
提交完代码后，github的actions页面立马又出现了一个job。
![rZ2IC6.png](https://s3.ax1x.com/2020/12/13/rZ2IC6.png)
等待job完成之后，再次访问azure服务应用的url，果然首页变成了我们修改的样子，说明持续部署成功了。

## 关注我的公众号一起玩转技术   
![](https://s1.ax1x.com/2020/06/29/NfQjds.jpg)