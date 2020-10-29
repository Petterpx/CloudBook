# Jenkins集成总结

在开始之前，允许我先喷一会，网上各种复制粘贴博客，真。。。em

先下载文件

> http://mirrors.jenkins.io/war-stable/latest/jenkins.war



然后命令行 进入下载的文件夹之后，执行 **java -jar jenkins.war**



***以下只针对安装包安装用户。***

### 卸载命令

> //进入以下目录，双击运行
> /Library/Application Support/Jenkins/Uninstall.command
> //也可以这样运行
> sh "/Library/Application Support/Jenkins/Uninstall.command"
>
> //删除配置，这个可选
> sudo rm -rf /var/root/.jenkins ~/.jenkins
> sudo launchctl unload /Library/LaunchDaemons/org.jenkins-ci.plist
> sudo rm /Library/LaunchDaemons/org.jenkins-ci.plist
> sudo rm -rf /Applications/Jenkins "/Library/Application Support/Jenkins" /Library/Documentation/Jenkins
> sudo rm -rf /Users/Shared/Jenkins
> sudo dscl . -delete /Users/jenkins
> sudo dscl . -delete /Groups/jenkins
> sudo rm -f /etc/newsyslog.d/jenkins.conf
> pkgutil --pkgs | grep 'org\.jenkins-ci\.' | xargs -n 1 sudo pkgutil --forget
>
> //如果使用brew安装的，可以执行以下命令
> brew uninstall jenkins

教程https://blog.csdn.net/momokuku123/article/details/102467750

***非安装包mac用户进入/Users/petterp/.jenkins/workspace 中删除此文件夹即可***

**windows用户进入 C:\Users\Administrator.jenkins\workspace**





#### Android开发插件

- git

- github

- gitlab

- [Email Extension Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Email-ext+plugin)

- [Localization: Chinese (Simplified)](https://github.com/jenkinsci/localization-zh-cn-plugin) 中文语言包(感觉有时候很鸡肋)

- [SSH plugin](http://wiki.jenkins-ci.org/display/JENKINS/SSH+plugin)

- [Upload to pgyer](https://wiki.jenkins.io/display/JENKINS/Upload+Pgyer+Plugin)

- [Gradle Plugin](https://github.com/jenkinsci/gradle-plugin)

- [description setter plugin](http://wiki.jenkins-ci.org/display/JENKINS/Description+Setter+Plugin) （自定义build）

  

#### 查看一下配置项

#### 查看Git

which git





### 服务器安装教程：

安装以下必备工具：

1. git
2. jdk
3. Sdk
4. tomcat



其中比较坑的是，sdk在安装时，官方下载的包只是sdk25,在构建项目时需要同意一些权限才可以，此时使用下面的方法：

> **.\sdkmanager.bat "platforms;android-28"**
>
> 输入后，按y即可，即同意一些权限。



如果客户端并没有安装git,可能会出现git HEAD等情况，此时重新安装git,使用自己安装的git即可 



## Android与蒲公英的搭配使用

**官方教程：**

[蒲公英官方链接](  https://www.pgyer.com/doc/view/jenkins)

[使用蒲公英插件进行上传](https://www.pgyer.com/doc/view/jenkins_plugin)





一些Shell命令行

> cp -ri /home/server/tomcat/* /home/server/test/
>
> 复制某文件到某文件

> ```
> if` `[ -f ``"/data/filename"` `];``then
> echo` `"文件存在"
> else
> echo` `"文件不存在"
> fi
> ```
>
> 判断文件是否存在

> cp -rf "源文件地址" "新文件地址"
>
> 复制文件到某文件夹

> rm -rf ""
>
> 删除某文件



设置下载生成二维码：

![image-20191119200406610](https://tva1.sinaimg.cn/large/006y8mN6ly1g93lqba56cj31sa0bygni.jpg)

> <a href="${appBuildURL}"> <img src="${appQRCodeURL}" width="118" height="118"/> <a/>

自定义build显示文本

> 星火指挥平台-${BUILD_NUMBER} -> ${BUILD_TYPE}



一些配置选项：

打包选项 ***clean assemble${BUILD_TYPE}***





**https://juejin.im/entry/6844903560497332231**