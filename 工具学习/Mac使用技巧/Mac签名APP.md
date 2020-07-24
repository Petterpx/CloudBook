# Mac签名APP

```
jarsigner -verbose -keystore 你的签名录路径 -signedjar 签名后apk路径 要签名的apk路径 keyAlias
```

```
jarsigner -verbose -keystore /Users/petterp/Desktop/工作/网评相关/网评上线预留/petech.jks -signedjar /Users/petterp/Desktop/test/sing.apk /Users/petterp/Desktop/apk/yuan.apk petech
```

> as

```
jarsigner -verbose -keystore /Users/petterp/Desktop/工作/网评相关/网评上线预留/petech.jks -signedjar /Users/petterp/Desktop/apk/sing.apk /Users/petterp/Desktop/test/xiaomi.apk petech
```

