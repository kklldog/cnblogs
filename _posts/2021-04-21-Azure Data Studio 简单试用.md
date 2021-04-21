
```
https://docs.microsoft.com/en-us/sql/azure-data-studio/download-azure-data-studio?view=sql-server-ver15
```
从以上地址下载azure data studio的安装包，进行安装。   
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
![](https://ftp.bmp.ovh/imgs/2021/04/b945380df75ab891.png)   
![](https://ftp.bmp.ovh/imgs/2021/04/e9f8300d35a2d57d.png)   
![](https://ftp.bmp.ovh/imgs/2021/04/e7701d471ad6c5cf.png)   
![](https://ftp.bmp.ovh/imgs/2021/04/cd0ee53ab97c757a.png)   
![](https://ftp.bmp.ovh/imgs/2021/04/930b46442090ffb6.png)   
![](https://ftp.bmp.ovh/imgs/2021/04/a127331353497c13.png)   