# 解决Mac中 adb: command not found



解决步骤：

```
touch .bash_profile   //创建文件
```

```
open -e .bash_profile  //打开文件
```

然后进行修改，格式为

> 注意，这里修改的时候，不要删除以前的配置，如果存在多个配置，可以加上：
>
> 比如下面：
>
> ```
> export PATH="$PATH:/Applications/010 Editor.app/Contents/CmdLine":/Users/petterp/Library/Android/sdk/platform-tools
> ```

```
 //其中的=后面为你的Android Studio，sdk目录所在位置，记得复制完最后加platform-tools
 export PATH=/Users/petterp/Library/Android/sdk/platform-tools
```

```java
source .bash_profile   //更新刚才更改的配置文件
```



然后就成功了！

#### 一些常用的 adb 命令

- 查看adb版本： `adb version`
- 查看所有设备： `adb devices`
- 安装指定apk：  `adb install <file>`
- 卸载指定包 ： `adb uninstall <package>`
- 连接设备 ：  `adb connect [<host>[:<port>]]`（默认端口号是：5555）
- 断开设备：  `disconnect [<host>[:<port>]]`
- 执行远程的shell：`adb shell`
- 执行远程shell命令： `adb shell <command>`
- 拷贝文件到设备上： `adb push <local> <remote>`
- 从设备中拷贝文件：`adb pull <remote> [<local>]`
- 查看设备所有信息： `adb bugreport`（包括 bug 报告）
- 最重要的命令：  `adb help` 查看命令帮助，有了他所有命令都知道了，就是 so important 。





#### 通过ADB安装APK

```
cd 到你的app目录
```

```
Adb install ....apk
```





android adb install apk 