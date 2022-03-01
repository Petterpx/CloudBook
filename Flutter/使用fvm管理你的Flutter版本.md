# 使用fvm管理你的Flutter版本

在使用Flutter中，我们会使用CLI来管理我们的Flutter版本，每次遇到老项目时，我们就需要再重新切一下版本，这种做法极其无聊且浪费时间。那么有没有一种方式，随意切换我们的Flutter版本呢，而且能不能让老项目使用指定版本，新项目使用另一个版本。

推荐使用 fvm ,其就可以满足我们的需求，说简单直接点。

<img src="/Users/petterp/Library/Application Support/typora-user-images/image-20220211180543084.png" alt="image-20220211180543084" style="zoom:50%;" />

如上图所示，可以针对这个项目随意切换版本，就是这么简单，当然也可以指定当前所有项目的全局版本(需要在bash_profile中配置)。

## 怎么使用？

桌面版链接：https://github.com/leoafarias/sidekick



### 配置你的bash_profile

具体打开终端

```
cd
open .bash_profile
//增加下面代码
export FLUTTER_HOME=/Users/petterp/fvm/default/bin
export FLUTTER_DART=/Users/petterp/fvm/default/bin/cache/dart-sdk
export PATH=$FLUTTER_HOME:$FLUTTER_DART:$PATH
//
source .bash_profile
```

### 配置IDE

需要注意的是，对于Android sutdio,fvm并不能做到独立项目级别的配置，只能使用全局去配置。