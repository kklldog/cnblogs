## 测试写性能
我们的测试方案：新建一张logs表，按年份写入数据。2019年写入1000000数据，2020年也写入100000数据。为了加快写入的速度，每个年份并行10个线程同时写，每个线程写100000数据，一共1000000数据。然后把logs表改成分区表再用同样的方式写入1000000数据。记录耗时比较两次的耗时。
## 未分区情况下测试
使用脚本建表：
```
CREATE TABLE [dbo].[logs](
	[id] [uniqueidentifier] NOT NULL,
	[log_txt] [varchar](200) NULL,
	[log_time] [datetime] NULL,
 CONSTRAINT [PK_logs] PRIMARY KEY CLUSTERED 
(
	[id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, IGNORE_DUP_KEY = OFF, ALLOW_ROW_LOCKS = ON, ALLOW_PAGE_LOCKS = ON) ON [PRIMARY]
)
```
新建一个控制台程序编写代码：
```
class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");
            Task.Run(() =>
            {
                InsertData(2019);
            });
            Task.Run(() =>
            {
                InsertData(2020);
            });
            Console.ReadLine();
        }

        static void InsertData(int year)
        {
            var tasks = new List<Task>();
            Stopwatch sw = new Stopwatch();
            sw.Start();
            for (int i = 0; i < 10; i++)
            {
                tasks.Add(Task.Run(()=> {
                    using (var conn = new SqlConnection())
                    {
                        conn.ConnectionString = "Persist Security Info = False; User ID =sa; Password =dev@123; Initial Catalog =fq_test; Server =.\\mssql2016";
                        conn.Open();
                        int index = 0;
                        for (int j = 0; j < 100000; j++)
                        {
                            var logtime = new DateTime(year, new Random().Next(1, 12), new Random().Next(1, 28));
                            conn.Execute("insert into logs2 values (newid(),'下订单',@logtime)", new
                            {
                                logtime
                            });
                            Console.WriteLine("logtime:{0} index {1}", logtime, index++);
                        }
                    }
                }));
            }
            Task.WaitAll(tasks.ToArray());
            sw.Stop();
            Console.WriteLine("Year {0} complete , total time: {1}.", year, sw.ElapsedMilliseconds);
        }
    }
```
![y0cke1.png](https://s3.ax1x.com/2021/02/10/y0cke1.png)   
写完1000000数据耗时1369454毫秒。   
## 分区情况下进行测试
### 开始分区
把一个表设置为分区表大概有5个步骤：    
1. 添加文件组
2. 在文件组添加文件
3. 新建分区函数
4. 新建分区方案
5. 开始分区
    
以下演示下如何使用SQL SERVER Management Studio管理器进行表分区：
![y02dZn.png](https://s3.ax1x.com/2021/02/10/y02dZn.png)   
选中数据库=>属性=>文件组，添加group1，group2两个文件组。
![y02waq.png](https://s3.ax1x.com/2021/02/10/y02waq.png)   
选中数据库=>属性=>文件。添加file1，文件组选group1，路径选择一个文件目录。这里选择E盘data目录。添加file2，文件组选择group2，路径选择一个文件目录。这里选择X盘的data目录。这样当分区的时候数据就会落在这2个目录下。这里的路径可以选择在同一个硬盘，但是为了更高的读写性能，如果有条件建议直接指定在不同的硬盘下。   
![y0ciLR.png](https://s3.ax1x.com/2021/02/10/y0ciLR.png)   
选中logs表=>存储=>创建分区，启动分区向导工具。
![y0cCQJ.png](https://s3.ax1x.com/2021/02/10/y0cCQJ.png)   
新建一个分区函数，点击下一步。
![yyYBZT.png](https://s3.ax1x.com/2021/02/14/yyYBZT.png)   
新建一个分区方案，点击下一步。
![y0cPy9.png](https://s3.ax1x.com/2021/02/10/y0cPy9.png)   
选择一个分区列，数据会根据该列进行水平拆分。这里选择logtime，因为时间是比较适合水平切分的一个维度。   
![y0cpz4.png](https://s3.ax1x.com/2021/02/10/y0cpz4.png)   
值得数据拆分的范围。范围选择“右边界”。右边界跟左边界的差异在于对边界值的处理。右边界是<，左边界是<=，也就是包含边界值。   
我们这里设置group1存储2019的数据，group2存储2020的数据。所以group1的边界值设置为2020-01-01，group2的边界值设置为2021-01-01 。   
![y06zJU.png](https://s3.ax1x.com/2021/02/10/y06zJU.png)   
设置完是这个样子，需要3个文件组。当出现不在group1，group2范围内的数据就会存储在第三个文件组内。   
![y06joV.png](https://s3.ax1x.com/2021/02/10/y06joV.png)   
![y06Xd0.png](https://s3.ax1x.com/2021/02/10/y06Xd0.png)   
建好分区函数、分区方案后，可以选择生成脚本或者立即执行。这里选择“立即执行”。当执行完成后，表里的数据会按照分区方案设置的边界分散到多个文件上。   
### 在分区情况下进行测试
![y0cSWF.png](https://s3.ax1x.com/2021/02/10/y0cSWF.png)   
先清空logs表所有的数据，然后使用同样的代码进行测试。测试结果显示数据库写性能大副提高，大概提高了1倍的不止的性能。这也比较符合两块磁盘同时IO的预期。
## 测试读性能
我们的测试方案：新建一张log2表，使用上面的代码按年份写入2000000数据。然后使用select语句同时读取2019,2020年的数据。把log表转换成分区表，重新测试select的时间。比较两次读取数据的时间。   
sql语句：
```
select * from log2 where (logtime > '2019-05-01' and logtime < '2019-06-01') or (logtime > '2020-05-01' and logtime < '2020-06-01')
```
![y02E8O.png](https://s3.ax1x.com/2021/02/10/y02E8O.png)   
![y02APK.png](https://s3.ax1x.com/2021/02/10/y02APK.png)   
