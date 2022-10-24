# Mac Charles使用教程

#### [下载地址](https://www.charlesproxy.com/download/latest-release/)

### 电脑端操作

#### 点击 Proxy-SSL-Proxying Settings

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1goo8046bsgj30yo0qqkcc.jpg" alt="image-20210318181101990" style="zoom: 50%;" />

#### **设置代理端口**

<img src="https://tva1.sinaimg.cn/large/008eGmZEly1goo80offa4j30xv0u0786.jpg" alt="image-20210318181134996" style="zoom:50%;" />



#### 设置要抓取的地址或端口(https需要)

![image-20210318182918061](https://tva1.sinaimg.cn/large/008eGmZEly1goo8j4jiimj31nr0u0b29.jpg)

![image-20210318182941300](https://tva1.sinaimg.cn/large/008eGmZEgy1goo8jk059oj31q00rcdpf.jpg)

> 其中 * 代表匹配任何host



#### 安装mac证书

![image-20210318182016576](https://tva1.sinaimg.cn/large/008eGmZEly1goo89qcn81j31xo0ow1kx.jpg)

![image-20210318182043625](https://tva1.sinaimg.cn/large/008eGmZEly1goo8a75kfpj30uu0ia0xh.jpg)

这里添加为系统证书。mac端的操作到这里就结束了





### 手机端操作

#### 设置代理地址

**按住option点击wifi图标，查看WiFi信息**

![image-20200724224348342](https://tva1.sinaimg.cn/large/007S8ZIlly1gh2g0vcp80j308w09279v.jpg)



> **需要注意的是Android设备或者模拟器需要和mac处于同一局域网**



**接着更改Android设备wifi代理信息**

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1gh2g54d995j30u01rcq7g.jpg" alt="Screenshot_20200724-224637_Settings" style="zoom:33%;" />



**稍等片刻，会弹出弹窗，提示将设备加入ip组，点击确认**

![image-20200724224741039](https://tva1.sinaimg.cn/large/007S8ZIlly1gh2g4vw6plj30ln04h74y.jpg)



**如果要抓https的包，请按以下执行**

> **注意，Android7.0以上默认禁止用户证书信任，所以建议在低版本手机抓包或者使用xopsend插件,如果是自己的app,则允许信任用户证书即可**







#### **手机安装证书**

Android 浏览器输入：chls.pro/ssl

ios浏览器输入： chls.pro/ssl

下载相应的证书文件，安装即可。

