上一次演示了如何使用Azure静态web应用部署VUE前端项目（[使用 Azure静态web应用+Github全自动部署VUE站点](https://www.cnblogs.com/kklldog/p/azure-static-webapp-vue.html)）。我们知道静态web应用支持VUE，react，angular等项目的部署。除了支持这些常见前端框架，静态web应用同样支持微软推出的最新的前端框架Blazor Webassembly。今天就来演示下如何通过静态web应用部署Blazor项目。
## 新建blazor项目
使用VS新建一个blazor项目，因为是演示项目所以啥都不用改。
![B0sagU.png](https://s1.ax1x.com/2020/11/01/B0sagU.png)   
项目名称：WebStaticAppp_Blazor，完成新建。
![B0sU3T.png](https://s1.ax1x.com/2020/11/01/B0sU3T.png)   
## 新建github仓库
我们把代码存放在github上，所以需要新建一个空repository。仓库名称命名为staticwebapp_balzor。
![B0sJNq.png](https://s1.ax1x.com/2020/11/01/B0sJNq.png)   
回到上面创建的blazor项目，把代码推送到github仓库。推送成功后目录结构如下：
![B0sY40.png](https://s1.ax1x.com/2020/11/01/B0sY40.png)   
## 新建静态web应用
在azure portal找到静态web应用（预览），点击创建弹出创建资源界面：
![B0sNCV.png](https://s1.ax1x.com/2020/11/01/B0sNCV.png)   
名称：staticwebapp-blazor    
区域：选个离你近的    
SKU：免费   
### 登录Github账号
在源代码管理信息界面点击“使用Github登录”，弹出Github授权页面，确认授权。   
![B0sluQ.png](https://s1.ax1x.com/2020/11/01/B0sluQ.png)   
授权成功后就可以选择刚才创建的仓库。   
储存库：staticwebapp_blazor。   
分支：master。   
生成预设；Blazor。   
应用位置：WebStaticApp_Blazor。   
API位置：默认。因为我们没有部署api，所以默认不用管他。   
应用项目位置：wwwroot。   
最后点击查看创建。等待创建资源，过一会portal会提示资源创建成功。   
![B0sMjg.png](https://s1.ax1x.com/2020/11/01/B0sMjg.png)   
资源创建成功后，我们打开github上的项目，点击Actions，可以看到Azure Static Web App CI/CD这个job正在运行。等到这个job提示绿色对勾的时候就表示执行成功了。   
![B0s3Hs.png](https://s1.ax1x.com/2020/11/01/B0s3Hs.png)   
返回portal查看刚新建的静态web应用，点击概述，查看URL。   
![B0vdDe.png](https://s1.ax1x.com/2020/11/02/B0vdDe.png)   
把URL贴到浏览器里访问一下，熟悉的Blazor默认项目首页显示出来了。   
![B0vcgf.png](https://s1.ax1x.com/2020/11/02/B0vcgf.png)   
我们把首页修改一下：然后推送到仓库。    
```
@page "/"
<h1>Azure static web app with BLAZOR .</h1>
```
推送成功后，仓库的actions会立马执行新的CI/CD任务，等到提示成功后，再次访问一下上面的URL，界面已经变化为我们修改的样式，说明部署成功了。   
![B0sGEn.png](https://s1.ax1x.com/2020/11/01/B0sGEn.png)   
## 总结
通过简单的演示，我们熟悉了如何使用Azure静态web应用来部署blazor项目。流程上同部署VUE几乎一致，就是预设模板那里需要选择blazor而已，相当方便。当然了只有前端界面没有api服务是无法真正用来生产的，下一次我们演示下如何使用Azure静态web应用集成并调用Azure Functions 。
## 关注我的公众号一起玩转技术   
![](https://s1.ax1x.com/2020/06/29/NfQjds.jpg)