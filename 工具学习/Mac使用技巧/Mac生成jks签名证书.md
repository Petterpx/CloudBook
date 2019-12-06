# Mac生成jks签名证书

必要环境：安装了jdk



终端：

**秘钥和证书管理工具**

```
keytool
```



**创建签名**

```
keytool -genkey -v -keystore myKey.jks -keyalg RSA -keysize 2048 -validity 10000 -alias myAlias
```



**查看秘钥的具体信息**

```
keytool -v -list -keystore myKey.jks -alias myAlias
```



