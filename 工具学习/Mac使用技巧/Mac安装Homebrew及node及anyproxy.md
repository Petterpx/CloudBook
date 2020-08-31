# Mac安装Homebrew及node及anyproxy







[原链接](https://medium.com/@weixiao_6070/%E5%85%B3%E4%BA%8Emac%E4%B8%8B%E8%BD%BDbrew%E6%8A%A5curl-7-failed-to-connect-to-raw-githubusercontent-com-port-443-connection-refused-2293c50b7ff)

## Mac下载brew

- 1、下载brew方法
- 2、相关报错

## 1、下载brew方法

下载很简单，网上的相关方式有很多

```
//Mac下载brew管理工具：
	/usr/bin/ruby -e "$(curl -fsSL http://raw.githubusercontent.com/Homebrew/install/master/install)"
	//只需要复制上面代码就可以，
```

## 2、相关报错

下面是下载brew之后的一个报错问题

```
error:
curl: (7) Failed to connect to raw.githubusercontent.com port 443: Connection refused
       查了好多方法，发现都是copy的，根本解决不了，然后，找到了一个算是
   脚本的东西，我将她上传至github上面了，直接clone就可以。
https://github.com/jackzhaoyu/ceshi.git
```

下载下来的rb文件，你可以随便放，但是你能找到就可以。首先你先执行curl 看是不是有这句话

```
curl: try 'curl --help' or 'curl --manual' for more information
```

没有问题的话，然后cd到你保存rb文件的那个目录，继续执行

```
ruby ***********.rb
```

执行完毕之后然后执行brew，就可以了。





## 安装node

```
brew link node
brew uninstall node
brew install node
```



## 安装  anyproxy

```
npm install -g anyproxy
```