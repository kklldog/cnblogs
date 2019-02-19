Docker可以说是现在微服务，DevOps的基础，咱们.Net Core自然也得上Docker。.Net Core发布到Docker容器的教程网上也有不少，但是今天还是想来写一写。   
你搜.Net core程序发布到Docker网上一般常见的有两种方案：
* 1、在本地编译成Dll文件后通过SCP命令或者WinSCP等工具上传到服务器上，然后构建Docker镜像再运行容器。该方案跟传统的发布很像，麻烦的地方是每次都要打开相关工具往服务器上复制文件。
* 2、在服务端直接通过Git获取最新源代码后编译成Dll然后构建Docker镜像再运行容器。该方案免去了往服务器复制文件这步操作，但是服务器环境需要安装.Net Core SDK 来编译源代码。   
**自从用了Docker简直懒的不能自理，我既不想手工复制文件到服务器，也不想在服务器装.Net Core环境。显然只要Docker镜像包含.Net Core SDK环境就可以在Docker内帮我们编译代码然后运行，这样连我们的服务器都不用装啥.Net Core的环境拉。**   
## 在Docker内编译.Net Core程序并运行
### 新建一个Asp.net Core MVC项目   
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
*修改HomeController下的index Action，直接输出一段文字*   
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
### 推送源码到代码仓库   
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
### 添加Dockerfile文件   
在CoreForDocker下新增一个Dockerfile文件，注意没有任何扩展名。我们需要基于microsoft/dotnet:latest这个镜像构建一个新的镜像。并且在构建的过程中直接对源码进行编译并发布。
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
**COPY /. /app**：*复制宿主机当前目录的内容到容器的app文件夹*   
**RUN dotnet restore**:*还原nuget包*   
**RUN dotnet publish -o /out -c Release**：*编译并发布程序集到容器的out目录*   
**EXPOSE 5000**:*暴露5000端口*   
**ENTRYPOINT ["dotnet", "/out/CoreForDocker.dll"]**:*容器启动的时候执行dotnet命令，参数为/out/CoreForDocker.dll*   
   
![](http://images.cnblogs.com/cnblogs_com/kklldog/1401672/o_201902196.jpg)
*Dockerfile的文件属性设置为始终复制*   
新建好Dockerfile后git push到代码仓库。   
### 在服务器上构建Docker镜像   
这里以Ubuntu为例，ssh登录到服务器后使用git clone命令拉取源代码。
```
git clone https://gitee.com/kklldog/CoreForDocker.git
```
进入源码目录
```
cd CodeForDocker\CodeForDocker
```
使用docker build命令构建新的镜像，注意不要忘记最后一个'.'
```
docker build -t image_code4docker .
```
### 运行容器  
如果以上步骤都没有报错，那么恭喜你镜像已经构建成功了，我们可以使用此镜像运行Docker容器了。
```
docker run -d --name code4docker -p 5000:5000 -v /ect/localtime:/ect/localtime image_core4docker
```
*使用image_core4docker镜像运行一个名为core4docker的容器，绑定宿主机的5000到容器的5000口。其中需要注意的是-v参数映射宿主机的/ect/localtime文件夹到容器的/ect/localtime文件夹，因为经过实践发现容器中的时区有可能跟宿主机不一致，需要映射宿主机的/ect/localtime让容器的时区跟宿主机保持一致。*
     
![](http://images.cnblogs.com/cnblogs_com/kklldog/1401672/o_20190219160556.png)   
*访问一下服务器的5000端口，发现能够正确返回数据表示我们的Asp.net Core程序在容器中运行成功了*  
   
以后当我们对源码进行修改，并提交后，我们只需在服务器上拉取最新的代码然后使用docker build，docker run命令来再次生成镜像并运行容器。但是手工输入docker build，docker run的命令好像也很麻烦，参数又那么多，太烦了。
### 使用shell脚本简化操作   
为了偷懒不想敲那么长的命令，我们可以构建一个脚本，把命令一次性写好，以后只要运行一次脚本就可以了。   
使用vim新建一个publish.sh的文件
```
vim publish.sh
```
键盘上按i进入编辑模式，输入以下内容
```
cd CoreForDocker/CoreForDocker
git pull
docker stop core4docker
docker rm core4docker
docker rmi image_core4docker
docker build -t image_core4docker .
docker run --name core4docker -d -p 5000:5000 -v /etc/localtime:/etc/localtime image_core4docker

```
*以上命令，不光有新建镜像跟运行容器的命令，还有移除原来的容器跟镜像的命令*   
按ecs进入命令模式，退出保存
```
:wq
```
让我们模拟修改一下源代码，并提交到代码仓库
```
    public IActionResult Index()
    {
        return Content($"Core for docker , {DateTime.Now} , version 2");
    }
```
*再次修改homecontroller的index action，输出内容上新增一个version*   
ssh登录到服务器，运行publish.sh文件
```
/bin/bash publish.sh
```
![](http://images.cnblogs.com/cnblogs_com/kklldog/1401672/o_TIM%E5%9B%BE%E7%89%8720190219162655.png)   
*跑完之后我们再次访问下服务器的5000口，数据返回正确，表示服务器上跑的已经是最新的程序了*
### 总结
通过以上演示我们基本了解如何通过git跟docker配合在Ubuntu服务器上不安装.Net Core SDK来发布.Net Core 程序到容器中运行，并且通过shell脚本的方式再次简化发布。但是尽管这样每次发布都需要ssh到服务器上然后运行脚本，特别是开发环境可能经常需要发布，还是觉得麻烦。有没有什么办法让我们push代码后服务器自动就开始部署最新的代码的到容器中运行了呢？   
后面我会介绍下如何通过jenkins跟webhook来做CICD。