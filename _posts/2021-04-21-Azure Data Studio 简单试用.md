最近折腾 azure sql database 的时候发现了微软的一款新的数据库管理工具： azure data studio。从名字上看 azure data studio 好像是专门为 azure 开发的，其实并不是这样的 。它同样支持对传统sql server的查询于管理。   
azure data studio 是一款跨平台数据库管理工具，支持 windows，macos，linux 。azure data studio 提供现代化的编辑体验，支持智能提示，代码补全，源代码版本管理等功能。它内建了图像画查询结果集，客户化首页等功能。
## 安装

```
https://docs.microsoft.com/en-us/sql/azure-data-studio/download-azure-data-studio?view=sql-server-ver15
```
从以上地址下载azure data studio的安装包，进行安装。   
## 试用
![](https://ftp.bmp.ovh/imgs/2021/04/dcce0bd629da9349.png)   
安装完成之后运行 azure data studio。可以看到跟 vscode 长的简直一模一样，可以推断 azure data studio 是基于 vscode 开发的。   
![](https://ftp.bmp.ovh/imgs/2021/04/9dd6fc80d3e0c0e9.png)   
要连接数据库，我们首先要添加一个连接。点击“Add Connection”按钮，弹出新建连接对话框。   
填写服务器地址，登录方式，账号密码，点击“连接”。   
![](https://ftp.bmp.ovh/imgs/2021/04/3330b1e5192af77a.png)   
如果成功登录到服务器，左侧会显示数据库列表。右侧会显示服务器的基本信息，以及一些数据库的基本信息。   
从上图中可以看到我们的服务器OS是Ubuntu16.04，sqlserver版本是 14.0.3162.1  Developer Edition 。   
![](https://ftp.bmp.ovh/imgs/2021/04/2ce1ade8610b3937.png)   
点开左侧菜单中的一个数据库实例，出现Tables，Views等文件夹，继续点开会出现表列表，视图列表等。这个跟SSMS大同小异。右键一张表，弹出快捷菜单，有一些常用功能，于SSMS同样大同小异。   
![](https://ftp.bmp.ovh/imgs/2021/04/d9c3e9d1470be2bc.png)   
按快捷CTRL+N新建一个查询，在这个页面可以编写SQL语句进行查询。编写的时候支持智能提示，这个智能提示的感觉比SSMS要厉害，支持中间字符的智能提示，而且速度很快。   
点击“RUN”可以执行查询，下面会出现查询的结果。
![](https://ftp.bmp.ovh/imgs/2021/04/4bf6d338f103de9e.png)   
## widget
azure data studio 还可以添加一些 Widget 来显示一些自定义信息。比如显示5个慢查询。
![](https://ftp.bmp.ovh/imgs/2021/04/b945380df75ab891.png)  
按CTRL+P打开指令框，输入 > settings 过滤选项。选择首选项。 
![](https://ftp.bmp.ovh/imgs/2021/04/e9f8300d35a2d57d.png)   
找到Dashboard>Database : Widgets    
在打开的json内容追加以下内容：
```
    "dashboard.database.widgets": [{
        "name": "slow queries widget",
        "gridItemConfig": {
            "sizex": 2,
            "sizey": 1
        },
        "widget": {
            "query-data-store-db-insight": null
        }
    }
    ],
    "workbench.colorTheme": "Default Dark Azure Data Studio",
    "dashboard.server.properties": true,
    "workbench.enablePreviewFeatures": true,
    "workbench.startupEditor": "welcomePage"
}
```
![](https://ftp.bmp.ovh/imgs/2021/04/e7701d471ad6c5cf.png)   
右键数据库选择“Manage” 弹出widgets界面。可以看到slow queries widget 显示出来了，显示的是最近5个慢查询。   
![](https://ftp.bmp.ovh/imgs/2021/04/cd0ee53ab97c757a.png)   
点击右上角的三个点，可以查看详情。   
## 插件
azure data studio 于 vscode 类似，支持安装插件。
![](https://ftp.bmp.ovh/imgs/2021/04/930b46442090ffb6.png)   
CTRL+SHIFT+X 打开插件搜索目录。可以看到有很多插件可以选。可以安装语言包，可以安装主题等。
![](https://ftp.bmp.ovh/imgs/2021/04/a127331353497c13.png)   
有个比较有意思的插件“Server Report”可以显示服务器当前的负载等情况。
## 总结
azure data studio 简单的试用了下。它非常轻量级，能够胜任基本的查询分析任务。它更偏向于sql语句的编辑器，还跟git有良好的集成。它还支持插件，widget 等组件可以在首页直接展示数据库的一些状态。但是它缺乏一些高级的数据库管理功能，比如你要做数据库复制订阅等操作它就不支持。当你只是像找个sql编辑查询工具可以考虑azure data studio ，而且它跨平台。