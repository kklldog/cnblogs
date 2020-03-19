---
layout: default
title:  "试用GithubPackages功能"
---
前几天微软收购npm的新闻对于软粉来收很是振奋。微软收购npm很可能是为了加强Github Packages。目前Github，Typescript，VSCode，npm这些开源社区的重磅工具全部都在微软旗下，显示出了微软对开源的态度，微软已经不是以前那个封闭的微软。Github推出Github Packages功能有一段时间了，一直没使用过，今天有空打算折腾一下，体验一下。
## 什么是Github Packages
Github Packages是一个包承载服务，它完全跟Github集成。Github Packages使你的源码和包在同一个地方进行统一的管理，使你可以集中的在Github上开发跟发布。你可以发布公共包（public packages）跟所有人分享，也可以发布私有包（private packages）提供给个人或者组织使用。以上是对官方文档的简单翻译。说简单点就是以前你代码是在Github，但是上包可能是在npm，maven或者nuget上，现在你在Github上传代码后还可以直接把包也上传到Github，方便统一管理，发布。
## 在Github Packages上发布包
下面让我们开始尝试使用Github Packages发布一个包吧。
### 在Github上新建一个仓库HiGithubPackage
新建一个公共的仓库命名HiGithubPackage
![]()
### 在Github上申请Access Token
在Github上申请一个新的Access Token。这个Token是用来上传Package的凭证，后面需要配置。登录Github后点击个人头像-Settings-Developer settings-Personal access tokens-Generate new token，然后勾选packages的权限后点Generate token按钮生成token。复制好这个token。
### 新建一个.net Core项目HiGithubPackage
使用Visual studio新建一个core标准库项目。新建一个类，这个类里只有一个静态方法Hi，调用的话会输出一行Hi GithubPackage ~。我打算把这个库上传到Github Packages上去。
```
   public class GithubPackage
    {
        public static void Hi ()
        {
            Console.WriteLine("Hi GithubPackage ~");
        }
    }
```
顺手把代码也push到github上去吧。
```
git push -u origin master
```
### 在项目文件夹下新建一个nuget.config文件
新建一个nuget.config文件，并且配置它。   
OWNER填写你Github的用户名   
UserName填写你Github的用户名   
Token填写上面申请的access token   
以下是我配置的
```
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <packageSources>
        <clear />
        <add key="github" value="https://nuget.pkg.github.com/kklldog/index.json" />
    </packageSources>
    <packageSourceCredentials>
        <github>
            <add key="Username" value="kklldog" />
            <add key="ClearTextPassword" value="xxx" />
        </github>
    </packageSourceCredentials>
</configuration>
```
### 修改包信息并打包
在Visual studio上右键项目，选择编辑项目文件。我们在csproj文件下编辑包信息。其中包含包的id，版本，授权等，比较简单一看就明白了。
```
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
    <PackageId>HiGithubPackage</PackageId>
    <Version>1.0.0</Version>
    <Authors>minjie.zhou</Authors>
    <Description>Test upload to github packages</Description>
    <PackageLicenseExpression>MIT</PackageLicenseExpression>
    <RepositoryUrl>https://github.com/kklldog/HiGithubPackage</RepositoryUrl>
    <GeneratePackageOnBuild>true</GeneratePackageOnBuild>
  </PropertyGroup>

</Project>
```
修改完后ctrl-b一下进行一次编译。编译完后在bin\debug下会生成一个.nupkg的包文件。
### 上传包到Github packages
使用dotnet cli进行上传
```
dotnet nuget push "bin/debug/HiGithubPackage1.0.0.nupkg" --source "github"
```
这里可能要多试几次，有的时候会提示401的错误。

### 新建一个.net Core控制台项目HiGithubPackage
