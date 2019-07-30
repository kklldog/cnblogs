---
layout: default
title:  "sql server in docker"
---
现在.net core已经跨平台了，大家也都用上了linux用上了docker。跟.net经常配套使用的SQL SERVER以前一直是windows only，但是从SQL Server 2017开始已经支持运行在docker上，也就说现在SQL Serer已经可以运行在linux下了。   
下面在Ubuntu 16.4上演示安装并使用SQL Server 2019-CTP3.2
## SQL Server in Docker
```
sudo docker pull mcr.microsoft.com/mssql/server:2019-CTP3.2-ubuntu
```
*使用docker pull命令从docker hub拉取sqlserver 2019-ctp3.2的镜像*   
![](http://images.cnblogs.com/cnblogs_com/kklldog/1401672/o_QQ%E6%88%AA%E5%9B%BE20190725235741.png) 
```
sudo mkdir /hd2/sqlserver2019_data
sudo docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=dev@123," -p 14330:1433 --name sqlserver2019 -v /hd2/sqlserver2019_data:/var/opt/mssql  -d mcr.microsoft.com/mssql/server:2019-CTP3.2-ubuntu
```
*使用docker run 命令启动容器，其中要注意的是使用-v参数指定了sqlserver2019_data目录挂载到容器的/var/opt/mssql目录，这个目录是用来存储数据库文件的，所以最好挂载到外容器外部，避免因为不小心删除容器而丢失数据*   
![](http://images.cnblogs.com/cnblogs_com/kklldog/1401672/o_QQ%E6%88%AA%E5%9B%BE20190726000058.png)
```
sudo docker ps -a
```
*使用docker ps 命令查看容器运行情况，可以看到sqlserver2019正在运行*   
![](http://images.cnblogs.com/cnblogs_com/kklldog/1401672/o_QQ%E6%88%AA%E5%9B%BE20190726000119.png)
##使用命令行连接SQL Server
```
sudo docker exec -it sqlserver2019 "bash"
```
*使用docker exec命令登录到容器内部执行命令*
```
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P "dev@123,"
```
*在容器内部执行命令，打开sqlcmd*
打开sqlcmd之后我们就可以进行一些数据库的操作了，比如创建数据库，创建表,查询数据等。
```
CREATE DATABASE TEST_DB
GO
USE TEST_DB
GO
CREATE TABLE Table1 (ID INT, NAME NVARCHAR(50))
GO
Insert Into Table1 Values (0, 'agile')
```
*创建TEST_DB数据库;创建表Table1；插入一行数据；查询表数据*
![](http://images.cnblogs.com/cnblogs_com/kklldog/1401672/o_QQ%E6%88%AA%E5%9B%BE20190726004029.png)
我们使用docker运行的SQL Server同样可以使用Sql Server Management Studio来管理。  
![](http://images.cnblogs.com/cnblogs_com/kklldog/1401672/o_QQ%E6%88%AA%E5%9B%BE20190726000914.png) 
*使用服务器ip加端口连接成功后，可以看到刚才新建的数据库TEST_DB跟表TABLE1还有里面的数据都在。能使用SSMS管理后就简单多了跟使用SQL Server其他版本没啥区别。*   
![](http://images.cnblogs.com/cnblogs_com/kklldog/1401672/o_QQ%E6%88%AA%E5%9B%BE20190726004109.png) 
至此SQL Server in Docker的基本操作演示的差不多了，还有更多的高级功能比如配置故障转移集群，复制订阅，Always On等功能跟windows环境配置还有点区别大家可以自己尝试一下。