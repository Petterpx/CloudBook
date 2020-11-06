# Flutter子widget调用父widget方法报 Tried calling: call()

在学Flutter状态管理时，需要子Widget调用父Widget，报如下错误：

![image-20201105225227362](https://tva1.sinaimg.cn/large/0081Kckwly1gkeopw8gexj30d903maa5.jpg)

找了好一会，最后删除 () 后发现正常了，经过查询后得出以下结论，如下图所示：

![image-20201105224848959](https://tva1.sinaimg.cn/large/0081Kckwly1gkeom5arabj30kz0dkq4l.jpg)

> 当然上面的  **final Function onChanged** ，**Function** 也可以不用加。

希望对刚开始学Flutter的同学能有所帮助。

