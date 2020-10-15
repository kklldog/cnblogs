Azure Cosmos DB 是 Microsoft 提供的全球分布式多模型数据库服务。Cosmos DB是一种NoSql数据库，但是它兼容多种API。它支持SQL, MongoDB、Cassandra或 Gremlin，你可以挑选自己喜欢的方式进行存储跟访问。
## 主要优势
### 统包式全局分发
凭借 Cosmos DB，你可以在全球范围内生成具有高响应性和高可用性的应用程序。 无论用户身处何处，Cosmos DB 均可以透明方式复制数据，因此用户可以与离他们最近的数据副本进行交互。
凭借 Cosmos DB，还可以随时向 Cosmos 帐户添加或删除任何 Azure 区域，只需单击一个按钮即可。 Cosmos DB 将无缝地将数据复制到与 Cosmos 帐户相关联的所有区域，同时，得益于该服务的多导功能，应用程序将继续保持高可用性。 有关详细信息，请参阅全局分发一文。
### AlwaysOn
凭借与 Azure 基础结构和透明多主数据库复制的深度集成，Cosmos DB 可为读写操作提供 99.999% 的高可用性。 Cosmos DB 还提供以编程方式（或通过门户）调用 Cosmos 帐户的区域性故障转移的功能。 此功能有助于确保应用程序能够在发生区域性灾难时进行故障转移。
### 吞吐量和存储的弹性可伸缩性（全球范围内）
Cosmos DB 采用透明的水平分区和多主数据库复制设计，在全球范围内为读写操作提供了前所未有的弹性可伸缩性。 通过单个 API 调用即可在全球范围内从数千个请求/秒扩展到数亿个请求/秒，并且只需为所需吞吐量（和存储）付费。 此功能有助于处理工作负载中的意外峰值，而无需为意外峰值进行过度预配。 有关详细信息，请参阅 Cosmos DB 中的分区、容器和数据库上的预配吞吐量以及全局缩放预配的吞吐量。
保证第 99 个百分位为低延迟（全球范围内）
使用 Cosmos DB，可以生成响应迅速、具全球规模的应用程序。 凭借其新颖的多主数据库复制协议、免闩锁及优化了写入的数据库引擎，，Cosmos DB 可保证全球任意位置第 99 个百分位的读取（已编入索引）和写入延迟均低于 10 毫秒。 此功能可以为高响应能力的应用持续引入数据，并提供快速查询。
### 精确定义的多个一致性选择
在 Cosmos DB 中构建全球分布式应用程序时，不再需要在一致性、可用性、延迟和吞吐量之间进行极端的权衡。 Cosmos DB 的多主数据库复制协议经过精心设计，为一个直观的编程模型（其低延迟和高可用性适用于全球分布式应用程序）提供五个明确定义的一致性选择 - “强”、“有限过期”、“会话”、“一致前缀”和“最终” 。
### 无需架构或索引管理
对于全球分布式应用来说，让数据库架构和索引与应用程序架构保持同步尤其不便。 借助 Cosmos DB，则无需处理架构或索引管理。 数据库引擎完全与架构无关。 由于不需要架构和索引管理，因此迁移架构时也不必担心应用程序停用时间。 Cosmos DB 自动为所有数据编制索引，并可快速提供查询服务。
> 以上内容摘自[Azure Cosmos文档](https://docs.microsoft.com/zh-cn/azure/cosmos-db/introduction)
## 创建Cosmos DB资源
在portal控制面板找到Cosmos点击创建。
![whO8Qs.png](https://s1.ax1x.com/2020/09/18/whO8Qs.png)
跟别的资源一样填写一个账户名，选择一个离自己近的位置。API选择MongoDB API。Apply Free Tier Discount选择Apply。这样就能开启免费额度了。
> Cosmos DB的免费额度为：5G存储，400请求单位/秒。  

## 复制Mongodb连接字符串
[![whO5SH.png](https://s1.ax1x.com/2020/09/18/whO5SH.png)](https://imgchr.com/i/whO5SH)
左侧菜单选择“连接字符串”，复制主连接字符串内容，下面会用到。
## 使用Mongodb API操作数据库
因为Cosmos支持mongodb协议，所以我们操作Cosmos的时候直接把Cosmos当做mongodb来使用就可以。下面代码演示了如何使用nodejs的mongodb驱动来操作Cosmos DB。
```
var MongoClient = require('mongodb').MongoClient;
var assert = require('assert');
var ObjectId = require('mongodb').ObjectID;
var endpoint = 'mongodb://';

var collectionName = "students";
//新增一个json文档
var insert = function(db, callback) {
    db.collection(collectionName).insertOne( {
            "id": "S001",
            "lastName": "zhou",
            "birthday": "2019-09-09",
            "sex": "m",
            "classId": 0
        }, function(err, result) {
        assert.equal(err, null);
        console.log("Inserted a document into the students collection.");
        callback();
    });
    };
    //把collection里的数据都查出来
    var find = function(db, callback) {
        var cursor =db.collection(collectionName).find( );
        cursor.each(function(err, doc) {
            assert.equal(err, null);
            if (doc != null) {
                console.dir(doc);
            } else {
                callback();
            }
        });
        };
    //修改S001的lastName
    var update = function(db, callback) {
        var myquery = { "id": "S001" };
        var newvalues = { $set: {lastName: "li"} };
        db.collection(collectionName).updateOne(
            myquery,newvalues,
            function(err, results) {
            console.log(results);
            callback();
        });
        };
    //移除lastName为li的内容
    var remove = function(db, callback) {
            db.collection(collectionName).deleteMany(
                { "lastName": "li" },
                function(err, results) {
                    console.log(results);
                    callback();
                }
            );
            };
    MongoClient.connect(endpoint, function(err, client) {
        assert.equal(null, err);
        var db = client.db('school');
        insert(db, function() {
            console.log('insert success .');
            find(db, function() {
                console.log('find success .');
                update(db, function() {
                    console.log('update success .');
                    remove(db, function(){
                        console.log('remove success .');
                    })
                });
            })
        });
        });
```

## 总结
Azure Cosmos DB是微软基于Azure开发的一款NoSql数据库，它支持多种数据库API。比如按SQL方式查询，按MongoDB方式读写等。如果你有海量文档数据需要存储及查询，你可以把它存储在Azure Cosmos DB上，由Azure来为你提供低延时、高吞吐量以及高达99.999%的SLA服务，而你只需要挑选自己喜欢的方式来进行它完成自己的业务。
    
## 关注我的公众号一起玩转技术   
![](https://s1.ax1x.com/2020/06/29/NfQjds.jpg)