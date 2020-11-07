# HTTP的原理和工作机制

> ### Http是什么？
>
> 两种最直观的印象
>
> - 浏览器地址输入地址，打开网页
> - Android发送请求，返回相对应的内容
>
> HypeText Transfer protocol 超文本传输协议
>
> - 超文本：在电脑显示，可以指向其他文本的链接的文本
>
> ### Url如何转换为HTTP报文
>
> ![image-20200517225946532](https://tva1.sinaimg.cn/large/007S8ZIlly1gevucilbukj31v70u0dw6.jpg)



### Request Methods

- #### Get

  获取资源：没有Body

- Post

  增加或修改资源:有body

- PUT

  修改资源：有body，幂等性

  ```
  什么是幂等性？
  指无论执行多次，返回的结果并不会改变，即服务器不会受到影响。
  其中put指挥对服务器资源改变一次，所以也满足幂等性。
  ```

- DELETE

  删除资源：没有body

  ```
  也具有幂等性  
  ```

- HEAD

  几乎和Get作用相同，区别在于，服务器返回内容时不会返回Body。

  比如你再做文件下载时，你可以用来获取文件的信息，大小，是否支持断点续传等；

### Status Code

- 作用：***对结果做出类型化描述 (如 获取成功  内容未

### Header

- HTTP消息的元数据(metadata)
- Host： 服务器主机地址：GET/user/HTTP/1.1
- Location: 重定向
- UserAgent: 告诉服务器当前访问的设备是什么，便于服务器返回指定的内容
- Range/Accept-Range:  分段加载，断点续传，指定Body的内容范围
- Cookie/Set-Cookie: 发送Cookie/设置Cookie

### 部分其他Header

- Accept ：客户端能接受的数据类型，如 text/html
- Accept-Charset  客户度接受的字符集，如 utf-8
- Accept-Encoding: 客户端接受的压缩编码类型，如zip
- Content-Encoding: 压缩类型。如zip

### Cache

- Cache 和 Buffer的区别
- Cache-Control :  no-cache,no-store,nax-age
- 

#### Host的作用是什么？

> 用于告诉服务器我要寻找的域名是什么，因为我们一个主机上可能会有多个服务器，或者对应了多个ip地址，如果没有host,那么就存在无法找到的问题。

### Content-Type/Content-Length

#### Content-Length用处是什么？

> 我们再发送或者接受http消息时，用于确定数据的长度，因为如果我们传递二进制数据，则没办法确定一个特定的结束符号。

### Content-Type 内容类型

- text/html  用于浏览器页面响应
- application/x-www-from-urlcoded: 普通表单,  参数最终encoded url格式
- Multipart/from-data   :  多部分形式，一般用于传输包含二进制内容的多项内容，如上传图片
- application/json:  最常见形式，json形式  ---RequestBody
- Image/jpeg/ 或 application/zip ：  单文件，用于WebApi 响应 或Post或者PUT请求



### Chunked Transfer Encoding分块请求

- Transfer-Encoding: chunked

- 表示 Body 长度无法确定，Content-Length 不能使用

- Body形式：

  <length1>

  <data1>

  <lenght2>

  <data2>

  0

  0表示结束

目的：**在服务端还未获取到完整内容时，更快的对客户端做出响应，减少用户等待**



## 总结

- Http到底是什么：用于传输超文本协议。以前是Html,现在也包括 WebApi 的数据
- HTTP 工作模型： 客户端按需求组装Http报文，发送给服务器，服务器处理后得到响应报文，发回给客户端，客户端处理响应报文；
-  请求报文格式 ： 请求行，Headers,Body
  - 请求行 ： Method,Path，Http version
    - Method: Get,Post,Put
  - Headers: 请求的meta data(元数据)
  - Body ： 要发送给服务器的内容
- 响应报文格式：状态行，Headers,Body
  - 状态行：HTTP version, Status Code
    - Status Code: 1xx(信息)，2xx(成功),3xx(重定向)，4xx(客户端错误)，5xx(服务器错误)
  - Headers : Host,Content-Type/Content-Length ,Location， User-Agent,Range.Accept-Range



#### 相关问题合集：

1. 用户在浏览器输入地址后回车，一段时间后浏览器显示出界面，简要概述背后干了什么？

   浏览器拼装HTTP报文并向服务器发送请求 -> 服务器处理并返回响应报文 -> 浏览器接收到响应报文后处理并使用渲染引擎来渲染出界面

2. 一个URL 如 http://api.baidu.com/user/1 中，对于 HTTP 组装报文来说可以拆分为哪几个部分

   http -> 协议部分   //api.baidu.com  -> 服务器地址   /user/1路径

3. Http的请求报文分为哪几个部分？

   请求行，Headers,Body

4. HTTP的请求报文分为哪几个部分

   请求行，Headers,Body

5. 请求行由哪几个部分组成？

   Method,path,HTTP版本

6. HTTP的响应报文分为哪几个部分？

   状态行，Headers,Body

7. 响应报文的状态行由哪三部分组成？

   HTTP版本，状态码，扩展信息





# 密码学

古典密码学

古代密码学

## 现代密码学

- 不止可以用于文字内容，还可以用于各种二进制数据

### 对称加密

- 使用秘钥和加密算法对数据进行替换，得到的无意义数据即为密文；使用秘钥和解密算法对密文进行逆向转换，得到原数据。

- ![image-20200522165447363](https://tva1.sinaimg.cn/large/007S8ZIlly1gf1bwa7ooej32ec0u0ahl.jpg)

- 经典算法 ：Des ，AES

  - ***DES*** 因为秘钥太短而被弃用，可能会被破解，存在破解的可能性大。
  - ***AED*** 

- 破解对称加密的方法？

  暴力破解，不断尝试。



###  非对称加密

原理：**使用公钥对数据进行加密得到密文；使用私钥对数据解密得到原数据**

![image-20200522171239219](https://tva1.sinaimg.cn/large/007S8ZIlly1gf1cev8i2bj327y0n0wm7.jpg)

```
如上图，在使用非对称加密时，公钥一般都是公开的，因为相对应的私钥只有自己持有，对方若想和你通信只能使用公钥去加密，但是至于如何解密对方就不得而知。这种做法使得安全性大大提高，但是，也带来了一个问题？
如果对方伪装发送方呢？
```

延伸用途：数字签名

![image-20200522175822760](https://tva1.sinaimg.cn/large/007S8ZIlly1gf1dqg1hbzj32e60r6any.jpg)

```
完整的用途如下：
我和小王两人想要通信
我们都先知道了对方的公钥和私钥，其中公钥我们可以公开，私钥只有我们彼此知道，当我们通信时：
我先使用小王的公钥加密我要发的内容，然后使用我自己的私钥对内容进行签名，再将内容发出去；小王收到后。先用我的私钥进行验证，如果验签成功就证明消息是消息我发的没错，然后再用他自己的公钥解密即可得到内容。
```

完整版过程

![image-20200522201732854](https://tva1.sinaimg.cn/large/007S8ZIlly1gf1hr99xylj322p0u0k71.jpg)

```
整个流程如下：
签名时：为了验证我的数据是否正常，先对原数据做一次hash，拿到它的摘要，然后使用自己的私钥进行签名，将加密后的摘要附加在原数据的末尾。
验签时：然后当对方拿到这个数据后，先使用公钥对加摘要进行一次加密算法
，得到待验证的摘要后再与原数据做了hash后的摘要进行对比，如果一致，即签名正确。
```



####  经典算法

 RSA,DSA



### Base64

- 将二进制数据转换成 64 个字符组成的字符串的编码算法

- 什么是Base64?

  由大写26个字母小写26个字母+,/,0-9这64个字符组成的文本。

- 用途

  - Base64 主要不是加密，它主要的用途是把一些二进制数转成普通字符用于[网络传输](https://www.baidu.com/s?wd=网络传输&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)。由于一些二进制字符在传输协议中属于[控制字符](https://www.baidu.com/s?wd=控制字符&tn=SE_PcZhidaonwhc_ngpagmjz&rsv_dl=gh_pc_zhidao)，不能直接传送需要转换一下。
  - 复杂化，使其不可读。

- **Base64 加密图片，可以更安全和高效吗？**

  不可以，Base64 加密算法本身都是公开的，而且加密解密的过程会使原长度增大1/3，所以谈不上安全和高效。

- **变种：Base58**

  去掉+,/及相似字符的文本串。甚至可以手抄

### 

### 百分号编码-**URL encoding**

- 将URL中的保留字符使用 百分号 “%” 进行解密

- 目的：消除歧义，避免解析错误

  - ```
    https://www.baidu.com/?name=petterp&and
    ```

    ```
    https://www.baidu.com/?name=petterp%26and
    ```



### 压缩和解压缩

- 压缩：把数据换一种方式来压缩，以减少存储空间

- 解压缩：把压缩后的数据还原成原先的形式，以便使用

- 压缩属于编码吗？

  属于，即不但能编码还能转回去。



###  序列化

- 序列号：把数据对象 (一般是内存中，例如JVM中的对象) 转换成字节序列的过程
- 反序列化：把字节序列重新转换成内存中的对象；
- 目的：让内存中的对象可以被存储和传输



### Hash

把任意数据转换成指定大小范围(通常很小)的数据

- 作用：摘要，数字指纹

- 经典算法：MD5,SHA1,SHA256

- 实际用途

  - 数据完整性验证

  - 快速查找：hashCode

  - 隐私保护 

    hash不能逆向，降低用户信息泄漏后带来的风险

- Hash破解方法---**彩虹表**

  **生成彩虹表 ，即将常见的密码提前hash一次，存进自己数据库，然后拿到用户密码后，查一遍**

- **彩虹表** 的破解方法 加盐

  在加密 时，只加密特定的字符，然后存储，这样即使密码被别人拿到，那么别人也无法得知最终密码是什么。

- Hash是加密吗？据说 MD5  是不可逆加密？

  

### 字符集

- 一个由整数向现实世界中的文字符号的隐射；

- 分支

  - ASCII: 128个字符，1字节

  - ISO-8859-1 : 对 ASCII 进行扩充，1字节

  - Unicode: 13万个字符，多字节

    - UTF-8:  Unicode 的编码分支
    - UTF-16 :  Unicode 的编码分支

  - GBK/GB2312/GB18030

    中国自研标准，多字节，字符集+编码





# 登录授权，TCP/IP

- Cookie
- Authorizataion

## Cookie

- Cookie的作用
  - 会话管理：登录状态，购物车等
  - 个性化：用户偏好，主题
  - Tracking: 分析用户行为
- 安全方面
  - XSS 跨站脚本攻击
  - XSRF  : Referer  

## Authorization

- Basic

  **Authorization: Basic <username:password(Base64ed)>**

- Bearer <access Token>

- 第三方授权---第三方登录

- OAuth2

  - OAuth2流程
  - 微信登录

- 自家App使用 Bearer token

- Refesh Token 



# TCP/IP 协议族

- 一系列协议所组成的一个网络分层模型
- 为什么要分层？因为现实网络不可靠性
- 具体分层
  - Application Layer 应用层：HTTP , FTP , DNS
  - Transport Layer 传输层： TCP, UDP
  - Internet Layer 网络层：IP
  - Link Layer 数据链路层：以太网，WI-FI



## TCP连接

- 什么叫连接？
- TCP连接的建立和关闭
- 长连接
  - 为什么要长连接？
  - 长连接实现方式：心跳
- Socket 套接字 怎么理解？插口 

# HTTPS

- HTTP Secure/HTTP over SSL/HTTP over TLS

- ssl 安全套接字层

- 定义 ： 在HTTP之下增加的一个安全层,用于保障 HTTP 的加密传输

- 本质： 在客户端和服务器之间用非对称加密协商出一套对称秘钥，每次发送信息之前将内容加密，收到之后解密，达到内容的加密传输

- 为什么不直接用非对称加密

  因为非对称加密太慢

### HTTPS连接

- 客户端请求建立 TLS 连接
- 服务端发回证书
- 客户端验证服务器证书
- 客户端信任服务器后，和服务器协商对称秘钥
- 使用对称秘钥开始通信 

![image-20200527233700207](https://tva1.sinaimg.cn/large/007S8ZIlly1gf7fmbqbmpj31vr0u0qt8.jpg)







## 从Retrofit 原理来看Http

### 什么是动态代理？

```

```















