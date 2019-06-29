# RandomAccessFile

RandomAccessFild 是 Java 输入/输出 体系中功能最丰富的的文件内容访问类，它既可以读取文件内容，也可以项文件输出数据。与普通的输入/输出流不同的是，RandomAccessFile 支持 “随机访问” 的方式，程序可以直接跳转到文件的任意地方来读写数据。

由于RandomAccessFile 可以自由访问文件的任意位置，**所以如果需要访问文件部分的内容，而不是把文件从头读到尾，使用 RandomAccessFile将是更好的选择**

与 OutputStream,Writer 等输出流不同的是，RandomAccessFile 允许自由定义文件记录指针，RandomAccessFile 可以不从开始的地方开始输出，因此 RandomAccessFile 可以向已存在的文件后追加内容。**如果程序需要向已存在的文件后追加内容，则应该使用 RandomAccessFile**

RandomAccessFile 的方法虽然多，但他有一个最大的局限，就是只能读写文件，不能读写其他 IO 节点。

RandomAccessFile 对象也包含了一个记录指针，用以标识当前读写处的位置，当程序新创建一个 RandomAccessFile 对象时，该对象的文件记录指针位于文件头(也就是0处)，当读/写了 n 个字节后，文件记录指针将会向后一定 n 个字节。除此之外，RandomAccessFile 可以自由移动该记录指针，即可以向前移动，也可以向后移动。RandomAccessFile 包含了如下来年各个方法来操作文件记录指针。

- long getFilePointer();  返回文件记录指针的当前位置
- void seek(long pos):  将文件记录指针定位到 pos位置。

RandomAccessFile 的一个重要使用场景就是网络请求中的多线程及断电续传。

### RandomAccessFile 中的方法

RandomAccessFile 的构造函数

RandomAccessFile 类有两个构造函数，其实这两个构造函数基本相同，只不过是指定文件的形式不同——一个需要使用String 参数来指定文件名，一个使用 File 参数来指定文件本身。除此之外，创建RandomAccessFile 对象时还需要指定一个 mode 参数，该参数指定 RandomAccessFile 的访问模式，一共有四种。

1. “r” 以只读方式打开指定文件。如果试图对该 RandomAccessFile 执行写入方法，都将抛出IOException 异常。

2. "rw" 以读，写方式打开指定文件，如果该文件尚不存在，则尝试创建该文件

3. "rws"  以读，写方式打开指定文件。相对于”rw“ 模式。还要求对文件的内容或元数据的每个更新都同步写入到底层存储设备。

4. "rwd" 以读，写 方式打开指定文件。相对于 "rw" 模式，还要求对文件内容的每个更新都同步写入到底层存储设备。

   ##### Demo

   ```java
   //输入内容(相当于内存)
   public class Test {
       public static void main(String[] args) throws IOException {
           RandomAccessFile raf=new RandomAccessFile("E:\\Android\\Mvp_Test\\app\\src\\main\\java\\com\\example\\pettepr\\mvp_test\\aaa.txt","rw");
           System.out.println(raf.getFilePointer());
           //移动 raf的文件记录指针位置
           raf.seek(3);
           byte[] buf=new byte[1024];
           int hasRead=0;
           while ((hasRead=raf.read(buf))>0){
               System.out.println(new String(buf,0,hasRead));
           }
       }
   }
   
   //输出内容(相当于内存)
   public class Test {
       public static void main(String[] args) throws IOException {
           RandomAccessFile raf=new RandomAccessFile("E:\\Android\\Mvp_Test\\app\\src\\main\\java\\com\\example\\pettepr\\mvp_test\\aaa.txt","rw");
           //移动 raf的文件记录指针位置
           raf.seek(raf.length());
           //\r\n 控制换行，补充在末尾而不换行
           raf.write("追加的内容\r\n".getBytes());
       }
   }
   
   
   public class Test {
       public static void main(String[] args) throws IOException {
   
           //创建临时文件
           File tep=File.createTempFile("tmp",null);
           //关闭虚拟机时删除临时文件
           tep.deleteOnExit();
   
           RandomAccessFile raf=new RandomAccessFile("E:\\Android\\Mvp_Test\\app\\src\\main\\java\\com\\example\\pettepr\\mvp_test\\aaa.txt","rw");
           //使用临时文件来保存插入点后的数据
           FileOutputStream tmpOut=new FileOutputStream(tep);
           FileInputStream tempIn=new FileInputStream(tep);
           raf.seek(10);
           byte[] buf=new byte[1];
           int hasread=0;
           while((hasread=raf.read(buf))>0){
               tmpOut.write(buf,0,hasread);
           }
   
           raf.seek(10);
           raf.write("我是追加的内容".getBytes());
           while ((hasread=tempIn.read(buf))>0){
               raf.write(buf,0,hasread);
           }
       }
   }
   ```

**当RandomAccessFile 向指定文件中插入内容时，将会覆盖掉原有内容。如果不想覆盖掉，则需要将原有内容读取出来，然后先把插入内容插入后再把原有内容追加到插入内容后**