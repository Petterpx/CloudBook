# Mac 设置动态切换jdk版本



## 一些需要的命令

#### 查看当前jdk版 本

```
java -version
```

#### 查看当前安装的所有java版本

```
/usr/libexec/java_home -V
```



#### 打开  .bash_profile

```
open .bash_profile
```

#### 刷新 .bash_profile

```
source .bash_profile
```



## 教程开始

#### 配置你的 bash_profile

```

export JAVA_13_HOME=/Users/petterp/Library/Java/JavaVirtualMachines/azul-13.0.5/Contents/Home
export JAVA_8_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home

# 下面这两个顺序决定了默认使用的jdk配置
alias jdk8='export JAVA_HOME=$JAVA_8_HOME'
alias jdk13='export JAVA_HOME=$JAVA_13_HOME'
```

配置完刷新一下

```
source .bash_profile
```

使用的时候 使用 jdk8 或者 jdk13 切换要使用的命令

如下：

![image-20210227225242820](https://tva1.sinaimg.cn/large/008eGmZEly1go2hdb5wdcj30yg0c8tg4.jpg)

### 注意：

> 当关闭终端后，之前设置的版本会被重置。所以如果想要默认某个版本，注意 alias 的默认顺序。

