# Gradle | allprojects ，根 repositories 区别是什么？

开发良久一直不是很明白(说白了还是懒)，上述的关系到底是什么区别，于是就没太注意，直到 `Jenkins`打包时发现(本地打包没遇到过)：

![image-20201209202024325](https://tva1.sinaimg.cn/large/0081Kckwly1glhve7mikfj31wc0dyjw4.jpg)

找不到 **fragment-ktx:1.2.4** 这个依赖，而且离奇的是，它居然去 `fabric` 的仓库底下去找，这就很离奇。一顿思考，先是尝试更改优先级也无济于事,最后狠心删除了爱彼迎仓库，换成了优先阿里云本地依赖，于是解决了这个比较 **简单** 的问题。

```gradle
maven { url "http://maven.aliyun.com/nexus/content/groups/public/" }
```



## 反思，为什么会出现这个问题呢？

|                      app->build.gradle                       |                         build.gradle                         |                      build.gradle->all                       |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![image-20201209202827125](https://tva1.sinaimg.cn/large/0081Kckwly1glhvmjrg2yj30ko096dgj.jpg) | ![image-20201209202916618](https://tva1.sinaimg.cn/large/0081Kckwly1glhvneh8rjj30o205nwez.jpg) | ![image-20201209203039692](https://tva1.sinaimg.cn/large/0081Kckwly1glhvoufg5qj30oe0aejsd.jpg) |

如上述所示，我们一般的项目都有后两者，特别是中间的 build.gradle 缺少了其，项目根本无法正常打包。

## 真相

### app->build.gradle

**buildScript 块** 的repositories主要是为了Gradle脚本自身的执行，获取脚本依赖插件。(**注意：**`上图的方式个是为了依赖本地的lib包`)。

### build.gradle

**根级别**的 `repositories` 主要是为了当前项目提供所需依赖包，比如log4j、spring-core等依赖包可从mavenCentral仓库获得。

### build.gradle->all

**allprojects块** 的 `repositories` 用于多项目构建，为所有项目提供共同所需依赖包。而子项目可以配置自己的repositories以获取自己独需的依赖包。

## 注意事项

### 慎用 `mavenLocal`

![image-20201209205124998](https://tva1.sinaimg.cn/large/0081Kckwly1glhwafuw9vj311p0u015y.jpg)

## 参考

[Gradle中的依赖管理](https://docs.gradle.org/current/userguide/dependency_management.html#sec:repositories)

[google开发者](https://docs.gradle.org/current/userguide/dependency_management.html#sec:repositories)

[buildScript块、allprojects块、根级别三种所属的repositories区别](https://blog.csdn.net/HD243608836/article/details/80696406)

