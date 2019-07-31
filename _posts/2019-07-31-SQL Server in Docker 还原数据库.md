# SQL Server in Docker 还原数据库
上一会演示了如果在Docker环境下安装SQL Server，这次我们来演示下如何还原一个数据库备份文件到数据库实例上。   
## 使用winscp上传bak文件到linux服务器
上一回我们启动docker容器的时候使用了-v参数挂账了本地目录/hd2/sqlserver2019_data到容器内目录/var/opt/mssql，所以我们只需要把文件testdb.bak上传到/hd2/sqlserver2019_data目录，docker容器即可访问。  
 ![](https://www.cnblogs.com/images/cnblogs_com/kklldog/1401672/o_TIM%e6%88%aa%e5%9b%be20190731144335.jpg)   
我使用了下Sql Server Management Studio的还原功能试了下，没有成功，不知是不是SSMS版本的问题。既然SSMS不能还原，那就使用命令行来试试吧。   
## 使用docker exec命令在容器内执行命令
因为SQL Server安装在Docker容器内，所以执行命令行都需要进入到容器内。   
```
sudo docker exec -it sqlserver2019 /bin/bash
```
![](https://www.cnblogs.com/images/cnblogs_com/kklldog/1401672/o_QQ%e6%88%aa%e5%9b%be20190801001331.png)
接下来的命令全部在sqlserver2019容器内执行。
## 使用RESTORE FILELISTONLY命令列出备份数据文件的逻辑名
```
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'dev@123,' -Q 'RESTORE FILELISTONLY FROM DISK = "/var/opt/mssql/testdb.bak"' | tr -s ' ' | cut -d ' ' -f 1-2
```
![](https://www.cnblogs.com/images/cnblogs_com/kklldog/1401672/o_TIM%e6%88%aa%e5%9b%be20190731154320.jpg)
使用该命令可以把数据库的数据文件，日志文件名称显示出来。在接下来的恢复操作中有用。
## 使用RESTORE DATABASE命令还原数据库
```
/opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P 'dev@123,' -Q 'RESTORE DATABASE testdb FROM DISK = "/var/opt/mssql/testdb.bak" WITH MOVE "testdb" TO "/var/opt/mssql/data/testdb.mdf" , MOVE "testdb_log" TO "/var/opt/mssql/data/testdb.ldf"'
```   
![](https://www.cnblogs.com/images/cnblogs_com/kklldog/1401672/o_TIM%e6%88%aa%e5%9b%be20190731154342.jpg)   
![](https://www.cnblogs.com/images/cnblogs_com/kklldog/1401672/o_TIM%e6%88%aa%e5%9b%be20190731154432.jpg)   
看到RESTORE DATABASE successfully的时候表示数据库还原成功了。让我们使用SSMS看看数据库是否真的还原成功了。
![](https://www.cnblogs.com/images/cnblogs_com/kklldog/1401672/o_TIM%e6%88%aa%e5%9b%be20190731154250.jpg)   
可以看到数据库已经还原上去，里面的表，数据都可以正常操作。至此，数据库文件还原成功。