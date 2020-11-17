前几次我们演示了如何通过Azure静态web应用功能发布vue跟blazor的项目([使用 Azure静态web应用+Github全自动部署VUE站点](https://www.cnblogs.com/kklldog/p/azure-static-webapp-vue.html)、[使用Azure静态Web应用部署Blazor Webassembly应用](https://www.cnblogs.com/kklldog/p/staticwebapp-blazor.html))。但是一个真正的web应用，总是免不了需要后台api服务为前端提供数据或者处理数据的能力。同样前面我们也介绍了Azure函数服务，Azure函数的http trigger可以对http作出响应，可以完美的承当web api的角色。现在Azure静态web应用可以直接集成Azure函数，使得一次发布可以同时发布前端项目（vue、blazor）及后台api服务（azure函数）。
## 新建Azure函数
上次已经演示过如何发布Blazor项目，这里不在啰嗦，直接找到我们上次的BlazorWebassembly项目的解决方案，添加一个Azure函数。   
[![DeQa1x.png](https://s3.ax1x.com/2020/11/18/DeQa1x.png)](https://imgchr.com/i/DeQa1x)   
Azure函数使用Http trigger。Http trigger可以对http请求作出响应，可以看成是一个webapi。    
[![DeQwjK.png](https://s3.ax1x.com/2020/11/18/DeQwjK.png)](https://imgchr.com/i/DeQwjK)   
新建完成之后修改Function1.cs类的代码为：
```
  public static class Function1
    {
        [FunctionName("sum")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            log.LogInformation("C# HTTP trigger function processed a request.");

            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            dynamic data = JsonConvert.DeserializeObject(requestBody);

            int a = data.a;
            int b = data.b;

            int c = a + b;

            return new OkObjectResult(c);
        }
    }
```
代码比较简单，通过读取request的body获取提交的a、b两个值，然后相加之后返回结果。
## 修改Blazor项目
我们开始修改上次的Blazor Webassembly项目。在首页上放置3个文本框及一个按钮。点击按钮的时候把其中两个文本框的值通过http传递到Azure函数中去得到返回值显示在第三个文本框内。
```
@page "/"
@inject HttpClient Http

<h1>Azure static web app with functions</h1>

A:
<input @bind="a" />+
B:
<input @bind="b" />=
<input @bind="c" />
<button @onclick="sum">求和</button>

@code{
    private int a;
    private int b;
    private string c;

    private async Task sum()
    {
        var result = await Http.PostAsJsonAsync("/api/sum", new
        {
            A = a,
            B = b
        });
        var sum = await result.Content.ReadAsStringAsync();

        c = sum;
    }
}

```
完成之后提交代码到github。想要了解Blazor的相关内容请阅读我的其他关于Blazor入门的文章。
[tag=Blazor](https://www.cnblogs.com/kklldog/tag/Blazor/)
## 配置静态web应用
打开azure portal，新建一个静态web应用资源，因为前面已经介绍过多次基本的新建过程，这里不在详细介绍。   
[![DeQdc6.png](https://s3.ax1x.com/2020/11/18/DeQdc6.png)](https://imgchr.com/i/DeQdc6)   
基本配置跟上次发布Blazor Webassembly应用一样，关键的不同在于API位置需要修改为我们上面新建的Azure函数的项目名称。以便Azure能够找到这个目录。配置好之后点击开始创建。
## 运行项目
静态web应用资源创建完成后会在github项目上自动添加一个workflow。等待这个workflow显示绿色完成之后就可以正式访问我们的web应用了。   
[![DeQDBD.png](https://s3.ax1x.com/2020/11/18/DeQDBD.png)](https://imgchr.com/i/DeQDBD)   
点击静态web应用资源的概述目录，找到url地址复制后在浏览器里打开：   
[![DeQBnO.png](https://s3.ax1x.com/2020/11/18/DeQBnO.png)](https://imgchr.com/i/DeQBnO)   
随便输入几个值，点击求和可以看到得到正确的结果。：）
## 总结
前两次我们演示了通过Azure静态web应用功能发布vue跟Blazor wasm项目。但是他们都是纯静态页面。一般实现一个真正的web应用还需要api服务。Azure静态web应用通过直接对Azure函数的支持简化了项目开发发布流程。我们开发一些简单的项目的时候可以直接使用Azure函数做为api服务，提交代码等待几秒就可以运行了。本来可能需要前后端代码分别部署一次，现在只需要提交一下代码等待几秒就可以运行了。有了云计算程序员真的越来越傻瓜了，笑哭。
## 关注我的公众号一起玩转技术   
![](https://s1.ax1x.com/2020/06/29/NfQjds.jpg)