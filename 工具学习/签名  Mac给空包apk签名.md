# 签名 | Mac给空包apk签名

```java
jarsigner -verbose -keystore petech.jks -signedjar sing_new.apk old.apk petech
```

-  petech.jks 你的签名
- sing_new.apk  签名后要生成的新的apk
- old.apk  你的apk路径
- petech 你jks的别名