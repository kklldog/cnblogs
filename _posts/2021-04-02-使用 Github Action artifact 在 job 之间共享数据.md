AgileConfig 在使用 react 编写UI后，变成了一个彻彻底底的前后端分离的项目，上一次解决了把react spa 跟asp.net core 站点集成起来 [asp.net core 集成 react spa](https://www.cnblogs.com/kklldog/p/netcore-embed-react.html)。本来我每次提交代码的时候都需要手动运行npm run build,然后把dist的内容复制到asp.net core网站的wwwroot/ui目录下。这样显然太麻烦了，于是尝试使用 github actions 来自动化这些步骤。   
我们要实现的目标是：提交代码后自动运行npm run build，自动把dist内容复制到wwwroot目录下，自动build dotnet程序，自动打包docker镜像，自动推送到dockerhub 。    
本来以为把这个actions分成两个job，job1负责编译react app，等job1完成后运行job2编译dotnet程序就可以了，但尝试下来并没有那么简单。其中有个问题就是job1生成的dist内容没有办法被job2使用，即使在job1里使用命令复制dist的内容到相应目录，job2还是无法使用这些内容，貌似每个job之间文件是隔离的。   
在经过咨询大佬后得知了Github Actions Artifact 这个功能。这样我们只需要把job1的产物先存储在Artifact内，job2去下载到指定目录就可以了。   
![](https://ftp.bmp.ovh/imgs/2021/04/a8f1d90741a0c784.png)
## Github Actions
Github actions 是 github 官方的 CICD 服务。它跟github 无缝集成，使得用户无需第三方服务就可以体验完整的CICD 服务。   
Github actions 可以完成很多功能，比如当你提交代码后自动build，test，然后打包docker镜像，发布到机器。这些功能只需要一个yml来描述就可以。   
Github actions 主要结构如下：
```
name:

on:

  job1:
    steps:
    ...
  job2：
    steps
    ...
```
## Artifact
Github actions Artifact 可以用来存储action生产出来的产物，比如npm build生成的静态文件。比如dotnet publish 生成的文件等等。当你上传成功后，后续的流程就可以下载这些文件来使用。
## job1 编译 react app
我们的workflow分两个job。第一个job用来编译 react app，并且上传dist的内容到artifact存储起来，以便第二个job使用它。这个job大概流程如下：   
1. 安装nodejs
2. run npm install 
3. run npm run build 
4. upload artifact

### actions/upload-artifact@v2
```
   - uses: actions/upload-artifact@v2
      with:
        name: agileconfig-ui
        path: AgileConfig.Server.UI/react-ui-antd/dist/
```
主要解释下actions/upload-artifact@v2这个命令。    
name：上传的artifact的名称，下载的时候需要使用。   
path：需要上传的文件夹的path。需要注意的是，这个path是相对repository的路径。因为使用npm命令的时候需要使用working-directory命令指定工作目录AgileConfig.Server.UI/react-ui-antd，所以不要觉得这个上传的path是相对working-directory的，如果只写dist是上传不了什么东西的。
## job2 编译发布 asp.net core
在编译完 react app 后我们得到了dist文件夹的内容。我们需要把这些内容复制到wwwroot/ui目录下面，之后进行docker镜像的打包工作。这个job大概流程如下：    
1. 安装dotnet
2. dotnet build & publish
3. download-artifact
4. docker build & push 
### actions/download-artifact@v2
```
    - uses: actions/download-artifact@v2
      with:
        name: agileconfig-ui
        path: AgileConfig.Server.Apisite/wwwroot/ui
```
这个命令跟上面的upload一样简单。   
name：需要下载的artifact的名称    
path：下载后存储数据的path。这个path还是相对repository的。
## 完整的yml
下面是workflow的完整yml配置：
```
name: master ci workflow

on:
  push:
    branches: [ master ]
    paths-ignore: 
      - '**/README.md'
      - '**/*.yml'
  pull_request:
    branches: [ master ]

jobs:
  build-reactapp:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: AgileConfig.Server.UI/react-ui-antd
    strategy:
      matrix:
        node-version: [12.x]

    steps:
    - uses: actions/checkout@v2
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v1
      with:
        node-version: ${{ matrix.node-version }}
        
    - run: npm install
    - run: npm run build
    - uses: actions/upload-artifact@v2
      with:
        name: agileconfig-ui
        path: AgileConfig.Server.UI/react-ui-antd/dist/
  build-dotnet:
    needs: build-reactapp
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Setup .NET Core
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 3.1.301
    - name: Install dependencies
      run: dotnet restore
    - name: Build
      run: dotnet build --configuration Release --no-restore
    - uses: actions/download-artifact@v2
      with:
        name: agileconfig-ui
        path: AgileConfig.Server.Apisite/wwwroot/ui
    - name: Push to Docker Hub
      uses: docker/build-push-action@v1
      with:
        username: ${{ secrets.DOCKER_HUB_NAME }}
        password: ${{ secrets.DOCKER_HUB_PASSWORD }}
        repository: kklldog/agile_config
        tags: test
```

## 关注我的公众号一起玩转技术   
![](https://s1.ax1x.com/2020/06/29/NfQjds.jpg)
