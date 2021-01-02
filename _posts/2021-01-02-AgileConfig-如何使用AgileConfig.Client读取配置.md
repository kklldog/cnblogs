下面通过几个示例来演示下如何AgileConfig.Client来读取配置：
## asp.net core mvc下读取配置
mvc项目应该是目前使用最广泛的项目，同样它与AgileConfig.Client的集成最深入。下面来看看如何在mvc项目下使用AgileConfig.Client。
### 安装AgileConfig.Client
```
Install-Package AgileConfig.Client
```
当然第一步是使用nuget命令安装最新版的Client库。
### 修改appsettings.json
```
  "AgileConfig": {
    "appId": "test_app",
    "secret": "",
    "nodes": "http://agileconfig.xbaby.xyz:5000"
  }
```
AgileConfig.Client连接服务端需要一点必要的信息，我们把这些信息配置在appsettings.json文件里。节点的名称叫“AgileConfig”，里面配置了：   
1. appId 应用id
2. secret 应用密钥，没有的话留空
3. nodes 节点地址，如果有多个则使用英文逗号(,)分隔
    
### AddAgileConfig
修改program.cs文件：
```
public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((context, config) =>
            {
                //注入AgileConfig Configuration Provider
                config.AddAgileConfig();
            })
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                });
```
通过AddAgileConfig扩展方法注入AgileConfigProvider。AgileConfigProvider才是跟配置系统打交道的组件。如果你想要使用Client的实例进行读取配置，也可以手动实例化一个client然后通过AddAglieConfig的另外一个重载注入进去。
```
Host.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((context, config) =>
            {
                //注入AgileConfig Configuration Provider
                var client = new ConfigClient();
                config.AddAgileConfig(client);
            })
```
### 读取配置
通过以上的设置，其实后面的配置读取跟使用appsettings.json没什么区别了。
```
 public HomeController(
            ILogger<HomeController> logger, 
            IConfiguration configuration, 
            )
        {
            _logger = logger;
            _IConfiguration = configuration;
        }

  /// <summary>
        /// 使用IConfiguration读取配置
        /// </summary>
        /// <returns></returns>
        public IActionResult ByIConfiguration()
        {
            var userId = _IConfiguration["userId"];
            var dbConn = _IConfiguration["db:connection"];

            ViewBag.userId = userId;
            ViewBag.dbConn = dbConn;

            return View("Configuration");
        }
```
## 控制台下读取配置
当然了从本质上来说控制台项目跟mvc项目没啥区别。同样可以引入ConfigurationBuilder来注入ConfigClient。但是一般我们使用控制台可能是写个小工具，不用搞的这么复杂，直接new一个ConfigClient的实例是最直接的方法。
```
 static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");

            var appId = "test_app";
            var secret = "";
            var nodes = "http://agileconfig.xbaby.xyz:5000";
            //使用有参构造函数，手动传入appid等信息
            var client = new ConfigClient(appId, secret, nodes);

            Task.Run(async () =>
            {
                while (true)
                {
                    await Task.Delay(5000);
                    foreach (string key in client.Data.Keys)
                    {
                        var val = client[key];
                        Console.WriteLine("{0} : {1}", key, val);
                    }
                }
            });

            client.ConnectAsync();//如果不是mvc项目，不使用AddAgileConfig方法的话，需要手动调用ConnectAsync方法来跟服务器建立连接

            Console.WriteLine("Test started .");
            Console.Read();
```
> 需要注意的一个地方是手工new ConfigClient是需要自己调用ConnectAsync方法进行连接服务器的。

## WPF程序读取配置
跟控制台程序一样，WPF同样首选直接new一个ConfigClient实例比较简单易用。
```
    public partial class App : Application
    {
        public static IConfigClient ConfigClient { get; private set; }
        private void Application_Startup(object sender, StartupEventArgs e)
        {
            //跟控制台项目一样，appid等信息取决于你如何获取。你可以写死，可以从配置文件读取，可以从别的web service读取。
            var appId = "test_app";
            var secret = "";
            var nodes = "http://agileconfig.xbaby.xyz:5000";
            ConfigClient = new ConfigClient(appId, secret, nodes);

            ConfigClient.ConnectAsync().GetAwaiter();
        }
    }
```
实例化的位置可以选在App文件的Application_Startup方法内。并且把实例直接挂到App类的静态变量上。
> 注意：Application_Startup方法是同步方法。调用ConnectAsync之后需要调用GetAwaiter()方法等待连接成功。
### 在窗体程序内读取配置
```
 private void Window_Loaded(object sender, RoutedEventArgs e)
        {
            this.tbx1.Text = App.ConfigClient["userId"];
            this.tbx2.Text = App.ConfigClient["connection"];
        }
```
我们通过直接访问App类上的ConfigClient对象读取配置信息。
