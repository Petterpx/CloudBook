# Linux命令行(应用型学习)

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



cp -r /root/jdk-11.0.5  /usr/local/java



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



#### 判断文件夹是否为null

```
if [ "`ls -A /var/www/html/xinghuo/new-app`" = "" ];
then 
        echo "is empty"
else
        echo " is not empty"
fi
```





#### 连接Mysql数据库

```
mysql -u root -p
jdbc:mysal://localhost:3306/
```



rpm -e 