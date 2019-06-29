# NIO

> Java NIO 是从Java 1.4版本开始引入的一个新的 IO API，可以替代标准的 Java IO API。NIO与原来的 IO 有同样的作用和目的，但是使用的方式完全不同，NIO 支持面向 **缓冲区** 的，基于 **通道** 的IO 操作，至于什么是缓冲区，什么是通道，接下来我将会用大白话一一说明。总之，**NIO 就是以更高效的方式进行文件的读写操作**。

 在学习本篇之前，首先你要对 IO 有一定的了解。当然不了解的话，也可以看得哈哈，我会说的很通俗易懂。

我们先看看 Java NIO 与 IO的主要区别：

| IO                      | NIO                         |
| ----------------------- | --------------------------- |
| 面向流(Stream Oriented) | 面向缓冲区(Buffer Oriented) |
| 阻塞IO(Blocking IO)     | 非阻塞IO（Non Blocking IO） |
| 无                      | 选择器(Selectors)           |

上面的什么 面向缓冲区，又什么非阻塞IO,又是选择器的，这些到底都啥啊，拍桌子。。。

下面我会在本篇中对上面出现的概念及盲点进行解析。



### IO与NIO的区别

> 首先我们看看他们的区别
>
> 
>
> **为什么说IO是面向流，那流又是什么呢？**
>
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190410111459614.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQwMzg5Nzc1,size_16,color_FFFFFF,t_70)
>
> 我们先看上面的图片。
>
> 我们在学IO的时候，肯定都听过这样的例子，IO流就相当于一条管道，它里面所有的操作都是单向的。如果要把文件中的数据拿到程序中，需要建立一条通道。想把程序中的数据存到文件中也需要建立一条通道。所以我们成io流是单向的。**因为io管道里实际面对的是字节的流动，所以我们称io流为面向流。**
>
> 
>
> **那什么说NIO是面向缓冲区呢？**
>
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190410111324871.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQwMzg5Nzc1,size_16,color_FFFFFF,t_70)
>
> **你可以这样想象，通道就相当于与一条道路，缓冲区相当于出租车，出租车上拉的是乘客，出租车可以上乘客也可以下乘客。回到NIO 上面，通道就是一条道路，他负责提供行驶的绝对条件，即就是有路啊，这样出租车才能基本出行，而缓冲区在这里是出租车，出租车里面坐的是人，出租车负责将乘客送到它要去的地方，当然，出租车不受限制，他可以在任意地方。所以简而言之，通道(Channel)负责传输，Buffer 负责存储。**



在 NIO 里面，有两个特别重要的东西，那就是 **通道(Channel)** 与 **缓冲区(Buffer)**

Java NIO系统的核心在于：通道 和缓冲区。通道表示 打开IO 设备(例如：文件，套接字)的链接。若需要使用 NIO 系统，需要获取用于链接 IO 的设备的通道以及用于容纳数据的缓冲区。然后操作缓冲区，对数据进行处理。



### 缓冲区(Buffer)



```java
/*缓冲区(Buffer)：在Java Nio中负责数据的存取。缓冲区就是数组。用于存储不同数据类型的数据
*
* 根据数据类型不同(boolean 除外)，提供了相应类型的缓冲区
* ByteBuffer
* CharBuffer
* ...
*
* 上述缓冲区的管理方式几乎一致，都是通过allocate() 获取缓冲区
*
* 2/缓冲区存取数据的两个核心方法：
* put(): 存入数据到缓冲区中
* get()：获取缓冲区中的数据
*
* 4.缓冲区中的4个核心属性：
* capacity:  容量，表示缓冲区中最大存储数据的容量。一旦声明不能改变,(底层就是数组)
* limit：界限，表示缓冲区中可以操作数据的大小。(limit 后面的数据不能进行读写)
* position：位置，表示缓冲区中正在操作数据的位置。
*
* 5.直接缓冲区与非直接缓冲区
* 非直接缓冲区：通过 allocate()方法分配缓冲区，将缓冲区建立在 JVM的内存中
* 直接缓冲区：通过 allocateDirect() 方法分配直接缓冲区，将缓冲区建立物理内存中。
*
*  mark: 标记，表示记录当前 postion 的位置，可以通过 reset() 恢复到mark 位置
* position<=limit<=capacity
* */

public class Test {
    public static void main(String[] args) throws IOException {
        test2();
        test3();
    }

    private static void test1() {
        String str="Petterp";
        //1.分配一个指定大小的缓冲区
        ByteBuffer buf = ByteBuffer.allocate(1024);

        System.out.println("____________allocate_________");
        System.out.println(buf.position());
        System.out.println(buf.limit());
        System.out.println(buf.capacity());

        //2.利用 put() 存入数据到缓冲区中
        buf.put(str.getBytes());
        System.out.println("____________put_________");
        System.out.println(buf.position());
        System.out.println(buf.limit());
        System.out.println(buf.capacity());

        //3.切换读取数据模式
        buf.flip();
        System.out.println("____________flip_________");
        System.out.println(buf.position());
        System.out.println(buf.limit());
        System.out.println(buf.capacity());

        //4.利用get() 读取缓冲区的数据
        byte[] dst=new byte[buf.limit()];
        buf.get(dst);
        System.out.println(new String(dst,0,dst.length));
        System.out.println("____________get_________");
        System.out.println(buf.position());
        System.out.println(buf.limit());
        System.out.println(buf.capacity());

        //5.rewind()_可重复读数据
        buf.rewind();
        System.out.println("____________rewind_________");
        System.out.println(buf.position());
        System.out.println(buf.limit());
        System.out.println(buf.capacity());

        //6.clear():  清空缓冲区.(缓冲区数据还在，但是处于"被遗忘"状态
        // 因为limit这些值全回到了初始状态，所以无法正确读取数据。)
        buf.clear();
        System.out.println("____________clear_________");
        System.out.println(buf.position());
        System.out.println(buf.limit());
        System.out.println(buf.capacity());
    }

    private static void test2(){
        ByteBuffer buf =ByteBuffer.allocate(1024);
        String res="Petterp";
        buf.put(res.getBytes());
        buf.flip();
        //记录指针位置为0
        buf.mark();
        System.out.println("____________mark记录position位置_________");
        System.out.println(buf.position());
        byte[] bytes = new byte[buf.limit()];
        buf.get(bytes,0,3);
        System.out.println("打印get到的数据"+new String(bytes,0,bytes.length));
        System.out.println("____________get之后position_________");
        System.out.println(buf.position());

        //会到记录的指针位置
        buf.reset();
        System.out.println("____________reset之后position_________");
        System.out.println(buf.position());

        System.out.println("____________remaining判断可操作数据长度_________");
        //判断缓冲区是否还有剩余数据
        if (buf.hasRemaining()){
            //获取缓冲区中可以操作的数据长度
            System.out.println(buf.remaining());
        }
    }
    
    private  static void test3(){
        ByteBuffer buf=ByteBuffer.allocateDirect(1024);
        //判断是否是直接缓存区
        System.out.println(buf.isDirect());
    }
}
```



#### 非直接缓冲区与缓冲区的区别

> 非直接缓冲区在，建立在JVM内存中，实际读写数据时，需要在 OS 和JVM之间进行数据拷贝。
>
> ![img](https://upload-images.jianshu.io/upload_images/12064261-bb3c8a7c5264521e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/812/format/webp)
>
> 为什么不直接让磁盘控制器把数据送到用户控件的缓冲区呢？
>
> 因为我们的硬件通常不能直接访问用户内存空间。如果有一个程序需要读写磁盘空间，出于系统安全考虑，磁盘中的文件无法直接传输到我们程序中，它必须经过系统的内核地址空间的缓存中，然后将内核地址空间数据复制到用户地址空间，这样数据才可以传输到我们的应用程序。

> 
>
> 内存映射空间
>
> **直接缓冲区**，缓冲区建立在受操作系统管理的物理内存中，OS和JVM直接通过这块物理内存进行交互，没有了中间的拷贝环节
>
> ![img](https://upload-images.jianshu.io/upload_images/12064261-9633cdcbe71515a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/790/format/webp)
>
> 但是直接缓冲区也有很多弊端：
>
> - 内存消耗大(分配与销毁不易控制)
> - 如果当Java 程序将数据写到物理内存中后，这个时候我们就无法管理这块内存，只能由系统进行控制。

```java
//1.利用通道完成文件的复制（非直接缓冲区）
public static void test1(){
    try{
        long l = System.currentTimeMillis();
        fis = new FileInputStream("D:1.zip");
        fos = new FileOutputStream("D:2.zip");

        //1.获取通道
        inChannel = fis.getChannel();
        outChanel1 = fos.getChannel();

        //2.分配指定大小的缓冲区
        ByteBuffer buf = ByteBuffer.allocate(1024);

        //3.将通道中的数据存入缓冲中
        while (inChannel.read(buf)!=-1){
            buf.flip();//切换读取数据模式
            //将缓冲区中的数据写入通道中
            outChanel1.write(buf);
            buf.clear();  //清空缓冲区
        }
        long l2 = System.currentTimeMillis();
        System.out.println("时间"+(l2-l));
    }catch (IOException e){
        e.printStackTrace();
    }finally {
        if (outChanel1 != null) {
            try {
                outChanel1.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (inChannel != null) {
            try {
                inChannel.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (fos != null) {
            try {
                fos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (fis != null) {
            try {
                fis.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
//使用直接缓冲区完成文件的复制(内存映射文件)
@RequiresApi(api = Build.VERSION_CODES.O)
public static  void test2(){
    try {
        long l = System.currentTimeMillis();
        //第一个参数是路径，第二个参数是模式
        FileChannel inchannel=FileChannel.open(Paths.get("D:demo.txt"),StandardOpenOption.READ);
        FileChannel outChannel=FileChannel.open(Paths.get("D:demo2.txt"),StandardOpenOption.WRITE,StandardOpenOption.READ,StandardOpenOption.CREATE);

        //内存映射文件
        MappedByteBuffer inMapBuf = inchannel.map(FileChannel.MapMode.READ_ONLY, 0, inchannel.size());
        MappedByteBuffer outMapBuf = outChannel.map(FileChannel.MapMode.READ_WRITE, 0, inchannel.size());
        //直接对缓冲区进行数据的读写操作
        byte[] bytes = new byte[inMapBuf.limit()];
        inMapBuf.get(bytes);
        outMapBuf.put(bytes);

        inchannel.close();
        outChannel.close();
        long l2 = System.currentTimeMillis();
        System.out.println("时间"+(l2-l));
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```





### 通道(Channel)

> 通道(Channel) 由java.nio.channels 包定义的。Channel 表示 IO 源于目标打开的链接。Channel 类似于传统的流，只不过 Channel 本身不能直接访问数据，Channel 只能与 Buffer进行交互。

```
通道的主要实现类：
* Java.nio/channels.Channel 接口
*       FileChannel     本地文件传输
*       SocketChannel       网络传输
*       ServerSocketChannel
*       DatagramChannel

```

```
		获取通道
* -1. Java 镇对支持通道的类提供了 getChannel() 方法
*       本地IO
*       FiledInputStream/FileOutputStream
*       RandomAccessFile
*
*       网络IO
*       Socket
*       ServerSocket
*       DatagramSocket
* -2. 在 JDK 1.7中的 NIO.2 针对各个通道提供了静态方法 open()
*
* -3. 在 jdk 1.7中的 NIO.2 的 Files 工具类 newByteChannel()
```

```java
//1.利用通道完成文件的复制（非直接缓冲区）
public static void test1(){
    try{
        long l = System.currentTimeMillis();
        fis = new FileInputStream("D:1.zip");
        fos = new FileOutputStream("D:2.zip");

        //1.获取通道
        inChannel = fis.getChannel();
        outChanel1 = fos.getChannel();

        //2.分配指定大小的缓冲区
        ByteBuffer buf = ByteBuffer.allocate(1024);

        //3.将通道中的数据存入缓冲中
        while (inChannel.read(buf)!=-1){
            buf.flip();//切换读取数据模式
            //将缓冲区中的数据写入通道中
            outChanel1.write(buf);
            buf.clear();  //清空缓冲区
        }
        long l2 = System.currentTimeMillis();
        System.out.println("时间"+(l2-l));
    }catch (IOException e){
        e.printStackTrace();
    }finally {
        if (outChanel1 != null) {
            try {
                outChanel1.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (inChannel != null) {
            try {
                inChannel.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (fos != null) {
            try {
                fos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (fis != null) {
            try {
                fis.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```





#### 通道之间的数据传输

* transferFrom()
* transferTo()

```java
  public static void test3(){
        try {
            FileChannel inchannel=FileChannel.open(Paths.get("D:demo.txt"),StandardOpenOption.READ);
            FileChannel outChannel=FileChannel.open(Paths.get("D:Demop.txt"),StandardOpenOption.READ,StandardOpenOption.WRITE,StandardOpenOption.CREATE);
            if (outChannel != null) {
//                inchannel.transferTo(0,inchannel.size(),outChannel);
                outChannel.transferFrom(inchannel,0,inchannel.size());
                inchannel.close();
                outChannel.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
```



### 分散(Scatter)与聚集(Gather)

![1554968002880](C:\Users\Pettepr\AppData\Roaming\Typora\typora-user-images\1554968002880.png)

* 分散读取(Scattering Reads):将通道中的数据分散到多个缓冲区中
* 聚集写入(Gathering Writes):将多个缓冲区的数据聚集到通道中

    //分散和聚集
    public static void test4() throws IOException {
        RandomAccessFile rafi=new RandomAccessFile("D:demo1.txt","rw");
    
        //1.获取通道
        FileChannel channel=rafi.getChannel();
    
        //2.分配指定大小的缓冲区
        ByteBuffer buf1 =ByteBuffer.allocate(100);
        ByteBuffer buf2 = ByteBuffer.allocate(1024);
    
        //3.分散读取
        ByteBuffer[] bufs={buf1,buf2};
        channel.read(bufs);
        for (ByteBuffer byteBuffer:bufs){
            byteBuffer.flip();
        }
        System.out.println(new String(bufs[0].array(),0,bufs[0].limit()));
        System.out.println("----------");
        System.out.println(new String(bufs[1].array(),0,bufs[1].limit()));
    
        //4.聚集写入
        RandomAccessFile raf2=new RandomAccessFile("D:demo2.txt","rw");
        FileChannel channel2=raf2.getChannel();
    
        channel2.write(bufs);
    }




### 字符集：Charset

> 计算机里的文件，数据，图片文件只是一种表面现象，所有文件在底层都是二进制文件，即全部都是字节码。
>
> 对于文本文件而言，之所以可以看到一个个的字符，这完全是因为系统将底层的二进制序列转换成字符的缘故。在这个过程中涉及两个概念：编码(Encode) 和解码 (Decode)，通常而言，把明文的字符序列转换成计算机理解的二进制序列称为编码，把二进制序列转换成普通人能看懂的明文字符串称为解码。

> Java 默认视同 Uniocde 字符集，但很多操作系统并不适用Unicode 字符集，那么当从系统中读取数据到 Java程序中时，就可能出现乱码等问题。
>
> JDK1.4 提供了 Charset来处理字节序列和字符序列(字符串)之间的转换关系，该类包含了用于创建解码器和编码器的方法。还提供了获取 Charset所支持字符集的方法，Charset类是不可变的。

* 编码：字符串 -> 字节数组
* 解码：字节数组 -> 字符串

 public static void test6() throws CharacterCodingException {
        //字符集
        Charset cs1 = Charset.forName("GBK");

```java
    //查看Java支持的字符集格式
    private static void test5(){
        SortedMap<String, Charset> map = Charset.availableCharsets();

        Set<Map.Entry<String, Charset>> set = map.entrySet();
        for (Map.Entry<String,Charset> entry: set){
            System.out.println(entry.getKey()+"="+entry.getValue());
        }
    }

	//获取编码器
    CharsetEncoder ce = cs1.newEncoder();

    //获取解码器
    CharsetDecoder cd=cs1.newDecoder();
    CharBuffer cBuf = CharBuffer.allocate(1024);
    cBuf.put("我是Petterp");
    cBuf.flip();

    //编码
    System.out.println(cBuf.limit());
    ByteBuffer bBuf = ce.encode(cBuf);
    for (int i=0;i<11;i++){
        System.out.println(bBuf.get());
    }

    bBuf.flip();
    CharBuffer cBuf2 = cd.decode(bBuf);
    System.out.println(cBuf2.toString());

    System.out.println("__________");
    //获得解码器
    Charset cs2=Charset.forName("GBK");
    bBuf.flip();
    CharBuffer cBuf3 = cs2.decode(bBuf);
    System.out.println(cBuf3.toString());
}
```
#### 二进制序列和字符之间如何对应呢？

> 为了解决二进制序列与字符之间的对应关系，这就需要字符集了。所谓字符集，就是为每个字符编个号码而已。任何人都可以制定自己独有的字符集明知要为每个字符编个号码即可。当然，如果每个人都制定自己独有的字符集，那程序就没法交流了。





Java7的 NIO.2

