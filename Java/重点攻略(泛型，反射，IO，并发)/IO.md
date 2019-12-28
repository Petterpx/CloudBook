# IO

### 字符和字节

在java 中有输入，输出两种 IO 流，每种输入，输出又分为字节流和字符流两大类。字节又是什么呢，每个字节(byte)右8bit 组成，每种数据类型又几个字节组成。

#### 字节和字符之间的关系是怎样的呢？

java 采用 unicode 编码，2个字节来表示一个字符，这点与C 语言中不同，C语言中采用 ASCII，在大多数系统中，一个字符通常占1个字节，但是在 0~127 整数之间的字符映射， unicode 向下兼容 ASCII 。而Java  采用unicode 来表示字符，一个中文或英文的 unicode编码都占两个字节。但如果采用其他编码方式，一个字符占用的字节数则各不相同。

例如：Java 中的String 类是按照 unicode 进行编码的，当使用 String(byte[] bytes,String encoding)构造字符串时，encoding 所指的是 bytes 中的数据时按照那种方式编码的，而不是最后产生的 String 是什么编码方式，换句话说，是让系统吧 bytes 中的数据由 encoding 编码方式转换成 unicode 编码。如果不指名，bytes 的编码方式将有 jdk 根据操作系统决定。

**getBytes(String charsetName)**  使用指定的编码方式将 String 编码为 byte 序列，并将结果存储到 一个 新的 byte 数组中。如果不指定将使用 操作 系统默认的编码方式，我的电脑默认的是 GBK编码。

```java
public class Test {
    public static void main(String[] args) {
       String str="Petterp";
       int byte_len=str.getBytes().length;
       int len=str.length();
        System.out.println("字节长度为"+byte_len);
        System.out.println("字符长度为："+len);
        System.out.println("系统默认编码方式为："+System.getProperty("file.encoding"));
    }
}
```

输出结果：

> 字节长度为7
> 字符长度为：7
> 系统默认编码方式为：UTF-8

在 UTF-8编码中，一个英文字母字符存储需要1个字节，一个汉字字符存储需要3到4个字节。在 UTF-16编码中，一个英文字母字符存储需要两个字节，一个汉字字符储存需要3-4个字节 (Unicode扩展区的一些汉字存储需要4个字节)。在 UTF-32 编码中，世界上任何字符的存储都需要4个字节。

**简单来讲，一个字符表示一个汉字或英文字母，具体字符与字节之间的代销比例视编码情况而定。有时候读取的数据是乱码，就是因为编码方式不一致，需要进行转换，然后再按照 unicode 进行编码。**



### File 类

File 类是 java.io 包下代表与平台无关的文件和目录，也就是说，如果希望在程序中操作文件和目录，都可以通过 File 类来完成。

```java
public class Test {
    public static void main(String[] args) throws IOException {
        //以当前路径来创建一个File 对象
        File file=new File(".");
        //直接获取文件名，输出一点
        System.out.println(file.getName());
        //获取相对路径的父路径可能出错，下面代码输出 null
        System.out.println(file.getParent());
        //获取绝对路径
        System.out.println(file.getAbsoluteFile());
        //获取上一级路径
        System.out.println(file.getAbsoluteFile().getParent());
        //在当前路径下创建一个临时文件
        File tempFile=File.createTempFile("aaa",".txt",file);

        //指定当 JVM 退出时删除该文件
        tempFile.deleteOnExit();

        //以系统当前时间作为新文件名来创建新文件
        File newFile=new File(System.currentTimeMillis()+"");
        System.out.println("newFile对象是否存在："+newFile.exists());
        //以指定newFile 对象来创建一个文件
        newFile.createNewFile();
        //以newFile对象来创建一个目录，因为newFile 已经存在。
        //所以下面方法返回 false,即无法创建该目录
        newFile.mkdir();
        //使用 list()方法列出当前路径下的所有文件和路径
        String[] fileList=file.list();
        System.out.println("当前路径下所有文件和路径如下==");
        for (String fileName:fileList){
            System.out.println(fileName);
        }

        //listRoots()静态方法列出所有的磁盘根路径
        File[] roots=File.listRoots();
        System.out.println("系统所有根路径如下");
        for (File root:roots){
            System.out.println(root);

        }
    }
}
```

#### 路径分隔符

windows:  "/" " \ " 都可以

linux/unix:  "/"

如果windows 选择使用 " \ "做分隔符的话，那么请记得 替换成 ”\“，因为 java 中 ”\“ 代表转义字符，所以推荐都使用 "/" ，也可以直接使用 代码 **File.separator** ，表示跨平台分隔符

##### 路径

**相对路径：**

./ 表示当前路径

../ 表示上一级路径

**其中当前路径**：默认情况下 ,java.io 包中的类总是根据当前用户目录来分析相对路径名，此目录由系统属性 user.dir 指定，通常是 Java 虚拟机的调用目录。

**绝对路径**： 绝对路径是完整的路径名，不需要任何其他信息就可以定位自身表示的文件



##### 创建与删除方法

```java
//如果文件存在返回false，否则返回true并且创建文件 
boolean createNewFile();
//创建一个File对象所对应的目录，成功返回true，否则false。且File对象必须为路径而不是文件。只会创建最后一级目录，如果上级目录不存在就抛异常。
boolean mkdir();
//创建一个File对象所对应的目录，成功返回true，否则false。且File对象必须为路径而不是文件。创建多级目录，创建路径中所有不存在的目录
boolean mkdirs()    ;
//如果文件存在返回true并且删除文件，否则返回false
boolean delete();
//在虚拟机终止时，删除File对象所表示的文件或目录。
void deleteOnExit();
```

##### 判断方法

```java
boolean canExecute()    ;//判断文件是否可执行
boolean canRead();//判断文件是否可读
boolean canWrite();//判断文件是否可写
boolean exists();//判断文件是否存在
boolean isDirectory();//判断是否是目录
boolean isFile();//判断是否是文件
boolean isHidden();//判断是否是隐藏文件或隐藏目录
boolean isAbsolute();//判断是否是绝对路径 文件不存在也能判断
```

##### 获取方法

```java
String getName();//返回文件或者是目录的名称
String getPath();//返回路径
String getAbsolutePath();//返回绝对路径
String getParent();//返回父目录，如果没有父目录则返回null
long lastModified();//返回最后一次修改的时间
long length();//返回文件的长度
File[] listRoots();// 列出所有的根目录（Window中就是所有系统的盘符）
String[] list() ;//返回一个字符串数组，给定路径下的文件或目录名称字符串
String[] list(FilenameFilter filter);//返回满足过滤器要求的一个字符串数组
File[]  listFiles();//返回一个文件对象数组，给定路径下文件或目录
File[] listFiles(FilenameFilter filter);//返回满足过滤器要求的一个文件对象数组
```

#### 文件过滤器

上面方法其中包含了一个重要的接口FileNameFilter，该接口是个文件过滤器，包含了一个`accept(File dir,String name)`方法，该方法依次对指定File的所有子目录或者文件进行迭代，按照指定条件，进行过滤，过滤出满足条件的所有文件。

```java
public class Test {
    public static void main(String[] args) throws IOException {
        //以当前路径来创建一个File 对象
        File file=new File(".");
        //创建临时文件
        File tempFike=File.createTempFile("aaa",".java",file);
        //虚拟机退出时删除该文件
        tempFike.deleteOnExit();
        //文件过滤：如果文件名以.java结尾，或者文件对应一个路径，则返回true
        String [] namList=file.list(new FilenameFilter() {
            @Override
            public boolean accept(File dir, String name) {
                return name.endsWith(".java");
            }
        });
        
        //new File(name).isDirectory()  测试文件是否为目录
        String[] namList2=file.list((dir,name)->name.endsWith(".java")||new File(name).isDirectory());
        //遍历
        for (String name:namList2){
            System.out.println(name);
        }
    }
}
```



## IO流

### 概念

Java 的 IO 流是实现输入/输出的基础，他可以方便的实现数据的输入/输出操作，在 Java 中 把不同的 输入/输出源抽象表达为 “流”，流是一组有顺序，有起点和终点的字节集合，是对数据传入的总称或抽象。即数据在两设备间的传输称为流，流的本质是数据传输，根据数据传入特性将流抽象为各种类，方便更直观的进行数据的操作。

**流有输入和输出，输入时是流从数据源流向程序。输出是时从程序传向数据源，而数据源可以是内存，文件，网络或程序等**。

### IO流的分类

#### 输入流和输出流

> 输入流与输出流是以 内存的角度来考虑。

1. 输入流：只能从中读取数据，而不能向其写入数据。
2. 输出流：只能向其写入数据，而不能从中读取数据。

 如下如所示：对程序而言，向右的箭头，表示输入，向左的箭头，表示输出。

![img](http://upload-images.jianshu.io/upload_images/3985563-9836dbd2ae2424d0.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

#### 字节流和字符流

字节流和字符流用法几乎一样，区别在于字节流和字符流所操作的数据单元不同。

字符流的由来：因为数据编码的不同，而又了对字符进行高效操作的流对象。本质其实就是基于字节流读取时，去查了指定的码表。字节流和字符流的区别：

1. 读写单位不同：字节流以字节(8bit) 为单位，字符流以字符(16bit)为单位，根据码表映射字符，一次可能读多个字节。
2. 处理对象不同：字节流能处理所有类型的数据(如图片，avi等)，而字符流只能处理字符类型的数据。

只要是处理纯文本数据，就优先考虑使用字符集。除此之外都使用字节流。

#### 字节流和处理流

按照流的角色来分，可以分为节点流和处理流。

可以从/向一个特定的 IO 设备(如磁盘，网络) 读/写 数据的流，称为节点流，节点流也被称为低级流。

处理流是对一个已存在的流进行连接或封装，通过封装后的流来实现数据读/写功能，处理流也被称为高级流

```JAVA
//节点流，直接传入的参数是 IO 设备
FileInputStream fis=new FileInputStream("test.text");
//处理流,直接传入的参数是流对象
BufferedInputStream bis=new BufferedInputStream(fis);
```

![img](https://img-blog.csdnimg.cn/20190410111416643.png)

**当使用处理流进行输入输出是，程序并不会直接连接到实际的数据源，没有和实际的输入/输出节点连接。使用处理流的一个明显好处是，只要使用相同的处理流，程序就可以采用完全相同的输入/输出 代码来访问不同的数据源，随着处理流所包装节点流的变化，程序实际所访问的数据源也相应的发生变化。**

实际上，Java使用处理流来包装节点流是一种典型的装饰器设计模式，通过使用处理流来包装不同的节点流，即可以消除不同节点流的实现差异，也可以提供更方便的方法来完成输入/输出的功能。因此处理流也被称为包装流。



#### 流的概念模型

![img](http://upload-images.jianshu.io/upload_images/3985563-38c3ea4562d6dbe3.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

Java 的IO流共设计 40 多个类，这些类看上去复杂，但实际上非常规则，而且彼此之间存在非常紧密的联系。它们都是从如下4 个抽象基类派生的。

- InputStream/Reader：所有输入流的基类，前者是字节输出流，后者是字符输出流

- OutputStream/Writer：所有输出流的基类，前者是字节输出流，后者是字符输出流。

  

**InputStream/Reader**

对于InPutStream 和Reader 而言，它们把输入设备抽象成一个水管，这个水管里的每个水滴一次排列，如图

![img](https://img-blog.csdnimg.cn/20190410112326378.png)

字节流和字符流的处理方式其实非常相似，只是它们处理的输入/输出单位不同而言。**输入流使用隐式的记录指针来表示 **当前正准备从哪个“水滴”开始读取，每当程序从 InputStream或 Reader 里取出一个或多个 “水滴” 后，记录指针自动向后移动；除此之外，InputSteam 和 Reader里都提供了一些方法来控制记录指针的移动。



##### **OutputStream/Writer**

对于OutputStream 和 Writer而言，它们同样把输出设备抽象成一个水管，只是这个水管里没有任何水滴。

![img](https://img-blog.csdnimg.cn/20190410113012435.png)

 当执行输出时，程序相当于依次把 “水滴” 放入到输出流的水管中，输出流同样采用隐式的记录指正来标识当前属地即将放入的位置，每当程序向 OutPutStream 或 Writer 里输出一个或多个水滴后，记录指针自动向后移动。



以上就是Java IO流的基本概念模型，除此之外，Java 的处理流模型则体现了 Java 输入/输出流设计的灵活性。处理流的功能主要体现在以下方面：

- 性能的提高：主要以增加缓冲的方式来提高输入/输出的效率。
- 操作的便捷：处理流可能提供了一系列便捷的方法来一次输入/输出大批量的内容，而不是输入/输出一个或多个水滴

处理流可以 嫁接 在任何已存在的流的基础之上，这就允许 java 应用程序采用相同的代码，透明的方式来访问不同的输入，输出设备的数据流。

![img](https://img-blog.csdnimg.cn/20190410130838235.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BldHRlcnA=,size_16,color_FFFFFF,t_70)

通过使用处理流，Java 程序无须理会 输入/输出 节点是磁盘，网络还是其他的输入/输出设备，程序只要讲这些节点包装成处理流，就可以使用相同的输入/输出代码来读写不同的输入/输出设备的数据。



##### InputStream,Reader

> InputStream和Reader 都是将输入数据抽象成一条水管，所以程序既可以通过 read() 方法每次读取一个 水滴，也可以通过 read(char[] cbuf)或rea(byte[] b)方法来读取多个 “水滴”。当使用数组作为 read() 方法的参数时，可以理解为使用一个 “竹筒”到水管去取水，而read(char[] cbuf)方法中的数组就相当于一个 “竹筒” ,程序每次调用输入流的 read(char[] cbuf)或read(byte[] b)方法，就相当于用 “竹筒”从输入流中取出一筒 ”水滴“，程序得到 “竹筒” 里的 “水滴” 后，转换成相应的数据即可；**程序多次重复这个取水过程，直到 read(char[] cbuf)或read(byte[] b)方法返回-1,表明到了输入流的结束点。内部每次又使用指针移动取水的下标。**

InputStream是所有输入字节流的父类，是一个抽象类，主要包含三个方法

```java
//读取一个字节并以整数的形式返回(0~255),如果返回-1已到输入流的末尾。 
int read() ； 
//读取一系列字节并存储到一个数组buffer，返回实际读取的字节数，如果读取前已到输入流的末尾返回-1。 
int read(byte[] buffer) ； 
//读取length个字节并存储到一个字节数组buffer，从off位置开始存,最多len， 返回实际读取的字节数，如果读取前以到输入流的末尾返回-1。 
int read(byte[] buffer, int off, int len) ；
```

Demo

```java
public class Test {
    public static void main(String[] args) throws IOException {
        try{
            //创建字节输入流
            FileInputStream fis=new FileInputStream("E:\\Android\\Mvp_Test\\app\\src\\main\\java\\com\\example\\pettepr\\mvp_test\\aaa.txt");
            //创建一个长度为32的字节数组
            byte[] bbuf=new byte[32];
            //用于保存实际读取的字节数
            int hasRead=0;
            while ((hasRead=fis.read(bbuf))>0){
                System.out.println(new String(bbuf,0,hasRead));
            }

        }catch (IOException e){
            e.printStackTrace();
        }
    }
}
```

解决中文乱码问题

```java
//创建字节输入流
FileInputStream fis = new FileInputStream("E:\\Android\\Mvp_Test\\app\\src\\main\\java\\com\\example\\pettepr\\mvp_test\\aaa.txt");
InputStreamReader input = new InputStreamReader(fis, "GBK");
BufferedReader bbuf = new BufferedReader(input);
String next;
while ((next=bbuf.readLine())!=null) {
    System.out.println(next);
}
```

**InputStream与Reader最大不同，一个是操作字节，一个是操作字符**

除了上面的方法之外，InputStream和Reader还支持如下方法来移动流中的指针位置：

```java
//在此输入流中标记当前的位置
//readlimit - 在标记位置失效前可以读取字节的最大限制。
void mark(int readlimit)
// 测试此输入流是否支持 mark 方法
boolean markSupported()
// 跳过和丢弃此输入流中数据的 n 个字节/字符
long skip(long n)
//将此流重新定位到最后一次对此输入流调用 mark 方法时的位置
void reset()
```



##### OutputStream 和 Writer

OutputStream 是所有的输出字节流的父类，它是一个抽象类，主要包含如下4个方法：

```java
//向输出流中写入一个字节数据,该字节数据为参数b的低8位。 
void write(int b) ; 
//将一个字节类型的数组中的数据写入输出流。 
void write(byte[] b); 
//将一个字节类型的数组中的从指定位置（off）开始的,len个字节写入到输出流。 
void write(byte[] b, int off, int len); 
//将输出流中缓冲的数据全部写出到目的地。 
void flush();
```

Writer 是所有的输出字符流的父类，它是一个抽象类,主要包含如下六个方法：

```java
//向输出流中写入一个字符数据,该字节数据为参数b的低16位。 
void write(int c); 
//将一个字符类型的数组中的数据写入输出流， 
void write(char[] cbuf) 
//将一个字符类型的数组中的从指定位置（offset）开始的,length个字符写入到输出流。 
void write(char[] cbuf, int offset, int length); 
//将一个字符串中的字符写入到输出流。 
void write(String string); 
//将一个字符串从offset开始的length个字符写入到输出流。 
void write(String string, int offset, int length); 
//将输出流中缓冲的数据全部写出到目的地。 
void flush()
```

可以看出，Writer比OutputStream多出两个方法，主要是支持写入字符和字符串类型的数据。

Demo

```java
public class Test {
    public static void main(String[] args) throws IOException {
        try{
            //创建字节输入流
            FileInputStream fis=new FileInputStream("E:\\Android\\Mvp_Test\\app\\src\\main\\java\\com\\example\\pettepr\\mvp_test\\Test.java");

            //创建字节输出流
            FileOutputStream fos=new FileOutputStream("E:\\Android\\Mvp_Test\\app\\src\\main\\java\\com\\example\\pettepr\\mvp_test\\aaa.txt"，true); 	//true指将内容添加到原内容后面
             byte[] buuf=new byte[32];
            int hashRead=0;
            //循环从输入流中取出数据
            while((hashRead=fis.read(buuf))>0){
                fos.write(buuf,0,hashRead);
            }
        }catch (IOException e){
            e.printStackTrace();
        }
    }

```

**使用Java的IO流执行输出时，不要忘记关闭输出流，关闭输出流除了可以保证流的物理资源被回收之外，还能将输出流缓冲区的数据flush到物理节点里（因为在执行close()方法之前，自动执行输出流的flush()方法）**



### 处理流

处理流可以隐藏底层设备上节点流的差异，并对外提供更加方便的输入/输出方法，让程序员只需关心高级流的操作。

使用处理流时的典型思路是，使用处理流老包装节点流，程序通过处理流来执行输入输出功能，让节点流与底层的 I/O设备，文件交互。

它的优势为以下两点：

1. 对开发人员来说，使用处理流进行输入、输出操作更简单
2. 使用处理流效率更高

```java
public class Test {
    public static void main(String[] args){
        try{
            FileOutputStream fos=new FileOutputStream("E:\\Android\\Mvp_Test\\app\\src\\main\\java\\com\\example\\pettepr\\mvp_test\\aaa.txt");
            PrintStream ps=new PrintStream(fos);
             //使用PrintStream执行输出
            ps.println("Petterp");
        }catch (Exception e){
        }
    }
}
```



### 转换流

转换流用于实现将字节流转换成字符流。

IntputStreamReader 将字节输入流转换成字符输入流，OutputStreamWriter 将字节输出流转换成字符输出流。

#### 为什么没有把字符流转成字节流的呢？

字节流比字符流的应用范围更广，但字符流比字节流操作方便。如果有一个流已经是字符流了，也就是说，是一个用起来更方便的流，为什么要转换成字节流呢？反之，如果现在有一个字节流，但可以确定这个字节流的内容都是文本内容，那么把他转为字符流来处理就会更方便一些，所以 java只提供了 将字节流转换成字符流的转换。

```java
public class Test {
    public static void main(String[] args) throws IOException {
        //创建字节输入流
        FileInputStream fis = new FileInputStream("E:\\Android\\Mvp_Test\\app\\src\\main\\java\\com\\example\\pettepr\\mvp_test\\aaa.txt");
        InputStreamReader input = new InputStreamReader(fis, "GBK");
        //将普通的 Reader包装成 BufferedReader
        BufferedReader bbuf = new BufferedReader(input);
        String next;
        //readLine() 每次读入一行
        while ((next=bbuf.readLine())!=null) {
            System.out.println(next);
        }
    }
}
```

