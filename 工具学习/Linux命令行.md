# Linux命令行

#### 删除文件夹

```
rm -rf ...
```

#### 删除文件

```
rm ...
```

#### 删除文件夹底下所有文件

```
rm -rf /Users/petterp/Documents/test/*
```

  Cp  -r /root/android-sdk-linux  /usr/local/



#### 创建文件夹

```
mkdir ..
```

#### 创建文件

```
touch ../a.txt
```





#### 复制文件夹到另一个文件夹

```
cp -r /home/packageA /home/packageB
```

#### 将一个文件夹下所有的内容复制到另一个文件夹

```
cp -r /home/packageA/* /home/cp/packageB/
或
cp -r /home/packageA/. /home/cp/packageB/
这两种方法效果是一样的。
```

/var/lib/jenkins/workspace/星火指挥平台/newim/build/outputs/apk/

#### 多条命令的顺序执行

(1) ; 分号，没有任何逻辑关系的连接符。当多个命令用分号连接时，各命令之间的执行成功与否彼此没有任何影响，都会一条一条执行下去。

(2) || 逻辑或，当用此连接符连接多个命令时，前面的命令执行成功，则后面的命令不会执行。前面的命令执行失败，后面的命令才会执行。

(3) && 逻辑与，当用此连接符连接多个命令时，前面的命令执行成功，才会执行后面的命令，前面的命令执行失败，后面的命令不会执行，与 || 正好相反。

(4) | 管道符，当用此连接符连接多个命令时，前面命令执行的正确输出，会交给后面的命令继续处理。若前面的命令执行失败，则会报错，若后面的命令无法处理前面命令的输出，也会报错。



#### 判断文件是否相等

```
if [ "BUILD_TYPE" = "$BILD_TYPE" ]
then 
echo "asd" ;
echo not equal ;
echo "asda" ;
fi
```

```
if [ "BUILD_TYPE" = "BILD_TYPE" ] 
then echo "asd" 
fi
||
if [ "BUILD_TYPE" = "$BILD_TYPE" ] 
then echo "123" 
fi ;

```



#### Linux中的switch语句

```
case 1 in
        1)     curl -F "file=@/var/www/html/new-app/newim-debug.apk" -F "uKey=a5560114f285630beb3d926760e02b56" -F "_api_key=e7cae18dd2f78ab3f68092e33ca19fe0" https://qiniu-storage.pgyer.com/apiv1/app/upload   ;;
        2)      echo "You selected perl"   ;;
        3)      echo "You selected python" ;;
        4)      echo "You selected ruby"   ;;
        *)      echo "I do not know"      ;;
esac
```





#### 判断是否存在已特定格式结尾的文件

```
  if [ls *.apk  2>&1]; then echo "123" ;fi
```

```
files=$(ls *.apk 2> /var/www/html/new-app/ | wc -l)

if [ "$files" != "0" ] ;

then
```

```
if [ -e *.log ]; then fi
```

```
if [ ! -f "/var/www/html/new-app/*.apk" ];then

echo "文件不存在"

fi
```

```
if [ -e &quot;*.apk&quot;]; 
```



#### 判断文件夹是否为null

```
if [ "`ls -A /var/www/html/xinghuo/new-app`" = "" ];
then 
        echo "is empty"
else
        echo " is not empty"
fi
```









```
if [ "`ls -A /var/www/html/xinghuo/new-app`" = "" ];
then 
curl 'https://oapi.dingtalk.com/robot/send?access_token=b2c8c091af59bde0d862afd5d07b10e3d3bf5d288a056c5cc906a44a10572b54' \
      -H 'Content-Type: application/json' \
      -d '
{"msgtype": "text", 
    "text": {
        "content": " 星火指挥平台-构建失败  \n
        具体报错信息请点击: http://123.57.221.167:8080/job/星火指挥平台/  \n
        @15129376934,@13681368522 "
    }, 
    "at": {
        "atMobiles": [
            "15129376934", 
            "13681368522"
        ], 
        "isAtAll": false
    }
}'
else
	case ${BUILT_TYPE} in
        ”debug“)     	curl 'https://oapi.dingtalk.com/robot/send?access_token=b2c8c091af59bde0d862afd5d07b10e3d3bf5d288a056c5cc906a44a10572b54' \
      -H 'Content-Type: application/json' \
      -d '
{"msgtype": "text", 
    "text": {
        "content": " 星火指挥平台-测试版-构建成功  \n
        蒲公英下载地址：https://www.baidu.com  \n
        备用下载地址：http://123.57.221.167/xinghuo/new-app/  \n  
        \n@15129376934，@13681368522"
    }, 
    "at": {
        "atMobiles": [
            "15129376934", 
            "13681368522"
        ], 
        "isAtAll": false
    }
}'   ;;
        ”“)    	curl 'https://oapi.dingtalk.com/robot/send?access_token=b2c8c091af59bde0d862afd5d07b10e3d3bf5d288a056c5cc906a44a10572b54' \
      -H 'Content-Type: application/json' \
      -d '
{"msgtype": "text", 
    "text": {
        "content": " 星火指挥平台-正式版-构建成功  \n
        蒲公英下载地址：https://www.baidu.com  \n
        备用下载地址：http://123.57.221.167/xinghuo/new-app/  \n
        \n@15129376934，@13681368522"
    }, 
    "at": {
        "atMobiles": [
            "15129376934", 
            "13681368522"
        ], 
        "isAtAll": false
    }
}'   ;;
        ”“)     	curl 'https://oapi.dingtalk.com/robot/send?access_token=b2c8c091af59bde0d862afd5d07b10e3d3bf5d288a056c5cc906a44a10572b54' \
      -H 'Content-Type: application/json' \
      -d '
{"msgtype": "text", 
    "text": {
        "content": " 星火指挥平台-多渠道-构建成功  \n
        蒲公英下载地址(基准包)：https://www.baidu.com  \n
        备用下载地址：http://123.57.221.167/xinghuo/new-app/  \n
        多渠道下载地址：http://123.57.221.167/xinghuo/channels-app/ \n
        \n@15129376934，@13681368522"
    }, 
    "at": {
        "atMobiles": [
            "15129376934", 
            "13681368522"
        ], 
        "isAtAll": false
    }
}' ;;
esac
fi
```





```
    	curl 'https://oapi.dingtalk.com/robot/send?access_token=b2c8c091af59bde0d862afd5d07b10e3d3bf5d288a056c5cc906a44a10572b54' \
      -H 'Content-Type: application/json' \
      -d '
{"msgtype": "text", 
    "text": {
        "content": " 星火指挥平台-测试版-构建成功  \n
        蒲公英下载地址：https://www.baidu.com  \n
        备用下载地址：http://123.57.221.167/xinghuo/new-app/  \n
        多渠道下载地址：http://123.57.221.167/xinghuo/channels-app/ \n
        \n@15129376934，@13681368522"
    }, 
    "at": {
        "atMobiles": [
            "15129376934", 
            "13681368522"
        ], 
        "isAtAll": false
    }
}'
```



测试

```
if [ "`ls -A /var/www/html/xinghuo/new-app`" = "" ];
then 
      curl 'https://oapi.dingtalk.com/robot/send?access_token=b2c8c091af59bde0d862afd5d07b10e3d3bf5d288a056c5cc906a44a10572b54' \
      -H 'Content-Type: application/json' \
      -d '
{"msgtype": "markdown",
     "markdown": {
         "title":"杭州天气",
         "text": "
 			 *星火指挥平台-构建失败*
 			 \n
			@15129376934
     
     "},
    "at": {
        "atMobiles": [
            "15129376934"
        ], 
        "isAtAll": false
    },
}'

else
	case ${BUILD_TYPE} in
        "ReleaseChannels")   curl 'https://oapi.dingtalk.com/robot/send?access_token=b2c8c091af59bde0d862afd5d07b10e3d3bf5d288a056c5cc906a44a10572b54' \
      -H 'Content-Type: application/json' \
      -d '
{"msgtype": "markdown",
     "markdown": {
         "title":"杭州天气",
         "text": "
 			 *星火指挥平台-多渠道打包-构建成功*
 			 \n
 			 # 
 			 #
			-	 更新内容：暂不支持
- 蒲公英下载链接：[点击去下载](https://www.pgyer.com/people?sig=m2zN7XFvrGF4rs1A%2FrCtbdFCLWEvp0lM2F5RAnedJ5X%2BWbZ0aajxcubfXWNUGK5f)
-  本地备用下载地址：[应对蒲公英每天上传限制](http://123.57.221.167/xinghuo/new-app/)
-	 渠道包下载地址：[如果当前打了渠道包](http://123.57.221.167/xinghuo/channels-app/)
\n
			@15129376934
     
     "},
    "at": {
        "atMobiles": [
            "15129376934"
        ], 
        "isAtAll": false
    },
}'     ;;
        ”debug“)     
          curl 'https://oapi.dingtalk.com/robot/send?access_token=b2c8c091af59bde0d862afd5d07b10e3d3bf5d288a056c5cc906a44a10572b54' \
      -H 'Content-Type: application/json' \
      -d '
{"msgtype": "markdown",
     "markdown": {
         "title":"杭州天气",
         "text": "
 			 *星火指挥平台-测试包-构建成功*\n
				##
			-	 更新内容：暂不支持
- 蒲公英下载链接：[点击去下载](https://www.pgyer.com/people?sig=m2zN7XFvrGF4rs1A%2FrCtbdFCLWEvp0lM2F5RAnedJ5X%2BWbZ0aajxcubfXWNUGK5f)
-  本地备用下载地址：[应对蒲公英每天上传限制](http://123.57.221.167/xinghuo/new-app/)
\n
			@15129376934
     
     "},
    "at": {
        "atMobiles": [
            "15129376934"
        ], 
        "isAtAll": false
    },
}'


        ;;
       ”release“)     
          curl 'https://oapi.dingtalk.com/robot/send?access_token=b2c8c091af59bde0d862afd5d07b10e3d3bf5d288a056c5cc906a44a10572b54' \
      -H 'Content-Type: application/json' \
      -d '
{"msgtype": "markdown",
     "markdown": {
         "title":"杭州天气",
         "text": "
 			 *星火指挥平台-线上-构建成功*\n
				##
			-	 更新内容：暂不支持
- 蒲公英下载链接：[点击去下载](https://www.pgyer.com/people?sig=m2zN7XFvrGF4rs1A%2FrCtbdFCLWEvp0lM2F5RAnedJ5X%2BWbZ0aajxcubfXWNUGK5f)
-  本地备用下载地址：[应对蒲公英每天上传限制](http://123.57.221.167/xinghuo/new-app/)
\n
			@15129376934
     
     "},
    "at": {
        "atMobiles": [
            "15129376934"
        ], 
        "isAtAll": false
    },
}'


        ;; 
        
        
esac

fi
```

[Asd](http://123.57.221.167/xinghuo/new-app/)

```
	# 星火指挥平台-${BUILD_TYPE}-构建成功
			- 当前版本：
			- 更新内容：
			- 下载链接：[蒲公英下载地址](https://www.pgyer.com/people?sig=m2zN7XFvrGF4rs1A%2FrCtbdFCLWEvp0lM2F5RAnedJ5X%2BWbZ0aajxcubfXWNUGK5f)
      -	本地备用下载地址：[应对蒲公英每天上传限制](http://123.57.221.167/xinghuo/new-app/)
      - 渠道包下载地址: [如果当前打了渠道包](http://123.57.221.167/xinghuo/channels-app/)
      *注意，每次打渠道包时会删除以前的备份*
      #
      #
			@15129376934
        
```

   	*星火指挥平台-${BUILD_TYPE}-构建成功*



-	 更新内容：
- 蒲公英下载链接：[点击去下载](https://www.pgyer.com/people?sig=m2zN7XFvrGF4rs1A%2FrCtbdFCLWEvp0lM2F5RAnedJ5X%2BWbZ0aajxcubfXWNUGK5f)
-  本地备用下载地址：[应对蒲公英每天上传限制](http://123.57.221.167/xinghuo/new-app/)
-	 渠道包下载地址：[如果当前打了渠道包](http://123.57.221.167/xinghuo/channels-app/)


