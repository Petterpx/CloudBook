# NIO

### 概念

javaNio 是一个可以替代标准 Java IO的Api的 IOApi，Java Nio提供了与标准IO 不同的 IO工作方式。

所以 Java NIO 是一种新式的 IO 标准，与之前的 普通 IO 的工作方式不同。标准 的 IO 基于字节流和字符流进行操作的，而 NIO 是基于通道(channel) 和缓冲区(Buffer) 进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入通道也类似。

**由上面的定义就说明Nio是一种新型的 IO ，但 NIO 不仅仅就是等于 非阻塞 IO，NIO 中有实现非阻塞 IO 的具体类，但不代表NIO 就是 非阻塞IO**

Java NIO 由以下几个核心部分组成：

- Buffer
- Channnel
- Selector

传统的 IO 操作面向数据流，意味着每次从流中读一个或多个字节，直至完成，数据没有被缓存在任何地方，因此效率不高。NIO 操作面向缓冲区，数据从 Channel 读取到 Buffer 缓冲区，随后在 Buffer中处理数据。

Channel(通道)和Buffer(缓冲)是新 IO 中的两个核心对象，Channel 是对传统的输入/输出 系统的模拟，在新IO 系统中所有的数据都需要通过通道传输；Channel 与传统的 InputStream，OutStream 最大的区别在于他提供了一个 map() 方法，通过该 map()方法可以直接将 "一块数据" 映射到内存中。如果说传统的输入/输出 系统是面向流的处理，则新 IO 则是面向块的处理.

Buffer 可以被理解成一个 容器,它的本质是一个数组，发送到 Channel 中的所有对象都必须首先放到 Buffer 中，而从 Channel 中读取的数据也必须先放到 Buffer 中。此处的 Buffer 有点类似 一个 竹筒，但该 Buffer 即可以像 “竹筒”那样一次次去 Channel 中取水，也允许用 Channel 直接将 文件的某块数据映射成 Buffer。

### Buffer的使用



利用 Buffer读写数据，通常遵循四个步骤：

1. 把数据写入 buffer
2. 调用 flip
3. 从 Buffer中读取数据
4. 调用 buffer.clear

当写入数据到 buffer 中时，buffer 会记录已经写入的数据大小。当需要读数据是，通过 flip() 方法吧 buffer 从写模式调整为读模式；在读模式下，可以读取所有已经写入的数据。

当读取完数据后，需要清空buffer，以满足后续写入操作。清空 buffer 有两种方式：调用clear() ，一旦读完 Buffer 中的数据，需要让Buffer 准备好再次被写入，clear 会恢复状态值，但不会擦除数据。

在Buffer中有3个重要的概念：容量(capacity)，界限(limit)，和位置(postion)

- 容量：缓冲区的容量表示该 Buffer 的最大数据容量，即最多可以存储多少数据。缓冲区的容量不可能为负值，创建后不能改变。
- 界限：第一个不应该被读出或者写入的缓冲区位置索引。也就是说，位于 limit 后的数据既不可读，也不可被写
- 位置：用于指明下一个可以被读取的或者写入的缓冲区位置索引(类似于IO 流中的记录指针)。当使用Buffer从Channel 中读取数据时，postion 的值恰好等于已经读到了多少数据。当刚刚新建了一个 Buffer对象时，其 postion 为0，如果 Chananel 中读取了 2个数据到该 Buffer 中，则postion 为2

```java
public class Test {
    public static void main(String[] args) throws IOException {
        //创建Buffer，容量为8
        CharBuffer buffer=CharBuffer.allocate(8);
        //查看容量
        System.out.println("caoacity:"+buffer.capacity());
        //查看界限
        System.out.println("limit:"+buffer.limit());
        //查看位置
        System.out.println("postion:"+buffer.position());
        buffer.put('a');
        buffer.put('b');
        buffer.put('c');
        System.out.println("加入3个元素后，postion="+buffer.position());

        //调用flip()方法
        //界限移动到原来poston位置，postion会到初始值，相当于封印没有数据的存储空间
        buffer.flip();
        System.out.println("执行flip()后，limit="+buffer.limit());
        System.out.println("postion="+buffer.position());

        //取出第一个元素
        System.out.println("第一个元素(postion=0)"+buffer.get());
        System.out.println("取出第一个元素后,postion="+buffer.position());

        //调用clear()方法
        //界限变为与容量相等，psotion回到初始值
        buffer.clear();
        System.out.println("执行 clear()后，limit="+buffer.limit());
        System.out.println("执行clear()后，postion="+buffer.position());
        System.out.println("执行clear()后，缓冲区内容并没有被清除"+"第三个元素为："+buffer.get(2));
        System.out.println("执行绝对读取后，postion="+buffer.position());
    }
}
```

![img](https://img-blog.csdnimg.cn/20190410205542249.png)

![img](https://img-blog.csdnimg.cn/20190410205554640.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BldHRlcnA=,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20190410205608605.png)

流程如上图所示。

通过allcate() 方法创建的 Buffer对象时普通 Buffer，ByteBuffer 还提供了一个 allocateDirect() 方法来创建直接 Buffer。直接 Buffer 的创建成本比普通Buffer 的创建成本高，但直接 Buffer的读取效率更高。



## Channel

Channel类似于传统的流对象，但与传统的流对象有两个主要区别。

1. Channel可以直接将指定文件的部分或全部直接映射成Buffer

2. 程序不能直接访问Channel 中的数据，包括读取，写入都不行，Channel 只能与 Buffer 进行交互。也就是说，如果要从 Channel 中取得数据，必须先用 Buffer 从Channel 中取出一些数据，然后让程序从 Buffer 中取出这些数据：如果要将程序中的数据写入 Channel,一样先让程序将输入放入 Buffer 中，程序再将Buffer 里的数据写入 Channel中。

   简单来说就是

   > - 通道可以读也可以写，流一般来说是单向的(只能读或者写)
   > - 通道可以异步读写
   > - 通道总是局域缓冲区 Buffer 来读写
   > - 我们可以从通道中读取数据，写入到buffer，也可以从 buffer内读数据，写入到通道中。
   > - ![img](http://upload-images.jianshu.io/upload_images/3985563-5dcaaf9b7106a7d9.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

Channel 的实现类有：

- FildeChannel    用于文件的数据读写
- DatagramChannel  用于 UDP的数据读写
- SocketChannel  用于 TCP的数据读写
- ServerSocketChannel   允许我们监听TCP链接请求，每个请求会创建一个 SocketChannel

需要注意的是：所有的 Channel 都不应该通过构造器来直接创建，而是通过传统的节点 InputStream ，OutPutStrea,的getChannel() 方法来返回对应的 Channel,不同的节点流获取的 Channel 不一样。例如，FileInputStream 的 getChannel()返回的是 FiledChannel。



