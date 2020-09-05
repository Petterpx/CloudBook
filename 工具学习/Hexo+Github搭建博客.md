# Hexo+Github搭建博客

### 如果已经Node.js 可以使用npm 命令安装 Hexo

```
npm install -g hexo
```

接着在任意位置创建一个文件夹，如Blog，cd到该路径下执行以下命令

```
hexo init
```

该命令会在目标文件夹内建立网站所需要的所有文件。接下来是安装依赖包

```
npm install
```

到这里本地博客就搭建好了。执行以下命令（在你博客的对应文件夹路径下）

```
hexo generate
hexo server
```

在浏览器输入http://localhost:4000/ 就可以进行查看了。
当然这个博客是本地的，别人是无法访问的，之后我们需要部署到GitHub上。

##### 6.同步本地博客到Github

编辑自己创建的本地博客文件夹中的_config.yml中的deploy节点

```
deploy:
  type: git
  repo: git@github.com:username/username.github.io.git
  branch: master
```

注意：repo为这种形式的是配置了SSH Key之后的，如果没有配置则使用Https形式的地址。

为了能够使Hexo部署到GitHub上，需要安装一个插件

```
npm install hexo-deployer-git --save
```

然后输入以下命令

```
hexo clean
hexo generate
hexo deploy
```

在浏览器输入username.github.io就可以访问你的博客了。

#### 三、配置主题

Hexo主题在Github上有很多，如

- https://github.com/iissnan/hexo-theme-next
- https://github.com/litten/hexo-theme-yilia
- https://github.com/viosey/hexo-theme-material
- https://github.com/LouisBarranqueiro/hexo-theme-tranquilpeak
- https://github.com/pinggod/hexo-theme-apollo
- https://github.com/ppoffice/hexo-theme-icarus

挑选自己喜欢的一款。当然，自己能够DIY也是极好的。

#### 四、发布新文章

执行以下命令

```
hexo new post '文章标题'
```

这样会在本地博客的source->_posts路径下看到新建的文章，是md格式的，推荐使用Typora进行编辑。

编辑完成之后，与上面一样，执行以下命令

```
hexo clean
hexo generate
hexo deploy
```

即可更新到Github上。