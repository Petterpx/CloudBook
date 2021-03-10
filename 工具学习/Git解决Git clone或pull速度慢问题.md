# Git解决Git clone或pull速度慢问题

### 设置代理

git config --global http.proxy http://127.0.0.1:7890

git config --global https.proxy https://127.0.0.1:1080

其中 **1080** 是你梯子的端口号。



### 取消代理

git config --global --unset http.proxy

git config --global --unset https.proxy





### 实测截图

开启代理后

![image-20210305154122705](https://tva1.sinaimg.cn/large/008eGmZEgy1go92mi8yr7j30dh02674c.jpg)

开启代理前

![image-20210305154218373](https://tva1.sinaimg.cn/large/008eGmZEgy1go92ne7999j30e9020t8r.jpg)