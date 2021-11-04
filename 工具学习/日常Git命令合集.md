# 日常Git命令合集



> 一个Android Dev日常的git命令合集。



#### 添加多个分支

```shell
git remote add 分支名 地址

git remote add dev https://github.com/xxx
```

#### 删除远程分支

```shell
git remote remove
```



#### 重新绑定分支

有些时候分支名不对，**push** 总要全名，这个时候先删除 **remote** 远程分支

再创建与远程相应的本地子分支名。**(**或者拉下来远程分支**)**

然后先 **pull** 远程分支，再执行下面语句更新

```shell
git push --set-upstream 远程分支名(origin) 子分支名(master)
```



#### 推送代码失败时

```shell
git push -u github master
```

**强推：**

```shell
git push -f github master
```



#### 新建子分支后，关联远程分支信息

```shell
git branch --set-upstream-to=blue/master
```





