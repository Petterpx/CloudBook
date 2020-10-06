# Dart 语言极简入门(非原创)

> **备注：[原作者地址](https://juejin.im/post/6844904122878017549),以下为复制内容**

## **var 变量、final,const常量**

### 变量var

变量用关键词var来定义,Dart语言是类型推断的所以在定义变量的时候可以不用指明变量的类型，例子如下

```dart
    void main() {
      var name = "小明";
      var age = 10;
      var job;

     print(name);
     print(age);
     print(job);
   }
```

在上面例子中，定义了变量 name 和 age ,job,其中给name赋值 "小明"，根据类型推断name就是String 类型的，给 age赋值 10，根据类型推断age 就是 int 类型的。对job没有赋值job变量的默认值就是null。

需要注意的是变量的类型一旦确定就不能再给他赋不同类型的值。



### 常量final,const

常量用 final 或 const 关键字，例子如下

```dart
    void main() {
      final a = "好";
      const b = 1;
    }
```

需要注意的是，常量在定义的时候一定要进行赋值，且定义好以后不能被修改否则会报错。



## Dart中的数据类型

在Dart中all is Object ，Dart 中有 numbers（数值类型），strings （字符串类型），booleans（布尔类型），lists(列表类型)，maps（字典类型）等

### 数值类型

在Dart中数值类型只有int 和 double两种，例子如下

```dart
   void main() {
        int a = 1;
        double b = 2;

        print(a);
        print(b);
   }
```

上面的例子中定义了整数类型的a,和double 类型的 b,打印结果是

```
flutter: 1

flutter: 2.0
```



### String字符串类型

Dart中字符串用单引号 '' 或双引号 "" 来创建，例子如下：

```dart
  void main() {
    var a = "热爱编程";
    var b = '哈哈哈哈';
    print(a);
    print(b);
  }
  
  //输出结果：
  //flutter: 热爱编程
  //flutter: 哈哈哈哈
```



### 字符串的格式化

在Dart 中用 ${} 来进行字符串的格式化，例子如下：

```dart
  void main() {
    var a = "热爱编程";
    var b = '哈哈哈哈';
    var c =  "我们都  ${a + b} ";
    print(c);
 }

//输出结果为
//flutter: 我们都  热爱编程哈哈哈哈
```

上面的例子中字符串 a 和 b 可以用 + 号 直接拼接在一起



#### 在字符串中可以用反斜杠\来进行字符转译

字符串的相关方法使用例子如下：

```dart
   void main() {
    var a = "热爱编程";
    //判断是否为空
    print(a.isEmpty);
    //字符串的长度
    print(a.length);
    //字符串的类型
    print(a.runtimeType);
  }

 //输出结果为：
 //flutter: false
 //flutter: 4
 //flutter: String
```

上面例子是字符串的常用方法，isEmpty是判断字符串是否为空，length是求字符串的长度，runtimeType是求字符串的类型。

字符串也是一种集合类型，例子如下：

```dart
  void main() {
    var a = "热爱编程";
    print(a[1]);
  }
  //输出为：
  //flutter: 爱
```

### bool布尔类型

Dart中的布尔类型只有 true和false，例子如下：

```dart
 void main() {
    var a = false;
    print(a.runtimeType);
 }
 //输出结果：
 //flutter: bool
```

### List列表类型（数组类型）

用List来定义列表，可以在<>内指定列表元素的数据类型，例子如下：

```dart
  void main() {
    List<int> a = [10,100,1000];
    List<String> b = ["热", "爱","编","程"];
  }
```

上面例子中，尖括号中的类型用来指定元素的数据类型，如果列表指定了元素的数据类型，就只能存放该种数据类型的数据。如果想在列表中存放不同类型的数据，可以将列表中元素的类型用dynamic关键字声明为动态类型的，例子如下：

```dart
  void main() {
    List<dynamic> a = [10,"哈哈哈",99.9];
  }
```

但是在一般开发中我们不需要指定列表的元素类型，因为Dart是可以类型推断的，所以直接使用var进行声明即可，上面的例子一般如下声明：

```dart
  void main() {
    var a = [10,"哈哈哈",99.9];
  }
```

列表中的常用方法的使用例子如下：

```dart
  void main() {
    var a = [10,"哈哈哈",99.9];
    //添加元素
    a.add("ccc");
    //删除一个元素
    a.remove(10);
    //元素个数
    a.length;
    //...等等其他方法
  }
```



### Map字典类型

字典定义使用例子如下：

```dart
  void main() {
    Map<String, dynamic> a = {"name": "Flutter" , "languge": "Datr"};
  }
```

用Map来定义一个字典，在尖括号内来指定字典的key和value的数据类型，需要主要的是和数组一样，一旦类型被确定，那么就必须要和定义的类型一致否则会报错。

因为 Dart是可以类型推断的，所以上面的例子可以直接用var来定义如下：

```
void main() {
  var a = {"name": "Flutter" , "languge": "Datr"};
  print(a);
}
//输出为：
//flutter: {name: Flutter, languge: Datr}
复制代码
```

Map可以通过键来获取，设置，新增值，例子如下：

```dart
void main() {
  var a = {"name": "Flutter" , "languge": "Datr"};
  //新增键值对
  a["version"] = "1.1.0";
  //修改键值
  a["name"] = "Swift";
  print(a);
}
//输出结果：
//flutter: {name: Swift, languge: Datr, version: 1.1.0}
```



## **Dart中的运算符**

### 算数运算符

加"+"，减 "-" ，乘"*" ，除"/"，取整 "~/",取模%

"+"是用来做加的运算，除了数字类型外，字符串，列表，字典也支持相加运算，例子如下：

```dart
    void main() {

      var a = 10 + 20;
      var b = "10"  + "10";
      print(a);
      print(b);

   }

   //打印结果：
   //flutter: 30
   //flutter: 1010
```

"~/" 是取整的运算符，例子如下：

```dart
    void main() {
      var a = 15;
      var b =  a ~/ 4;
      print(b);
    }
    //打印结果：
    //flutter: 3
```

再其他运算符与其他编程语言基本一样不做介绍



### 比较运算符 

== ，!= ,> , < , >= ,<=

与其他语言基本一样不做介绍



### 类型运算符 

#### as,is, is!

- as是做类型的转换
- is是判断是否属于某个类型
- is！是判断是否不属于某个类型

例子如下：

```dart
    void main() {

        var dogA = Dog();
        var dogB = dogA;
        
        //
        if (dogB is Dog ){
            dogB.func_swim();
        }

        //用as来简化上面的代码结果一样
        (dogB as Dog).func_swim();

   }
   
   //打印结果；
   //flutter: 游泳
   //flutter: 游泳
```

上面的例子中 可以用as来简化代码。



### ??

“??” 是Dart的空条件运算符，就是取默认值的意思，例子如下

```dart
    void main() {
        var a;
        var b = a ?? 1;
        print(b);
    }
    //输出结果：
    //flutter: 1
```

上面的例子中， 变量a没有赋值所以默认为null，在将a赋值给b的时候因为a是null，所以 通过?? 给把1赋值给了a。



### 级联运算符 “..”

级联运算符可以对某个对象进行连续的操作而不用生成中间变量，例子如下： 例子一

```dart
  class Person{

    func_one(){
      print("Dart语言");
    }

     func_two(){
      print("Flutter使用Dart");
     }
  }

  void main() {
    Person()..func_one()..func_two();
  }
  //打印结果如下：
  //flutter: Dart语言
  //flutter: Flutter使用Dart
```



### 点运算符以及“?.”的使用

点运算符用来访问对象的属性或方法，对不确定是否为null的对象可以用“?.”来进行操作，例子如下：

```dart
  class Person{
    var name;

  }

  void main() {
    var p = Person();
    print(p?.name);
  }
```



## 函数

### 函数的一般书写格式

```
返回值  函数名称（参数列表）{
    函数体
}
复制代码
```

例子如下：

```dart
  void main() {

    int func_add(int a,int b){
      return a + b;
    }

    var c = func_add(1, 2);
    print(c);
  }
  //打印结果：
  //flutter: 3
```

上面例子中main函数是程序的入口， 里面定义了func_add函数



#### all is

Dart中的all is对象， 所以可以将函数赋值给一个变量或者将函数作为参数，例子如下；

```dart
  void main() {

    int func_add(int a,int b){
      return a + b;
    }

    var c = func_add;
    var d = c(1,2);
    print(d);
 }
```



#### 方法类型推断

Dart语言中在定义函数的时候函数的参数类型和返回值类型都可以省略，Dart语言会根据传入的参数和返回的值来动态推断。例子如下：

```dart
  void main() {

    func_add(a,b){
      return a + b;
    }

    var c =  func_add(1, 2);
    print(c);
}
//打印结果
//flutter: 3
```

如果某个函数的函数体只有一句话可以用箭头函数来定义，例子如下：

```dart
  void main() {

    func_add(a,b) => a + b;

    var c =  func_add(1, 2);
    print(c);
  }
  //输出结果：
  //flutter: 3
```



### 可选参数的函数定义

#### 名称可选参数函数

```dart
 返回值类型 函数名称({可选参数列表}){
      函数体
  }
```

可选参数放在{}号内部，例子如下：

```dart
  void main() {

    func_text({a,b}) {

      if(a != null){
          print(a);
      }

      if(b != null){
         print(b);
      }

     };

     func_text(a: 2);
 }
 //打印结果
 //flutter: 2
 
 //上面的例子中定义了函数func_text其中参数 a和b都是可选的,需要注意的是，在调用的时候需要参数名称加：的形式来进行传参数
```



#### 位置可选参数

只需要将可选参数放入中括号即可，例子如下；

```dart
  void main() {

    func_text(a, [b]) {

      if(a != null){
        print(a);
      }

      if(b != null){
       print(b);
      }

    };

    func_text(2,3);

  }
  //输出结果
  //flutter: 2
  //flutter: 3
```

在上面例子中func_text中有两个参数a和b,其中b 是可选的。在调用的时候可以不传。



#### 参数默认值

函数可选的可以给个默认值，如果没有默认值函数中取到的参数值为null，，设置默认值以后如果调用的时候没有传入这个参数，在函数内就会使用默认值，在定义可选参数的时候应该尽量提供一个默认值。

例子如下：

```dart
  void main() {

      func_text(a, [b = 4]) {

       if(a != null){
          print(a);
        }

        if(b != null){
           print(b);
        }

       };

      func_text(3);
  }
  
  //打印如下：
  //flutter: 3
  //flutter: 4
```



### 匿名函数

**Dart中没有名称的函数是匿名函数，匿名函数可以直接赋值给变量，通过变量来调用。例子如下：**

```dart
  void main() {

    var a = (b,c){
      return b + c;
    };

    var d = a(1,2);
    print(d);
  }
  //打印如下：
  //flutter: 3
```

在上面的例子中，把匿名函数赋值给了变量a,然后通过a来调用。



## 闭包

- **闭包是一种特殊的函数。**
- **闭包是定义在函数内部的函数。**
- **闭包能够访问外部方法内的局部变量并持有。**

经典例子如下：

```dart
  void main() {

    var  func = a();
    func();
    func();
    func();
  }

  a(){
    int count = 0;

    func_count(){
       print(count ++);
    }
    return func_count;
 }
 //打印结果：
 //flutter: 0
 //flutter: 1
 //flutter: 2
```

上面例子中， 函数a中的 func_count函数就是闭包，他定义在a函数内，能够访问并持有他外部函数的局部变量count,暂时理解到这里后面会详细说明闭包的使用。



## 类

**Dart是基于类和继承的面向对象的语言。对象是类的实例。在类中定义了对象的属性和方法。**

### 自定义类和类的构造函数

Dart中用 **class**关键字来进行类的定义，例子如下:

```dart
class Person {
    
    double weight;
    int age;
}
```

上面的例子中定义了Person类，类中定义了两个属性weight和age;我们可以通过Person来创建对象，通过类来创建对象需要调用类的构造函数。类的构造函数一般和类的名字一样。当定义一个类时Dart会默认生成一个没有参数的构造函数，可以直接通过类的名称来调用。

例子如下：

```dart
    class Person{
          var name;

     }

  void main() {
    var p = Person();
    print(p?.name);
  }
```

上面的例子中，定义的Person类可以直接通过Person()来创建对象。类对象中的方法属性都可以通过点来访问调用。

一般Dart语言的构造方法的书写格式如下：

```dart
    class Person{
          var name;
          var age;
          
          //一般构造方法的书写格式
          Person(this.name,this.age);

     }
```



### 类的实例方法：

**类定义了属性和方法，一般属性用来存储数据，方法用来定义行为。**

例子如下：

```dart
 class Person{
          var name;
          var age;
          
          //一般构造方法的书写格式
          Person(this.name,this.age);
          
          func_run(){
             print("在跑步");
          }
}
```

上面的例子中定义了方法func_run()就是Person类中的方法。调用的例子如下：

```dart
void main() {
    var p = Person();
    p.func_run();
}
```

在类中用this来访问自身的属性和方法，例子如下：

```dart
  class Person{
          var name;
          var age;
          
          //一般构造方法的书写格式
          Person(this.name,this.age);
          
          func_run(){
             print(" ${this.name}  在跑步"); //用this访问name属性
          }
}
```

类中还有两个比较特殊的方法，Setters 和 Getters方法，Setters用来设置对象的属性，Getters用来获取对象的属性。在定义属性时Dart会默认生成Setters和Getters方法。



### 抽象类和抽象方法

**Dart中 在抽象类中可以定义抽象方法，抽象方法是只有定义没有函数实现的方法。抽象方法是面向接口开发的基础。**

抽象类用abstract关键字来定义。例子如下：

```dart
abstract class PersonInterface{
         func_run();
   }
```

上面例子中定义了一个PersonInterface 的接口，里面只定义了一个func_run()的抽象方法。Person类可以对这个接口进行实现例子如下，用关键词implements来实现一个接口例子如下：

```dart
 abstract class PersonInterface{
         func_run();

 }
 
 class Person implements PersonInterface{
          var name;
          var age;
          
          //一般构造方法的书写格式
          Person(this.name,this.age);
          
          实现接口函数
          func_run(){
             print(" ${this.name}  在跑步"); //用this访问name属性
          }
}
```

- **一个类也可以同时实现多个接口。**
- **抽象类不可以被实例化。**



### 类的继承

**子类继承父类，就可以直接使用父类中的属性和方法。并且子类可以重写父类中的属性和方法。Dart中用extends关键字来进行类的继承。**

例子如下

```dart
  class Person{
          var name;
          var age;
          //一般构造方法的书写格式
          Person(this.name,this.age);
          
          func_run(){
             print("在跑步");
          }
}

class Man extends  Person{
    
    //构造函数调用夫类的构造函数
    Man(name,age) : super(name,age);
    
    func_eat(){
        print("正在吃饭");
    }
    
}
```

上面的例子中Man类继继承了Person类，需要注意的是构造函数不能被继承，但可以通过super来调用父类的方法。子类也可以重载父类的方法。例子如下：

```dart
  class Person{
          var name;
          var age;
          //一般构造方法的书写格式
          Person(this.name,this.age);
          
          func_run(){
             print("在跑步");
          }
}

class Man extends  Person{
    
    //构造函数调用夫类的构造函数
    Man(name,age) : super(name,age);
    
    func_eat(){
        print("正在吃饭");
    }
    
    @override
    func_run(){
        print("Man在跑步");
    }

}
```

上面例子中子类重载父类的func_run()方法，其中@override关键字用来标注这个方法是子类重载父类的，@override关键字可以省略。



### 枚举

**枚举用enum来定义。**例子如下：

```dart
  enum  Language{
      Dart,
      Oc,
      Swift
  }


  class Person{
          var name;
          Language language;
          
          //一般构造方法的书写格式
          Person(this.name,this.language);

 }
    
    
    void main() {
         var p = Person();
         p.language = Language.Dart;
         print(p.language);
   } 
   
   //输出：
   //Language.Dart
```



### 类的Mixin特性

**Mixin是Dart中非常强大的一个功能。Dart语言是单继承的只能继承一个父类。很多时候需要集合多个类的功能来实现一个复杂的自定义类，这时候Mixin特性就不错。**

**Mixin的功能就是让一个类把其他类的功能混合进来**。例子如下：

```dart
 mixin Walker{

      func_walk(){

       print("跑步");
      }
 }

  mixin Swim{

    func_swim(){

      print("游泳");
    }
 }

 mixin Flying{

    func_flying(){

       print("飞");
    }
  }

  class Dog with Swim, Walker {


   }


   void main() {

      var dog =  Dog();

      print(dog.func_walk());
      print(dog.func_swim());

   }
```

程序运行结果：

```dart
//flutter: 跑步

//flutter: 游泳
```

在上面例子中定义了，mixin的跑Walker和游泳Swim，然后让Dog类通过with 来遵守。Dog类就有了Walker和Swim里面的功能。

能被用来混合的类是Mixin类，Mixin类不能实现构造方法否则不能被其他类混合，用with关键字来进行混合。Mixin也支持多混合。



### 泛型

Dart的泛型和Java中的一样，也非常简单。

例子如下：

```dart
 class Person<T>{
          T data;   
          //一般构造方法的书写格式
          Person(this.data);
 }
 
 void main() {

     var p =Person("xiaoming");
     print(p.data);
     //打印结果：
     //flutter: xiaoming
 }
```

#### 限制泛型类型

```dart
class Apple{
  
}

class  Cloud<T extends Apple>{

}
```

#### 泛型方法

```dart
class Apple{}

class  Cloud<T extends Apple>{}

void  testT<T extends Cloud>(T t){
	//泛型方法
}
```



## 异步编程 

> **在Dart语言中可以通过 async ， await 关键字来编写异步执行的代码。也可以通过Future相关方法处理异步任务。**

### async ， await

```dart
 void main() {

      requestData();
      print("继续执行程序");

 }

requestData(){

  print("收到数据");
}
```

调用 上面函数的打印结果是：

```dart
flutter: 收到数据
flutter: 继续执行程序
```

可以看出上面例子中的函数是按顺序执行的。如果我们模拟真实的网络请求，那么requestData会有一定的耗时，即应该后打印“收到数据”，先打印“继续执行程序”。这时候我们就需要用到异步处理，对上面的例子修改如下：

```dart
    void main() {

       requestData();
       print("继续执行程序");

   }

    requestData() async {

      var data = await "收到的数据";
      print(data);
    }
```

上面程序运行结果为：

```dart
//flutter: 继续执行程序

//flutter: 收到的数据
```

现在可以看出是先打印了“继续执行程序”，后打印了“收到的数据”。实现了异步的效果。

被异步执行的函数需要用async关键字来修饰，且在函数内需要进行滞后处理的语句要用await关键字来修饰如上面例子中的 requestData()函数用 async关键字修饰， 函数内部 "收到的数据"前用await修饰。

异步和回调一般都是结合在一起来使用的。如上面的例子中，先从网络请求中获取数据， 接下来把拿到的数据渲染到UI界面上。就是一个异步回调的使用例子。该例子如下：

```dart
   void main() {

       requestData(refreshUI);
       print("继续执行程序");

   }

    refreshUI(data){
       print(data);
       print("刷新UI界面");
    }

    requestData(callBack) async {

       var data = await "服务端返回的Json数据";
       callBack(data);
    }
```

上面程序的运行结果是：

```dart
//flutter: 继续执行程序

//flutter: 服务端返回的Json数据

//flutter: 刷新UI界面
```

在Dart中函数可以作为参数，在上面的例子中将refreshUI函数当做参数传给requestData，在后在requestData中拿到数据data后调用。



### Future的使用

**任意一个 async函数都会返回一个Future对象。我们可以通过Future 的then方法来设置回调。**

上面的例子可以改写为：

```dart
   void main() {

      var future =  requestData();
      future.then( (data){

        refreshUI(data);
 
        });
        print("继续执行程序");

    }

   refreshUI(data){
     print("刷新UI界面 ${data}");
   }

   requestData() async {

       var data = await "服务端返回的Json数据";
       return data;
  }
```

程序运行输出结果：

```dart
//flutter: 继续执行程序

//flutter: 刷新UI界面 服务端返回的Json数据
```



## 模块的引用

### 用 import来导入模块

例子：

在lib 文件夹下创建aa.dart文件，其中aa.dart可以作为模块，在aa.dart中编写如下代码：

```dart
void  print_func(){
    print("aa文件的函数");
}
```

在程序的main.dart文件中导入aa.dart文件，就可以调用aa.dart文件中的print_func()函数了，例子如下：

```dart
import "./lib.aa.dart";

main(){
    
     print_func();
     //打印结果：
     //aa文件的函数
}
```

import 关键字用来导入模块，也可以选择只导入某个模块的某一部分，例子如下：



### 使用 show 导入特定代码

在上面例子中的aa.dart 文件中新增一个func_run(),func_eat()函数

```dart
    void  print_func(){
    
        print("aa文件的函数");
    
   }

    void func_run(){
    
        pint("我在跑步");
   }
   
   void func_eat(){
       print("在吃饭");
   }
```

在main.dart 中用关键字show只导入aa.dart 中的run_func()部分，如果要导入多个内容，可以使用逗号分割开即可，main.dart中的代码如下：

```dart
    import "./lib.aa.dart" show run_func ;
    
    main(){
        run_func();
        //打印结果：
        //我在跑步
    }
```



### 模块的重命名

**如果在不同的Dart文件中定义了相同名称的函数，将这些文件导入并调用这个函数的时候就会出错，这时候我们可以用 *as* 关键字来对模块进行重命名的方式避免错误。**

例子如下：

在lib文件夹下新建一个bb的文件，在里面写如下代码：

```dart
 bb.dart文件中写入：
 
    void  print_func(){
    
        print("bb文件的函数");
    
    }


在main.dart中写如下使用：


    import "./lib.aa.dart";
    import "./lib.bb.dart";
     
    
    main(){
        print_func();
        //运行就会报错
    }
```

对于上面的错误可以通过用as关键字对模块重新命名来避免。例子如下：

在main.dart文件中

```dart
    import "./lib.aa.dart";
    import "./lib.bb.dart" as bb;
    
    main(){
        print_func(); //aa.dart中的
        bb.print_func(); //bb.dart中的
        //打印结果：
        //aa文件的函数
        //bb文件的函数
    }
```

