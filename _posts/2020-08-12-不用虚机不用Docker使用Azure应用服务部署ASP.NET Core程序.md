一般我们写好了应用程序想要部署发布它，要么发布到物理机，要么发布到虚拟机，要么发布到容器来运行它。现在有了Azure应用服务，我们可以完全不用管这些东西，只管写好自己的代码，然后使用VisualStudio的发布功能就可以一键部署了。如果你觉得性能不够用了还可以自动扩容，弹性伸缩。
## 应用服务概述
Azure 应用服务是一项基于 HTTP 的服务，用于托管 Web 应用程序、REST API 和移动后端 。 可以使用 .NET、NET Core、Java、Ruby、Node.js、PHP 或 Python 等偏好的语言进行开发。
应用服务不仅可将 Microsoft Azure 的强大功能（例如安全性、负载均衡、自动缩放和自动管理）添加到应用程序。 还可以利用其 DevOps 功能，例如包管理、过渡环境、自定义域和 SSL 证书。
> 引用自[微软Azure文档](https://docs.azure.cn/zh-cn/app-service/overview)
## 创建应用服务资源
![avUSmj.png](https://s1.ax1x.com/2020/08/12/avUSmj.png)   
通过portal控制台创建一个新的应用服务资源。   
取个名字，这个名字会分配一个二级域名，到时候可以通过它来访问你的应用程序。选择对于的运行时，操作系统，区域。应用服务对于12月免费账号也是一个免费服务，支持1G内存60分钟CPU时间/天10个实例。
> 注意：SKU和大小，这里默认是要收费的，需要改成对应的免费计划。 
![avNv6g.png](https://s1.ax1x.com/2020/08/12/avNv6g.png)   
点击“更改大小”，选择开发/测试标签，选择F1定价计划，这个才是免费的。
![avU97n.png](https://s1.ax1x.com/2020/08/12/avU97n.png)
这些设置完成后点击创建，等待一会就会提示资源创建完成。选择新建的资源，可以看到一些基本信息，以及一些输入、输出的监控信息等。
## 创建ASP.NET Core应用程序
![avcVmV.png](https://s1.ax1x.com/2020/08/12/avcVmV.png)   
打开VisualStudio新建一个ASP.NET Core应用程序，我们只是演示，啥都不用改。
## 发布程序
![avUJje.png](https://s1.ax1x.com/2020/08/12/avUJje.png)   
有了新建的.net程序，我们要发布它到Azure应用服务上去。在VS上选择发布，弹出发布界面。。选择"IIS,FTP等"选项。   
VS其实跟Azure有深度的集成，其实直接支持应用服务的发布，但是因为网络的问题，我没有连接成功，Microsoft账号这么都登录不上，所以只好改用FTP发布。
![avUp0s.png](https://s1.ax1x.com/2020/08/12/avUp0s.png)   
回到portal门户，选择“部署中心>FTP”
![avNxXQ.png](https://s1.ax1x.com/2020/08/12/avNxXQ.png)   
复制好FTPS终结点，用户名密码。
![avUPkq.png](https://s1.ax1x.com/2020/08/12/avUPkq.png)   
回到VS的发布界面，填写上一步获得的FTP信息，点击保存。
![avUit0.png](https://s1.ax1x.com/2020/08/12/avUit0.png)   
点击发布按钮，VS会开始编译代码然后发布代码到指定的FTP位置，最后提示发布成功。
## 访问应用服务
上面提到了新建资源的时候需要填写名称，这个名称加上.azurewebsites.net就是服务对应的地址。让我们访问一下吧。
![avUFhV.png](https://s1.ax1x.com/2020/08/12/avUFhV.png)   
可以看到我们的asp.net core应用程序发布成功了，示例代码可以运行了。
## 总结
通过上面的演示，我们没有通过任何虚机、Docker、K8S等东西就把我们的asp.net core应用程序给跑起来了。而且还是通过VS直接发布的，不需要借助任何其他工具，真的非常方便。而且它还支持自动扩容，弹性伸缩等特性只要动动鼠标就可以完成，这让开发更加专注于代码，不会被运维等内容困扰。Azure应用服务是一个非常棒的功能。