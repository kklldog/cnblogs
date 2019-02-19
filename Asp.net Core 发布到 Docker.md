Docker可以说是现在微服务，DevOps的基础，咱们.Net Core自然也得上Docker。.Net Core发布到Docker容器的教程网上也有不少，但是今天还是想来写一写。   
.Net core程序发布到Docker一般常见的有两种方案：
* 1、在本地编译成Dll文件后通过SCP命令或者WinSCP等工具上传到服务器上，然后构建Docker镜像再运行容器。该方案跟传统的发布很像，麻烦的地方上每次都要打开相关工具往服务器上复制文件。
* 2、在服务端直接通过Git获取最新源代码后编译成Dll然后构建Docker镜像再运行容器。该方案免去了往服务器复制文件这步操作，但是服务器环境需要安装.Net Core SDK 来编译源代码。   
自从用了Docker简直懒的不能自理，我现在连在服务器装.Net Core环境都懒的装。显然只要Docker镜像包含.Net Core SDK环境就可以在Docker内帮我们编译代码然后运行，这样连我们的服务器都不用装啥.Net Core的环境拉。   
### 在Docker内编译.Net Core程序并运行
#### 新建一个Asp.net Core MVC项目
我们使用一个Asp.net Core MVC程序来演示如何发布到Docker并运行。
![新建项目](http://images.cnblogs.com/cnblogs_com/kklldog/1401672/o_20190219.jpg)   
*使用vs新建一个Asp.net core mvc项目*
```
  public class HomeController : Controller
    {
        public IActionResult Index()
        {
            return Content($"Core for docker , {DateTime.Now} , verson 2");
        }
    }
```
*修改HomeController下的index Action*   
```
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
            WebHost.CreateDefaultBuilder(args)
            .UseKestrel(op =>
            {
                op.ListenAnyIP(5000);
            })
            .UseStartup<Startup>();
```
*修改Program下的CreateWebHostBuilder方法，让Kestrel监听5000端口*
![](http://images.cnblogs.com/cnblogs_com/kklldog/1401672/o_201902192.jpg)   
*本地运行一下试试*   
#### 推送源码到代码仓库
把我们的代码推送到对应的Git仓库，方便我们从部署服务器上直接拉取最新的代码。
```
X:\workspace\CoreForDocker>git remote add origin https://gitee.com/kklldog/CoreForDocker.git

X:\workspace\CoreForDocker>git push -u origin master
Username for 'https://gitee.com': xxx@gmail.com
Password for 'https://xxx@gmail.com@gitee.com':
Counting objects: 88, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (83/83), done.
Writing objects: 100% (88/88), 527.07 KiB | 2.43 MiB/s, done.
Total 88 (delta 7), reused 0 (delta 0)
remote: Powered By Gitee.com
To https://gitee.com/kklldog/CoreForDocker.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'origin'.
```
#### 构建Docker镜像
在CoreForDocker下新增一个Dockerfile文件，注意没有任何扩展名。我们需要基于microsoft/dotnet:latest这个镜像构建一个新的镜像。
```
FROM microsoft/dotnet:latest
WORKDIR /app
COPY /. /app
RUN dotnet restore
RUN dotnet publish -o /out -c Release
EXPOSE 5000
ENTRYPOINT ["dotnet", "/out/CoreForDocker.dll"]
```
大概解释下Dockerfile的意思：   
**FROM microsoft/dotnet:latest**:*使用dotnet的最新镜像，这个镜像其实对应的应该就是2.2-sdk这个镜像，里面包含了dotnet-core 2.2 sdk*   
**WORKDIR /app**:*指定工作目录为app*   
**COPY /. /app**：**
![](http://images.cnblogs.com/cnblogs_com/kklldog/1401672/o_201902196.jpg)
*Dockerfile的文件属性设置为始终复制*   

