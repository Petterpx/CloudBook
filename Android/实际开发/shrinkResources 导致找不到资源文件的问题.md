# shrinkResources 导致找不到资源文件的问题

`shrinkResources` 用于缩减我们项目中哪些未曾使用到的资源文件，这其中也包括三方库中的资源文件。

一般情况下，资源缩减与代码混淆会配合使用，在官方的建议中，建议先启用了混淆，确定哪些类是否需要保留，然后再决定是否移除资源。

正常情况下，一般是不会报错的，当遇到三方库时，情况也就不一定了。

## 启用 shrinkResources 后的微博sdk资源报错

![image-20211216170535999](https://tva1.sinaimg.cn/large/008i3skNly1gxfsa5hc7uj31d60u0444.jpg)

当接入微博 sdk 之后，开启资源缩减后，就会出现上述报错，原因是微博使用了 **getResources().getIdentifier()** 这种方式去获取资源文件，具体源码见这里：

![image-20211216155224264](https://tva1.sinaimg.cn/large/008i3skNgy1gxfq60cik5j323i0a4whv.jpg)

按照 google 的文档，当使用 `shrinkResources` 时，默认会启用严格引用检查，当使用 **getResources().getIdentifier(name,x)** 这种方式获取资源文件时，即意味着代码会根据 `name` 去查询资源，资源缩减器会将 与 `name` 匹配的所有资源标记为可能使用，避免被移除。

按道理来说，本不应该报错，不知道为什么，这里还是报错了，所以我们需要手动去保留一下资源文件，如下所示：

![image-20211216162335113](https://tva1.sinaimg.cn/large/008i3skNgy1gxfr2gimi2j31yy0e4tai.jpg)

```
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:ignore="MissingDefaultResource"
    tools:keep="@drawable/retry_btn_*,@drawable/weibosdk_*" />
```

按照上述操作，即可解决相应的报错问题。