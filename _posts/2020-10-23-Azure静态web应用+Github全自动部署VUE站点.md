## 什么事Azure静态web应用
Azure 静态 Web 应用是一种服务，可从 GitHub 存储库自动构建完整的堆栈 Web 应用，并将其部署到 Azure，目前它还是预览版。   
![BAJAC4.md.png](https://s1.ax1x.com/2020/10/23/BAJAC4.md.png)   
Azure 静态 Web 应用通过与github actions集成，通过监听仓库的分支，当分支有push，pull request等动作的时候自动触发构建，并且部署到Azure。   
Azure 静态 Web 应用支持对常见的VUE，React，Angular甚至Blazor进行自动构建及部署。并且部署的网站会使用Azure分布在全球的服务器，当用户访问的时候会选择地理位置最近的服务器来加速访问速度提高用户体验。   
主要特点：
1. 适用于 HTML、CSS、JavaScript 和映像等静态内容的 Web 托管。
2. 由 Azure Functions 提供的集成 API 支持。
3. 一流的 GitHub 集成，其中存储库更改将触发生成和部署。
4. 全球分布的静态内容，使内容更接近你的用户。
5. 可自动续订的免费 SSL 证书。
6. 自定义域为应用提供品牌自定义。
7. 调用 API 时使用反向代理的无缝安全模型，这不需要配置 CORS。
8. 身份验证提供程序与 Azure Active Directory、Facebook、Google、GitHub 和 Twitter 集成。
9. 可自定义的授权角色定义和分配。
10. 后端路由规则，使你能够完全控制所提供的内容和路由。
11. 生成的临时版本由拉取请求提供支持，在发布前提供站点的预览版本。

## 创建VUE项目
这次我们使用国内最常见的VUE作为前端的框架来体验下Azure静态web应用的功能。   
使用VUE CLI新建一个VUE项目，使用过VUE的用户应该都知道，CLI生成的项目直接是可以运行的。
```
vue create az_static_vue_test
```
有了VUE的代码之后我们还需要把代码存在Github上。   
在Github上新建一个repository：   
![BAKrod.png](https://s1.ax1x.com/2020/10/23/BAKrod.png)   
新建完成之后使用Git Push命令把az_static_vue_test的代码推上去。
## 创建静态Web应用
我们新建好VUE项目然后推送到Github之后就可以开始在Azure创建静态Web应用资源了：   
在portal找到静态web应用功能，点击“创建”，弹出创建界面：   
![BAKaQK.png](https://s1.ax1x.com/2020/10/23/BAKaQK.png)   
![BAKyFA.png](https://s1.ax1x.com/2020/10/23/BAKyFA.png)   
跟创建其他资源类似，填写一个名称，区域选离自己近的。源代码管理选择使用Github账户，点击之后会跳转到Github授权页面。授权完成后就可以选择刚才上次的VUE项目了。   
储存库：az_static_vue_test   
分支：main   
填写完成之后点击“创建”开始创建资源，等待一会Azure提示创建成功之后我们可以进入资源的概览界面。复制URL地址到浏览器访问一下：
![BAKwLD.png](https://s1.ax1x.com/2020/10/23/BAKwLD.png)   
可以看到我们的VUE项目的默认界面出现了。也就是说Azure静态web应用为我们自动编译了VUE的代码并把产物直接部署好了。   
![BAKdsO.png](https://s1.ax1x.com/2020/10/23/BAKdsO.png)   
接下来让我们修改下项目源代码，再次推送到Github上：
```
  
<template>
  <div class="hello">
     <h2>
       Azure Static App by Vue
     </h2>
  </div>
</template>

```
我们把src\components\HelloWorld.vue的组件简单的修改下，只留下一句话Azure Static App by Vue 然后提交。
![BAKDdH.png](https://s1.ax1x.com/2020/10/23/BAKDdH.png)    
我们回到github上那个repository，选择Acitons，可以看到有个任务正在执行，其实Azure静态web应用跟Github就是通过Actions串联起来的。等待这个任务变成绿色，我们再次访问下上面的URL，可以看到首页已经变成了我们编辑后的样子，说明已经自动化部署成功了，真香。
![BAKBee.png](https://s1.ax1x.com/2020/10/23/BAKBee.png)

## 总结
今天试用了Azure静态web应用功能，并且配合github全自动部署了一个VUE站点，虽然它还是一个预览版，体验相当不错，简单易用。Azure静态web应用不光支持VUE，还支持angular，react等常见的前端框架，甚至还支持自己最新的blazor技术。有了它开发者只管玩命写代码就行了，至于其他的啥都不用管，什么CICD，什么Devops，什么Workflow统统不用管，一切交给Azure，真香。

## 关注我的公众号一起玩转技术   
![](https://s1.ax1x.com/2020/06/29/NfQjds.jpg)