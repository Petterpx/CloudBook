# mac终端设置Socks代理方法



**首先打开你的梯子工具。v2ray 或者 ssr。**

**找到你的代理端口设置。**

#### v2ray

![image-20200812182430456](https://tva1.sinaimg.cn/large/007S8ZIlly1gho7awuz6dj30cs0c6wft.jpg)

#### SSR

![image-20200812182505445](https://tva1.sinaimg.cn/large/007S8ZIlly1gho7bif1ssj30dh07vt9b.jpg)





**记录下你相应的Sockes 端口。然后打开终端：**

```
open -e .bash_profile
```



**进去后输入以下配置信息：**

```
#proxy list
alias proxy='export all_proxy=socks5://127.0.0.1:1080'
alias unproxy='unset all_proxy'
```

**接着刷新配置：**

```
source .bash_profile
```

**然后打开代理**

```
proxy
```

**查看当前ip**

```
curl cip.cc
```

![image-20200812183438324](https://tva1.sinaimg.cn/large/007S8ZIlly1gho7lg5lw1j30ku0820uq.jpg)





#### 如果不想使用时关闭代理

```
unproxy
```



> 每次关闭终端时，代理也会自动关闭，再次使用需再次打开

