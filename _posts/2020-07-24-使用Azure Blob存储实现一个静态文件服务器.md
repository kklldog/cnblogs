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
这里使用dnspod来管理域名。   
添加一条记录：    
主机记录：files   
记录类型：cname   
记录值：azblob123.blob.core.windows.net   
这有配置之后，访问我自己的域名files.xbaby.xyz其实指向的是azblob123.blob.core.windows.net    
![UvMUWF.png](https://s1.ax1x.com/2020/07/24/UvMUWF.png)   
我们使用新域名访问下 http://files.xbaby.xyz/static/1.jpg 浏览器里出现了对应的图片，表示我们的自定义域名起作用了。
## 使用SDK上传文件
显然每次上传文件都要登录到Azure的管理平台太麻烦了，我们可以使用Azure Blob提供的.net sdk来制作一个小工具来方便上传文件。
### 新建一个winform项目
新建一个winform项目，一个框放一个按钮用来选择文件，选择后进行上传。   
![aScEOe.png](https://s1.ax1x.com/2020/07/25/aScEOe.png)
    
从nuget上安装AzureBlobStorage的sdk
```
Install-Package Azure.Storage.Blobs -Version 12.4.4
```
使用sdk上传文件需要一个连接串   
![UvKhV0.png](https://s1.ax1x.com/2020/07/24/UvKhV0.png)   
实现上传代码：
```
        private void btnSelectfiles_Click(object sender, EventArgs e)
        {
            if (openFileDialog1.ShowDialog() == DialogResult.OK)
            {
                var path = openFileDialog1.FileName;
                var fileName = path.Split("\\").Last();
                string connectionString = "DefaultEndpointsProtocol=https;AccountName=azblob123;AccountKey=GLtYbcXjy+KCOLUgIbdRoEPeWA+esNF/DWDNR7jABJuJrh46SuXfc7EOVS8yJXGXpZej3h/QFR9zzFrIAtuqrw==;EndpointSuffix=core.windows.net";
                var container = new BlobContainerClient(connectionString, "static");
                using (var file = File.OpenRead(path))
                {
                    container.UploadBlob(fileName, file);

                    MessageBox.Show($"{fileName}上传成功！");
                }
            }
        }
```
使用工具选择一张图片稍等一会图片就会上传上去拉。    
![aSRlSx.png](https://s1.ax1x.com/2020/07/25/aSRlSx.png)
## 总结
使用Azure Blob Storage可以方便的上传跟管理各种图片、文本、音视频等文件。上传的每个文件都有一个唯一的url对应，可以方便的通过http在全球访问内进行访问。使用这些特性我们可以轻松的把它当做静态文件服务器来用。我们还可以通过定义域名跟自己的域名结合起来使用，获得更加友好的使用体验。Azure Blob Storage还提供了各种语言的sdk方便使用代码来管理数据。

    
关注我的公众号一起玩转技术   
![](https://s1.ax1x.com/2020/06/29/NfQjds.jpg)