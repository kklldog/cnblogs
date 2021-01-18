## Restful API
为了更加方便的跟业务系统集成最新版的AgileConfig已支持json格式的 restful api来维护配置 。   
本API入参跟出参为json格式，所以请求的时候需设置Content-Type头部为application/json 。    
使用basic简单认证，设置Authorization头部为Basic base64(userName:password) 。    
当操作节点、应用api的时候basic认证的userName固定设置为admin，password为当前密码 。    
当操作配置api的时候basic认证的userName为应用的appid，password为应用的秘钥 。
### 节点
因为本系统登录的时候没有用户名所以basic认证的时候用户名固定使用admin密码为当前设置的密码
#### model
```
    {
        "address": "http://localhost:5000",
        "remark": "this",
        "status": 0, // 1=online 0=offile
        "lastEchoTime": null
    }
```
#### 获取所有节点
| 参数名 | 值 |   
| ---- | ---- |   
| url | /api/node |   
| method | GET |   
| status code| 200 |    
| response content | [model] |        
#### 添加节点
| 参数名 | 值 |   
| ---- | ---- |   
| url | /api/node |   
| method | POST |   
| status code | 201 |    
| request body | model |   
| response content | 空 |    
#### 删除节点
| 参数名 | 值 |   
| ---- | ---- |   
| url | /api/node?address={address} |   
| method | DELETE |   
| status code | 204 |    
| response content | 空 |    
### 应用
因为本系统登录的时候没有用户名所以basic认证的时候用户名固定使用admin密码为当前设置的密码
#### model
```
   {
        "id": "xxx",
        "name": "测试程序3",
        "secret": "",
        "enabled": true, //是否启用
        "inheritanced": true, //是否可以继承
        "inheritancedApps": null //继承的app列表
    }
```
#### 获取所有应用
| 参数名 | 值 |   
| ---- | ---- |   
| url | /api/app |   
| method | GET |   
| status code | 200 |    
| response content | [model] |    
#### 获取单一应用
| 参数名 | 值 |   
| ---- | ---- |   
| url | /api/app/{appid} |   
| method | GET |   
| status code | 200 |    
| response content | model |    
#### 添加应用
| 参数名 | 值 |   
| ---- | ---- |   
| url | /api/app |   
| method | POST |   
| status code | 201 |    
| request body | model |    
| response content | 空 |   
#### 修改应用
| 参数名 | 值 |   
| ---- | ---- |   
| url | /api/app |   
| method | PUT |   
| status code | 200 |    
| request body | model |    
| response content | 空 |   
### 配置
配置的basic认证用户名使用appId密码使用secret
#### model
```
    {
        "id": "0986e7ed33c447618f28e92360394cea",
        "appId": "xxx",
        "group": "", //组
        "key": "key1", 
        "value": "3333",
        "description": null, //描述
        "onlineStatus": 0, //是否在线 0=等待上线 1=在线
        "status": 1 // 0=删除 1=正常
    }
```
#### 获取所有app的配置
| 参数名 | 值 |   
| ---- | ---- |   
| url | /api/config |   
| method | GET |   
| status code | 200 |    
| response content | [model] |   
#### 获取单一配置
| 参数名 | 值 |   
| ---- | ---- |   
| url | /api/config/{id} |   
| method | GET |   
| status code | 200 |    
| response content | model | 
#### 新建配置
| 参数名 | 值 |   
| ---- | ---- |   
| url | /api/config |   
| method | POST |   
| status code | 201 |    
| request body | model |   
| response content | 空 |   
#### 修改配置
| 参数名 | 值 |   
| ---- | ---- |   
| url | /api/config |   
| method | PUT |   
| status code | 200 |    
| request body | model |   
| response content | 空 |   
#### 删除配置
| 参数名 | 值 |   
| ---- | ---- |   
| url | /api/config/{id} |   
| method | DELETE |   
| status code | 204 |    
| response content | 空 | 
#### 上线配置
| 参数名 | 值 |   
| ---- | ---- |   
| url | /api/config/publish/{id} |   
| method | POST |   
| status code | 200 |    
| response content | 空 |
#### 下线配置
| 参数名 | 值 |   
| ---- | ---- |   
| url | /api/config/offline/{id} |   
| method | POST |   
| status code | 200 |    
| response content | 空 |

gihub地址：    
[AgileConfig](https://github.com/kklldog/AgileConfig)    
[AgileConfig.Client](https://github.com/kklldog/AgileConfig_Client)    