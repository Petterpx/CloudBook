# 使用fvm管理你的Flutter版本

Flutter 中，我们会使用 `CLI`(命令行) 来管理我们的 Flutter 版本，每次遇到切换其他项目或者老项目时，我们就需要再重新切一下版本，这种做法极其无聊且浪费时间。那么有没有一种方式，随意切换我们的Flutter版本呢，而且能不能让老项目使用指定版本，新项目使用另一个版本。

推荐使用 [fvm](https://fvm.app/) ,其就可以满足我们的需求，说简单直接点。

如果你使用桌面版本，就可以直接这样

<img src="/Users/petterp/Library/Application Support/typora-user-images/image-20220211180543084.png" alt="image-20220211180543084" style="zoom:50%;" />

如上图所示，可以针对这个项目随意切换版本，就是这么简单，当然也可以指定当前所有项目的全局版本(需要在bash_profile中配置)。

## 怎么使用？

桌面版链接：https://github.com/leoafarias/sidekick

使用命令行：https://fvm.app/docs/getting_started/overview

### 配置你的 bash_profile

打开终端

```
cd
// 编辑器模式打开
open .bash_profile

// 增加下面代码，记得更改你的路径 如petterp->xxx 
export FLUTTER_HOME=/Users/petterp/fvm/default/bin
export FLUTTER_DART=/Users/petterp/fvm/default/bin/cache/dart-sdk
export PATH=$FLUTTER_HOME:$FLUTTER_DART:$PATH

// 刷新bash
source .bash_profile
```

### 配置IDE

需要注意的是，对于 Android sutdio ,fvm 并不能做到独立项目级别的配置，只能使用全局去配置。