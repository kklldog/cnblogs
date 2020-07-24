## 什么是Azure Blob Stoage
Azure Blob Stoage 是微软Azure的对象存储服务。国内的云一般叫OSS，是一种用来存储非结构化数据的服务，比如音频，视频，图片，文本等等。用户可以通过http在全球任意地方访问这些资源。这些资源可以公开访问，也可以私有访问。看到这些描述立马就想到这这个服务可以用来做静态文件服务。   
![Uv3x3T.png](https://s1.ax1x.com/2020/07/24/Uv3x3T.png)    
如果你有免费账户那么可以使用5G的免费额度，用来存一些图片跟JavaScript等小文件也足够了。    
![UxefG4.png](https://s1.ax1x.com/2020/07/24/UxefG4.png)   
Azure Blob Stoage的存储结构。
## 创建存储账户
![UvmRDx.png](https://s1.ax1x.com/2020/07/24/UvmRDx.png)
    
创建账户跟其他服务类似，取个实例的名称，选区域，还是那个套路哪个区域离你近就选哪个。   
![UvmsC4.png](https://s1.ax1x.com/2020/07/24/UvmsC4.png)
    
设置网络，默认设置即可。   
![UvmcvR.png](https://s1.ax1x.com/2020/07/24/UvmcvR.png)
    
高级设置，把“需要安全传输”禁用，为了测试方便咱不走https。   
![Uvm629.png](https://s1.ax1x.com/2020/07/24/Uvm629.png)
    
点击“创建”就开始部署实例，等待一会就可以完成了。   
![UvKTGF.png](https://s1.ax1x.com/2020/07/24/UvKTGF.png)
    
![UvKoPU.png](https://s1.ax1x.com/2020/07/24/UvKoPU.png)    
回到资源主界面开始新建容器，取个名字“static”,公共访问级别选择“Blob仅匿名访问blob”。   
![UvK55T.png](https://s1.ax1x.com/2020/07/24/UvK55T.png)
    
点击新建的容器，可以查看容器里的资源文件，可以上传删除文件。    
![UvK4aV.png](https://s1.ax1x.com/2020/07/24/UvK4aV.png)
    
每个上传上去的文件，都会对应一个url，通过这个url可以直接进行访问。
![UvMNJU.png](https://s1.ax1x.com/2020/07/24/UvMNJU.png)    
在浏览器里访问一下这张图片，可以在浏览器里显示出来。    
分析一下这个url：https://azblob123.blob.core.windows.net/static/1.jpg    
https://azblob123.blob.core.windows.net代表帐户实例地址    
static代表容器   
1.jpg代表文件    
## 自定义域名
到这我们的文件可以上传，可以访问，已经做为静态文件服务器使用了。但是这个域名不太友好，让我们来给它换个自己的域名访问。
![UvKWbq.png](https://s1.ax1x.com/2020/07/24/UvKWbq.png)
选择左边菜单“自定义域”。界面上提示有两种方式可以设置自定义域名，我们使用CNAME来实现以下。
![UvKRrn.png](https://s1.ax1x.com/2020/07/24/UvKRrn.png)
     15
![UvMUWF.png](https://s1.ax1x.com/2020/07/24/UvMUWF.png)
    7
![UvKhV0.png](https://s1.ax1x.com/2020/07/24/UvKhV0.png)
