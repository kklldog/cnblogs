AgileConfig自发布以来有个“大问题”-UI太丑。因为当初这个项目是给自己用的，连UI界面都没有，全靠手动在数据库里改配置。后来匆匆忙忙使用bootstrap3简单的码了一些界面就发布出来了，易用性上也做的不够好。对此我一直耿耿于怀。终于在过年期间动手翻新UI。   
对于一个后端程序员，标准的直男审美，想做出好看的UI几乎不可能。所以只能借助前端框架了。在经过一番考察后决定使用Ant-design-pro这个框架。Ant-design是当前最流行的前端组件库，Ant-design-pro是官方出品的一个基于Ant-design的admin后台快速开发框架。Ant-design基于react开发，本人没玩过react，也正好学习一下。   
在经过几个preview版本之后，今天release-1.2.0版本终于上线了。

## release-1.2.0
* 使用ant-design-pro重写了全部UI
* 支持英文国际化   
   
![](https://ftp.bmp.ovh/imgs/2021/04/44242b327230c5e6.png)   
![](https://ftp.bmp.ovh/imgs/2021/04/7e93011590c55d12.png)   
![](https://ftp.bmp.ovh/imgs/2021/04/a48014f02ced6804.png)   
![](https://ftp.bmp.ovh/imgs/2021/04/8ae7d8bfcef72518.png)
## AgileConfig 介绍
这是一个基于.net core开发的轻量级配置中心。说起配置中心很容易让人跟微服务联系起来，如果你选择微服务架构，那么几乎逃不了需要一个配置中心。事实上我这里并不是要蹭微服务的热度。这个世界上有很多分布式程序但它并不是微服务。比如有很多传统的SOA的应用他们分布式部署，但并不是完整的微服务架构。这些程序由于分散在多个服务器上所以更改配置很困难。又或者某些程序即使不是分布式部署的，但是他们采用了容器化部署，他们修改配置同样很费劲。所以我开发AgileConfig并不是为了什么微服务，我更多的是为了那些分布式、容器化部署的应用能够更加简单的读取、修改配置。    
AgileConfig秉承轻量化的特点，部署简单、配置简单、使用简单、学习简单，它只提取了必要的一些功能，并没有像Apollo那样复杂且庞大。但是它的功能也已经足够你替换webconfig，appsettings.json这些文件了。如果你不想用微服务全家桶，不想为了部署一个配置中心而需要看N篇教程跟几台服务器那么你可以试试AgileConfig  ：）   
![GitHub Workflow Status](https://img.shields.io/github/workflow/status/kklldog/agileconfig/.NET%20Core)
![GitHub stars](https://img.shields.io/github/stars/kklldog/AgileConfig)
![Commit Date](https://img.shields.io/github/last-commit/kklldog/AgileConfig/master.svg?logo=github&logoColor=green&label=commit)
![Nuget](https://img.shields.io/nuget/v/agileconfig.client?label=agileconfig.client)
![Nuget](https://img.shields.io/nuget/dt/agileconfig.client?label=client%20download)
![Docker image](https://img.shields.io/docker/v/kklldog/agile_config?label=docker%20image)
![GitHub license](https://img.shields.io/github/license/kklldog/AgileConfig)

## 特点
1. 部署简单，最少只需要一个数据节点，支持docker部署
2. 支持多节点分布式部署来保证高可用
3. 配置支持按应用隔离，应用内配置支持分组隔离
4. 应用支持继承，可以把公共配置提取到一个应用然后其它应用继承它
5. 使用长连接技术，配置信息实时推送至客户端
6. 支持IConfiguration，IOptions模式读取配置，原程序几乎可以不用改造
7. 配置修改支持版本记录，随时回滚配置
8. 如果所有节点都故障，客户端支持从本地缓存读取配置
9. 支持Restful API维护配置
    
✨✨✨Github地址：[https://github.com/kklldog/AgileConfig](https://github.com/kklldog/AgileConfig)  开源不易，欢迎star✨✨✨   

演示地址：[AgileConfig Server Demo](http://agileconfig.xbaby.xyz:5000)   密码：123456   

## 关注我的公众号一起玩转技术   
![](https://s1.ax1x.com/2020/06/29/NfQjds.jpg)
