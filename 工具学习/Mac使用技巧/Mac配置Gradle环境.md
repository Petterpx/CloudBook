# Mac配置Gradle环境

最近不知道怎么了，Gradle命令莫名其妙用不了，以前也没有配置过 Gradle 的路径，都是AS自动装的，今天出现问题了，于是手动造一遍吧。

> *操作设备 **MAC***



一般下载Gradle，都是去百度搜，这里告诉大家一个小的方法吧。

打开Android Studio/
![image-20191205102131617](https://tva1.sinaimg.cn/large/006tNbRwly1g9lmt19aqxj30vm090jtk.jpg)

复制下面的地址，记得复制到浏览器时，删掉 https后面的  \ ,然后下载当前项目所使用的Gradle，或者自行去Gradle网站下载，目前Android Studio推荐使用的Gradle版本是 Gradle6.0.



下载完之后，本地解压，然后点访达进入资源库或者个人的文件夹。

![image-20191205102447670](https://tva1.sinaimg.cn/large/006tNbRwly1g9lmwkqvn1j30rc0o41kx.jpg)



这一步就是存放你Gradle文件的路径，自行决定即可，我的存放位置是：

![image-20191205102717393](https://tva1.sinaimg.cn/large/006tNbRwly1g9lmz1d12cj31cw0t2gw0.jpg)





存放完之后，**复制路径**，打开终端。

命令如下，一步步输：

```
cd
```

```
 touch .bash_profile 
```

```
open -e .bash_profile
```

然后会以文本形式打开你的环境变量脚本。



复制如下：(其中的路径改为你的即可)。

```
export GRADLE_HOME=/Users/petterp/Library/Android/gradle/gradle-5.4.1
export PATH=${PATH}:${GRADLE_HOME}/bin
```



然后保存，输入以下命令立即生效

```
source .bash_profile
```



#### 测试是否成功

```
gradle -version
```

![image-20191205103207421](https://tva1.sinaimg.cn/large/006tNbRwly1g9ln41lxyjj30vk0cy0xd.jpg)

如下的话，就成功了。如果出现别的问题，看看你的java环境配置是否正确，配置流程和**gradle**一样。mac貌似可以自己一键配...em







