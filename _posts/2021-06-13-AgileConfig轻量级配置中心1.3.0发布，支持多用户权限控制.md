AgileConfig 当初是设计给我自己用的一个工具，所以只设置了一道管理员密码，没有用户的概念。但是很多同学在使用过后都提出了需要多用户支持的建议。整个团队或者整个公司都使用同一个密码来管理非常的不方便。   
今天 AgileConfig 1.3.0 版本终于支持了多用户，以及简单的权限管理。用户跟权限的设计，在我们开发管理系统的时候经常涉及，最常用的就是RBAC基于角色的权限控制。但是基于 AgileConfig 简单的理念，我稍微简化了一点权限控制的功能设计，尽量的降低学习成本。   
## 权限设计
AgileConfig 的权限设计分为3个固定的角色：
1. 超级管理员   
超级管理员具有一切的控制权限，可以随意添加修改删除用户、应用、配置等等任何信息
2. 管理员   
普通管理员可以新建应用，可以删除修改属于他的应用（应用的管理员属性为当前用户），以及该应用的配置项。管理员可以给任何用户授权所属应用配置项的管理权限。管理员可以添加修改删除角色为操作员的用户。
3. 操作员   
操作员对应用没有任何控制权限，只能编辑或者发布下线经过管理员授权的应用的配置项。

## 用户管理
1.3.0 版本新增了多用户支持，那么用户管理是必须的功能。
![](https://ftp.bmp.ovh/imgs/2021/06/2e2e689a3030daea.png)   
使用管理员级别的用户登录系统后，点击“用户”=>“添加”按钮弹出用户新增界面。
![](https://ftp.bmp.ovh/imgs/2021/06/18e98c199188529e.png)   
添加“用户名”、“密码”、团队等基本信息后，选择用户的角色。点击“确定”新建用户。提示成功后就可以使用该用户登录系统了。
## 应用授权
1.3.0 版本支持对用户进行简单的授权管理。
![](https://ftp.bmp.ovh/imgs/2021/06/fa386cbc975cf819.png)   
管理员在新建/编辑应用的时候可以维护一个管理员角色的用户。该账号对该应用具有完全的控制权限。
![](https://ftp.bmp.ovh/imgs/2021/06/af3f1c361f5a48b9.png)   
如果想要其它用户来编辑配置项，可以在授权界面进行授权。点击“授权”按钮弹出授权界面。
![](https://ftp.bmp.ovh/imgs/2021/06/54ffbb21f31d2d17.png)   
权限分为两部分：   
1. 配置修改权：配置项的改删查权限
2. 配置上下线权：配置项的上线，下线权限。   

## 升级需要更新的数据库结构
由于1.3加入了多用户的支持，新增了几张表跟字段，导致1.2升级1.3后程序运行报错的问题，需要手工调整表结构。   
以下以mysql为例：
1. agc_app表新增字段 app_admin varchar(36) 
2. 新建agc_user表
```
CREATE TABLE `agc_user` (
  `id` varchar(36) NOT NULL,
  `user_name` varchar(50) DEFAULT NULL,
  `password` varchar(50) DEFAULT NULL,
  `salt` varchar(36) DEFAULT NULL,
  `team` varchar(50) DEFAULT NULL,
  `create_time` datetime(3) NOT NULL,
  `update_time` datetime(3) DEFAULT NULL,
  `status` enum('Normal','Deleted') NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
3. 新建agc_user_app_auth表
```
CREATE TABLE `agc_user_app_auth` (
  `id` varchar(36) NOT NULL,
  `app_id` varchar(36) DEFAULT NULL,
  `user_id` varchar(36) DEFAULT NULL,
  `permission` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```
4. 新建agc_user_role表
```
CREATE TABLE `agc_user_role` (
  `id` varchar(36) NOT NULL,
  `user_id` varchar(50) DEFAULT NULL,
  `role` enum('SuperAdmin','Admin','NormalUser') NOT NULL,
  `create_time` datetime(3) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

```
新建完成表跟字段后重新运行程序，会提示重置超级管理员密码,之后就可以正常使用了。

## 最后

✨✨✨Github地址：[https://github.com/kklldog/AgileConfig](https://github.com/kklldog/AgileConfig)  开源不易，欢迎star✨✨✨   

演示地址：[AgileConfig Server Demo](http://agileconfig.xbaby.xyz:5000)  超级管理员账号：admin 密码：123456   

## 关注我的公众号一起玩转技术   
![](https://s1.ax1x.com/2020/06/29/NfQjds.jpg)
