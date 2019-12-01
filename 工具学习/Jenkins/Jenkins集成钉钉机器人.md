# Jenkins集成钉钉机器人

[钉钉机器人开发文档：](https://ding-doc.dingtalk.com/doc#/serverapi2/qf2nxq)

Jenkins使用钉钉机器人有两种方式，一种是使用Jenkins插件 Dingding,一种是通过 shell 脚本，后者更为灵活。



### 第一种

Jenkins 钉钉 插件：

![image-20191201111909101](https://tva1.sinaimg.cn/large/006tNbRwly1g9h1zrwsh5j30qk04kdga.jpg)

使用也很简单，直接在项目设置里配置：

![image-20191201112044504](https://tva1.sinaimg.cn/large/006tNbRwly1g9h21eygkkj31nk0i8gnt.jpg)

其中钉钉 access token就是你阿里机器人 url里的 token.



效果：

![image-20191201112151752](https://tva1.sinaimg.cn/large/006tNbRwly1g9h22k6k1bj30sw07u0uc.jpg)







### 第二种：

> 第一种虽然简单，但是不能扩展(插件也提供了另一种，不过并不好使)，所以我们往往采用第二种。



钉钉开发文档中的demo直接复制可能会一直提示 缺少json字段，所以复制以下的demo替换其中的 token等即可。具体参考钉钉开发文档。



**demo**:

```
curl 'https://oapi.dingtalk.com/robot/send?access_token=这是token' \
      -H 'Content-Type: application/json' \
      -d '
{"msgtype": "text", 
    "text": {
        "content": " 星火指挥平台-多渠道-构建成功  \n
        下载地址：http://123.57.111.111/xinghuo/channels-app/ \n
        \n@15111116934，@13611118522"
    }, 
    "at": {
        "atMobiles": [
            "15111116934", 
            "1361118522"
        ], 
        "isAtAll": false
    }
}'
```



复制上面的demo到项目配置中，执行shell,如下：

![image-20191201114253780](https://tva1.sinaimg.cn/large/006tNbRwly1g9h2ofwbnaj31h40jk0vx.jpg)



效果：

![image-20191201114028030](https://tva1.sinaimg.cn/large/006tNbRwly1g9h2lx10sxj30t60ek76c.jpg)

注意事项：全局变量，比如 ${BUILD_TYPE}并不能用于 消息中，不识别。

> 如下：暂时还未找到解决方案。![image-20191201121033710](https://tva1.sinaimg.cn/large/006tNbRwly1g9h3h8375oj30oi09ymy6.jpg)



***其中消息类型支持以下格式：***

#### MD

```
curl 'https://oapi.dingtalk.com/robot/send?access_token=我是token' \
      -H 'Content-Type: application/json' \
      -d '
{"msgtype": "markdown",
     "markdown": {
         "title":"杭州天气",
         "text": "
     	# 星火指挥平台-debug-构建成功
			*阿斯达阿斯达*
			- 当前版本：
			- 更新内容：asdasdasdasndkadsbadsasdasdnakjsdbahbjgvhggggggggggggg ghv h h
			adsakdbajda
			- 下载链接：[蒲公英下载地址](https://www.baidu.com) \n
			@1511231934
         "},
    "at": {
        "atMobiles": [
            "15122226934", 
            "189xxxx8325"
        ], 
        "isAtAll": false
    },
}'

```



#### 发送链接

```
curl 'https://oapi.dingtalk.com/robot/send?access_token=我是token' \
      -H 'Content-Type: application/json' \
      -d '
{
"msgtype": "link", 
    "link": {
        "text": "文本", 
        "title": "星火指挥平台构建成功", 
        "picUrl": "", 
        "messageUrl": "链接"},
}'
```



...更多的请参照官网教程复制更改。



