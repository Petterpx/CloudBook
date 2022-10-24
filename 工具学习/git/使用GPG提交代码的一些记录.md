# 使用GPG提交代码的一些记录

### commit时提示找不到gpg的文件夹

![image-20220126120850564](https://tva1.sinaimg.cn/large/008i3skNly1gyqy40yh80j30mq0400sz.jpg)

解决方式：

在终端运行

```shell
git config --global gpg.program $(which gpg)
```

参考链接：https://github.com/isaacs/github/issues/675
