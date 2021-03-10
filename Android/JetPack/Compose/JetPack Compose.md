# JetPack Compose

## 组合函数

JetPack Compost 是围绕可以组合函数而构建的，这些函数可以让开发者以编程的方式定义应用界面，就像Flutter一样，不过相比于Flutter,在Compose中，还需要手动增加 @Composable 注释。

组合函数只能在其他可组合函数的范围内调用，要使函数成为可以组合的函数，需要添加 **@Composable** 注释。

如下所示:

![image-20210227123606458](https://tva1.sinaimg.cn/large/008eGmZEly1go1zjqpgccj32060u04bg.jpg)



#### 实现预览功能

**@Preview(showBackground = true)**

通过在@Composable 修饰的函数身上增加 @Preview 注解，即可实现在Android Studio中预览的效果，但需要注意的是，方法不能接受任何参数。所以我们不能直接创建一个可接受参数的组合函数就去预览，往往需要写一个用于专门预览的无参数函数。

如下所示：

![image-20210227120610059](https://tva1.sinaimg.cn/large/008eGmZEly1go1yokpuk2j323q0jw0xq.jpg)

> 注意：如果右侧的预览没有及时刷新,可以通过点击刷新，手动刷新。
>
> <img src="https://tva1.sinaimg.cn/large/008eGmZEly1go1ypnly1bj30nq068t8t.jpg" alt="image-20210227120712094" style="zoom:50%;" align="left" />





## 布局

### 示例效果：

![image-20210227130215996](https://tva1.sinaimg.cn/large/008eGmZEly1go20ay7qzzj32hl0u0du4.jpg)

如上图所示，要做出类似的效果，我们首先要考虑，这个布局是一个垂直布局，而在Compose中，没有LinearLayout，所以我们使用Column,内部是一个图片加文字，需要注意的是图片与文件之间有间隔。



### 新认识的布局元素

#### Column

Column 用于垂直堆叠元素，如我们最上面示例那样。

#### Spacer

表示空白空间布局的组件，可以使用[Modifier.width]，[Modifier.height]和[Modifier.size]修饰符定义其大小。

#### Text

如同TextView一样,用于展示一段文本。

#### Modifier

用于配置布局的修饰符



## 对于状态的处理

如下示例

![image-20210227212941267](https://tva1.sinaimg.cn/large/008eGmZEly1go2eyx6bppj31b50u0wpj.jpg)

我们使用了 mutableStateOf 提供可组合可变存储器的函数



export JAVA_13_HOME=/Users/petterp/Library/Java/JavaVirtualMachines/azul-13.0.5/Contents/Home
export JAVA_8_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home

alias jdk8='export JAVA_HOME=$JAVA_8_HOME'
alias jdk13='export JAVA_HOME=$JAVA_13_HOME'


#JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home
PATH=$JAVA_HOME/bin:$PATH:.
CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:.

PATH=$PATH:/Users/petterp/Library/Android/sdk/platform-tools/



export JAVA_HOME
export PATH
export CLASSPATH


export PATH=/Users/petterp/flutter/flutter/bin:$PATH

#proxy list
alias proxy='export all_proxy=socks5://127.0.0.1:1080'
alias unproxy='unset all_proxy'



export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn