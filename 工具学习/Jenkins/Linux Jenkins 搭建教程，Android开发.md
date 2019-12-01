# Linux Jenkins 搭建教程，Android开发

**搭建环境：**

Contos 7.2 64位，阿里学生云

**操作设备 Mac** 

软件：终端，TranSmit

> windows使用
>
> Shell5,xftp



既然你能看到这篇博文，那么肯定知道Jenkins是什么？那么多的解释就不说了，直接开始我们的整个完整教程。无废话。如果过程有什么不明白，欢迎留言给我。

![Jietu20191129-175559](https://tva1.sinaimg.cn/large/006tNbRwly1g9h6en2ef4j30jm0jktac.jpg)



### 安装jdk

```java
yum install -y java
```

查看jdk版本

```
java -version
```



### 安装jenkins

#### 安装

```
wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
yum install -y jenkins
```

如果失败，就去官网下载最新 jenkins 包。[下载地址](http://pkg.jenkins-ci.org/redhat-stable/)

```java
wget http://pkg.jenkins-ci.org/redhat-stable/jenkins-2.190.3-1.1.noarch.rpm
rpm -ivh jenkins-2.190.3-1.1.noarch.rpm
```

#### 配置jenkins端口

```java
 vi /etc/sysconfig/jenkins
```

jenkins默认8080端口，如果不冲突可以不修改。

```
JENKINS_PORT="8080"  此端口不冲突可以不修改 
```



#### 相应的 Jenkins 命令

```java
service jenkins start
service jenkins stop
service jenkins restart
```



#### 修改默认为root用户运行(可选)

***Jenkins默认不是以root用户运行，这样的话就无法执行 shell 命令。***

**编辑配置文件**

```java
vim /etc/sysconfig/jenkins
```

**将 jenkins 用户修改为 root**

```java
$JENKINS_USER="root"
```

**修改jenkins 相关文件夹用户权限**

```java
chown -R root:root /var/lib/jenkins
chown -R root:root /var/cache/jenkins
chown -R root:root /var/log/jenkins
```

**重启jenkins**

```
service jenkins restart
```





### 安装Android SDK

[官网下载](http://tools.android-studio.org/index.php/sdk/)，移动到你 服务器上某个位置

比如 /usr/local/Android SDK

**然后解压**

```
tar -zvxf android-sdk_r24.4.1-linux.tg
```



**配置环境变量**

在 /etc/profile下添加以下两句

```java
export ANDROID_HOME=/usr/local/Android SDK
export PATH=$ANDROID_HOME/tools:$PATH
```



**更改配置生效**

```
source /etc/profile
```



然后安装相应的SDK。

先查看所有版本：

```java
android list sdk --all
```

![image-20191201133619258](https://tva1.sinaimg.cn/large/006tNbRwly1g9h5ygqvm2j30ug0u0avz.jpg)



**下载指定版本SDK:**

按你的项目配置来定

```
android update sdk -u --all --filter 2,3,4,5,6,7,8
```



**安装所有版本SDK**

或者安装所有版本的SDK

```java
android update sdk --no-ui
```



### 安装Git

```
 yum install git
```

[但是yum安装的版本过低，详细教程请参考。](https://www.cnblogs.com/imyalost/p/8715688.html)



### 配置Jenkins(Android定制)

刚安装的Jenkins,在打开的时候需要安装一些插件，因国内网实在太慢，需要设置一些代理，自行设置即可。

插件默认全部安装，除了以下插件(Android用户)，其余的皆不需要，后续再安装。

> - gradle
> - git
> - github
> - maven
> - ssh
> - 中文插件



#### 系统配置

***上面的最基本的环境安装完了，下面我们正式开始配置：***

***这里以Android开发为例。***

进入jenkins，点击系统管理-全局工具配置：

### ![image-20191201142923048](https://tva1.sinaimg.cn/large/006tNbRwly1g9h7hofbmqj323m0qmq6t.jpg)

配置 jdk,git,gradle路径，其中gradle可以让jenkins自行下载，免除手动安装





**配置SDK环境**

进入jenkins,点击系统管理，全局工具配置：

这里配置的就是刚才安装的Android SDK

![image-20191201143123860](https://tva1.sinaimg.cn/large/006tNbRwly1g9h7jrrvzvj32100gm415.jpg)





