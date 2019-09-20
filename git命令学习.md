# git命令学习



**删除git本地信息**

```
rm -rf .git
```

**更改git用户名，邮箱**

```
git config --global user.name "Petterp"
git config --global user.email "1509492795@qq.com"
```



**新创建项目**

```
git clone git@gitlab.example.com:Shiyihui/comments-teach-cass.git
cd comments-teach-cass
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master
```



**在你的文件中使用**

```
cd existing_folder
git init
git remote add origin git@gitlab.example.com:Shiyihui/comments-teach-cass.git
git add .
git commit -m "Initial commit"
git push -u origin master
```



**从现有的 git仓库移植过来**

```
cd existing_repo 
git remote rename origin old-origin 
git remote add origin git@gitlab.example.com：Shiyihui / comments-teach-cass.git 
git push -u origin --all 
git push -u origin --tags
```