# Windows命令行

在给Windows服务器集成Jenkins时，用到了一些 cmd的命令行 ，资料非常不好查，所以做个记录



### 删除

##### 删除某个文件

```
del C:\wamp64\www\app\xinghuo\1.txt
```

删除某个文件夹下全部文件

```
del C:\wamp64\www\app\xinghuo*.*
```

删除某个文件夹连同子文件夹下所有文件，不包含文件夹

```
del C:\wamp64\www\app\xinghuo.  /s /q  (/q忽略选择)
```



删除某个文件夹

```
rd C:\wamp64\www\app\xinghuo
```

删除某个文件夹下所有文件夹

```
rd C:\wamp64\www\app\xinghuo /s /q
```



同时执行两个命令

```
...  & ...
```



### 复制

从某个文件夹复制到另一个文件夹，不复制文件夹内的子文件

```
xcopy  C:\Users\Administrator\.jenkins\workspace\星火指挥平台\newim\build\outputs\apk\wpy C:\wamp64\www\app\xinghuo\
```

从某个文件夹复制到另一个文件夹，包含复制文件夹内的子文件

```
xcopy  C:\Users\Administrator\.jenkins\workspace\星火指挥平台\newim\build\outputs\apk\wpy C:\wamp64\www\app\xinghuo\ /s
```