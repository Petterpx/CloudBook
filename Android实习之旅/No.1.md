# 命令行操作

1. mac命令行的笔记
2. 熟悉代码
3. 了解开发流程，熟悉各个同事
4. 计划第二天的工作



## Mac本地生成 ssh的方法

**查看秘钥是否存在**

```
cd ~/.ssh
```

**生成新的秘钥**

```
ssh-keygen -t rsa -C “你的邮箱”
```

默认回车即可

**获取你的ssh**

```
cat ~/.ssh/id_rsa.pub
```



## Mac命令行

> ls 参数 目录名
> ls /
> ls /System/Libray
> -w 显示中文 -l 详细信息 -a 包括隐藏文件
> cd 目录名
> pwd 当前目录名
> mkdir 目录名
> rmdir 删除一个空目录
> mvdir dir1 dir2
> dircmp dir1 dir2
> diff dir1 dir2
> cp file1 file2
> rm 参数 文件
> mv file1 file2
> chmod 参数 权限 文件
> cat filename
> pg filename
> od -c filename
> find .-name"*.c" -print
> file filename
> grep "^[a-zA-Z]" filename
> uniq file1 file2
> comm file1 file2
> wc filename
>
>  
>
> 编辑：
> nano 文件名， ctrl+O(save) ctrl+X(exit)
>
> vi编辑器使用：
> vi filename
> i 插入模式
> esc 推出插入模式
> :w 保存当前编辑的文件但不退出
> :w newfile 文件另存为
> :w! filename 当前文件的内容替换filename中的原有内容
> :q 退出，文件为保存时会提示
> :q! 强制退出，不保存文件
> :wq 先保存文件，然后退出到shell
> :x 或者ZZ ,保存并退出返回shell
>
> ctrl + u 清空当前行
> ctrl + a 移动到行首
> ctrl + e 移动到行尾
> ctrl + f 向前移动
> ctrl + b 向后移动
> ctrl + p 上一条命令
> ctrl + n 下一条命令
> ctrl + r 搜索历史命令
> ctrl + y 召回最近用命令删除的文字
> ctrl + h 删除光标之前的字符
> ctrl + d 删除光标所指的字符
> ctrl + w 删除光标之前的单词
> ctrl + k 删除从光标到行尾的内容
> ctrl + t 交换光标和之前的字符
>
>  
>
> 
> 脚本命令
> sh 脚本文件名
> sh /clean
>
> alias ip="ifconfig | grep inet | grep netmask"
>
> Mac
> 截图：
>
> say "hello"
> screencapture -iW ~Desktop/screen.jpg 截取选定的窗口
> screencapture -S ~Desktop/screen.jpg 截取当前窗口
> screencapture -iC ~Desktop/screen.jpg 截取一部分
>
> 快捷键：
> ctrl + space 输入法切换
> 大写键 ，中英文切换
> cmd + Click 打开文件／文件夹／链接
> cmd + n 新建窗口
> cmd + w 关闭当前页
> cmd + t 新建标签页
> cmd + 数字 ， cmd + 方向键，切换标签页
> cmd + enter 全屏
> cmd + tab 切屏
> cmd + d 左右分屏
> shift cmd + d 上下分屏
> cmd + ; 自动补全历史纪录
> shift cmd + h 自动补全剪贴板历史
>
> ctrl + 方向键 切换窗口
> cmd + ctrl + d 查Mac自带词典
>
> 
> 后台服务管理：
> LaunchDaemons 未登录前就启动的服务
> LaunchAgents 用户登录后启动的服务
>
> 服务的plist文件目录：
> user：
> ～/Library/LaunchAgents
> administrator:
> /Library/LaunchAgents
> /Library/LaunchDaemons
> Mac OSX
> /System/Library/launchAgents
> /System/Library/LanuchDeamons
>
> 禁用服务的方法：
> $sudo launchctl unload plist filepath
> $sudo launchctl unload -w plist filepath
> 比如禁用spotlight
> $sudo lanuchctl unload /System/Library/LaunchAgents/com.apple.Spotlight.plist
> $sudo lanuchctl unload -w /System/Library/LaunchAgents/com.apple.Spotlight.plist
> 然后重启Mac OS；
> 方法2:
> $sudo lanuchctl unload /System/Library/LaunchAgents/com.apple.Spotlight.plist
> 然后将plist文件mv到其他目录备份，重启。
>
> 还原服务：
> $sudo launchctl load -wF plist filepath
> 或者：
> 将备份的plist文件mv回原来的文件夹
> $sudo launchctl load plist filepath
>
> 查看服务的状态：
> launchctl list
>
> 环境变量设置
> echo $SHELL 查看当前的shell
> PATH变量设置：
> vi .bash_profile 回车 编辑文件
> i进入编辑模式，添加usr/local/mysql/bin到环境变量
> export PATH=${PATH}:/usr/local/mysql/bin
> 按esc退出插入文字编辑模式
> 键入wq回车 ，保存并退出vi
> 键入source .bash_profile 让修改的文件生效

