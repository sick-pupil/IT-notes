**BIO**：基于流模型实现，同步且阻塞，读写输入输出流时，未完成则线程会一直被阻塞，`BIO`很容易成为应用性能瓶颈
**NIO**：`Java1.4`引入`java.nio`，提供`Channel、Selector、Buffer`，可以构建多路复用`IO`流，同步且非阻塞
**AIO**：`Java1.7`引入，`NIO`升级版本，异步且非阻塞，基于事件和回调机制

同步：一个任务完成需要依赖另外一个任务，主动依赖的任务需要等待被动依赖的任务完成，才能算是完成，即依赖的任务之间会出现先后顺序
异步：两个互相依赖的任务不需要等待，主动依赖的任务只需要完成自己的任务以及通知被动依赖的任务需要完成什么工作

阻塞：`CPU`需要停下来等待慢操作完成才会继续执行
非阻塞：`CPU`不需要等待慢操作，慢操作被委托执行，同时`CPU`继续执行其他操作

| 组合方式 | 性能分析 |
| ------ | ------ |
| 同步阻塞 | 最常用的一种用法，使用也是最简单的，但是 I/O 性能一般很差，CPU 大部分在空闲状态 |
| 同步非阻塞 | 提升 I/O 性能的常用手段，就是将 I/O 的阻塞改成非阻塞方式，尤其在网络 I/O 是长连接，同时传输数据也不是很多的情况下，提升性能非常有效。 这种方式通常能提升 I/O 性能，但是会增加CPU 消耗，要考虑增加的 I/O 性能能不能补偿 CPU 的消耗，也就是系统的瓶颈是在 I/O 还是在 CPU 上 |
| 异步阻塞 | 这种方式在分布式数据库中经常用到，例如在网一个分布式数据库中写一条记录，通常会有一份是同步阻塞的记录，而还有两至三份是备份记录会写到其它机器上，这些备份记录通常都是采用异步阻塞的方式写 I/O。异步阻塞对网络 I/O 能够提升效率，尤其像上面这种同时写多份相同数据的情况 |
| 异步非阻塞 | 这种组合方式用起来比较复杂，只有在一些非常复杂的分布式情况下使用，像集群之间的消息同步机制一般用这种 I/O 组合方式。如 Cassandra 的 Gossip 通信机制就是采用异步非阻塞的方式。它适合同时要传多份相同的数据到集群中不同的机器，同时数据的传输量虽然不大，但是却非常频繁。这种网络 I/O 用这个方式性能能达到最高 |

## 1. BIO
<img src="D:\Project\IT-notes\Java\IO\img\BIO模型.png" style="width:700px;height:350px;" />

`BIOClient`
```java
package io_learn.BIO;

import java.io.IOException;
import java.net.Socket;
import java.util.Date;
import java.util.Scanner;

/**
 * @Auther: ARong
 * @Date: 2019/11/29 5:42 下午
 * @Description: BIO 的 客户端
 */
public class BIOClient {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("127.0.0.1", 6666);
        Scanner input = new Scanner(System.in);
        while (input.hasNext()) {
            socket.getOutputStream().write(input.nextLine().getBytes());
        }
    }
}
```

`BIOServer`
```java
package io_learn.BIO;

import java.io.IOException;
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;

/**
 * @Auther: ARong
 * @Date: 2019/11/29 5:38 下午
 * @Description: BIO 的 服务端
 */
public class BIOServer {
    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(6666);
        while (true) {
            // 阻塞直到有客户端连接
            Socket socket = serverSocket.accept();
            // 处理请求
            int len = -1;
            byte[] data = new byte[1024];//每次读取1k
            InputStream inputStream = socket.getInputStream();
            while ((len = inputStream.read(data)) != -1) {
                System.out.println(new String(data, 0, len));
            }
        }
    }
}
```
## 2. NIO
<img src="D:\Project\IT-notes\Java\IO\img\NIO模型.png" style="width:700px;height:250px;" />

<img src="D:\Project\IT-notes\Java\IO\img\buffer数据流程.png" style="width:700px;height:500px;" />

特点：
1. 每个`channel`对应一个`buffer`
2. `selector`对应一个线程，一个线程对应多个`channel`
3. 多个`channel`注册到一个`selector`
4. 程序切换到哪个`channel`由`event`决定
5. `selector`根据不同的事件在各个通道上切换
6. `buffer`为一个内存块，底层为一个数组
7. 数据读取写入都是通过`buffer`，即`buffer`为全双工，需要使用`flip()`切换读写模式
### 1. Channel
<img src="D:\Project\IT-notes\Java\IO\img\NIO模型2.png" style="width:700px;height:700px;" />

`Channel`为数据的抽象通道，无论是文件还是`Socket`，可以通过`Channel`进行读取与写入；而`Channel`与`Stream`的不同之处在于`BIO`中的`Stream`为单工，而`Channel`为全双工；`Channel`一端连接`Buffer`，一端连接实体（文件、`Socket`），而`Channel`无论是读写数据都是先经过`buffer`

`Channel`实现：
1. `FileChannel`：读写文件
2. `DatagramChannel`：读写`UDP`
3. `SocketChannel`：读写`TCP`客户端
4. `ServerSocketChannel`：读写`TCP`服务端

存在`Scatter/Gather`（分散/聚集）：即针对单个`Channel`使用多个`buffer`进行读写
```java
ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body = ByteBuffer.allocate(1024);
ByteBuffer[] bufferArray = [heander, body];
channel.read(bufferArray);

ByteBuffer header = ByteBuffer.allocate(128);
ByteBuffer body = ByteBuffer.allocate(1024);
ByteBuffer[] bufferArray = [heander, body];
channel.write(bufferArray);
```
### 2. Buffer
<img src="D:\Project\IT-notes\Java\IO\img\buffer类型.png" style="width:700px;height:300px;" />

```java
put(byte b) //将给定单个字节写入缓冲区的当前位置
put(byte[] src) //将 src 中的字节写入缓冲区的当前位置
put(int index, byte b) //将指定字节写入缓冲区的索引位置(不会移动 position)

get() //读取单个字节
get(byte[] dst) //批量读取多个字节到 dst 中
get(int index) //读取指定索引位置的字节(不会移动 position)
```

使用`Buffer`读写数据，一般遵循四个操作：
1. 写数据到`Buffer`
2. 调用`flip`方法切换读写模式，默认读模式
3. 从`Buffer`中读取数据
4. 调用`clear`或者`compact`清空`Buffer`，`clear`清空整个缓冲区，`compact`清除已经读写过的数据

`Buffer`中存在三个重要属性：`capacity`、`position`、`limit`
<img src="D:\Project\IT-notes\Java\IO\img\Buffer三个基本属性.png" style="width:700px;height:300px;" />

`Buffer`几个重要方法：
1. `alloacte`：分配若干大小缓冲区
2. `read`：从`channel`中读取数据到`buffer`
3. `write`：从`buffer`中写数据到`channel`
4. `flip`：将`buffer`从写模式（读模式）切换到读模式（写模式）；从写模式切换到读模式`position`会置0，从读模式切换到写模式`position`会置为`limit`
5. `rewind`：`position`置为0，重新读取`buffer`中的所有数据
6. `clear`：清空缓冲区，`buffer`中的数据并未清除，只是`position limit capacity`被重置了而已，即`position=0`、`limit=capacity`
7. `compact`：将第一个未读元素到`limit`处之间的所有元素拷贝到`buffer`起始处，然后`position=此时第一个未读元素位置`，`limit=capacity`
8. `mark&reset`：调用`mark`标记一个`position`，之后通过`reset`恢复到这个`position`
9. `slice`：创建子缓冲区
10. `asReadOnlyBuffer`：设置为只读缓冲区
11. `allocateDirect`：设置为直接缓冲区，即使用操作系统的`native`方法进行IO读写，避免将数据先从虚拟机堆栈中操作后再拷贝到直接内容中
12. `map`：内存映射文件，`FileChannel channel = FileChannel.open(Paths.get("src/c.txt");MappedByteBuffer mapBuffer = channel.map(FileChannel.MapMode.READ_WRITE, 0, length);`，直接调用系统底层的内存缓存，适合大文件的内存读写
### 3. Selector
<img src="D:\Project\IT-notes\Java\IO\img\channel轮询.png" style="width:700px;height:250px;" />

特点：
1. `FileChannel`不能被选择器复用，因为`FileChannel`并未继承抽象类`SelectableChannel`
2. 一个`Channel`可以被注册到多个`Selector`上，而在同一个`Selector`上一个`Channel`只能被注册一次
3. `Selector`会轮询每个`Channel`的就绪状态，满足就绪状态则可进行读写，就绪状态包括四种：
	- 可读：`SelectionKey.OP_READ`
	- 可写：`SelectionKey.OP_WRITE`
	- 连接：`SelectionKey.OP_CONNECT`
	- 接收：`SelectionKey.OP_ACCEPT`
4. `Selector`对注册的`Channel`进行`select()`轮询查询，选中后得出选择键`SelectionKey`，使用选择键对信道进行操作
### 5. 完整实例
**FileChannel**
```java
public class NIOInTest {
    public static void main(String[] args) throws Exception {
        File file = new File("C:/Users/25852/Desktop/HTTP请求.jmx");
        FileInputStream inputStream = new FileInputStream(file);
        FileChannel fileChannel = inputStream.getChannel();// 流变channel
        ByteBuffer byteBuffer = ByteBuffer.allocate((int)file.length());
        fileChannel.read(byteBuffer);//从channel中读数据给buffer
        byteBuffer.flip();//翻转 从写变成读模式
        while(byteBuffer.hasRemaining()){//buffer 中是否还有元素
             byte b = byteBuffer.get();
             System.out.print((char) b);
        }
        inputStream.close();
    }
}

public class NIOOutTest {
    public static void main(String[] args) throws Exception {
        File file = new File("test.txt");
        FileOutputStream outputStream = new FileOutputStream(file);
        FileChannel fileChannel = outputStream.getChannel();// 流变 channel
        byte[] message = "hello world ,It's me".getBytes();
        ByteBuffer byteBuffer = ByteBuffer.allocate(message.length);
        byteBuffer.put(message);// 数据都要装入buffer
        byteBuffer.flip();  //Buffer 默认是读模式，需要反转
        fileChannel.write(byteBuffer);// 将buffer写入channel
        System.out.println("输出成功");
        outputStream.close();
    }
}
```

**NIOServer**
```java
public class NioServer {
    private static Map<String, SocketChannel> clientMap = new HashMap<>();//记录客户端信息，方便内容分发

    public static void main(String[] args) throws Exception {
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        serverSocketChannel.configureBlocking(false);
        ServerSocket serverSocket = serverSocketChannel.socket();
        serverSocket.bind(new InetSocketAddress(8899));
        Selector selector = Selector.open();
        /*
        当accept触发时，就可以触发对应的事件逻辑，
        是将channel绑定到selector ，注册会有 SelectionKey生成
         */
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);//一般以连接事件为起源
        while (true){
            selector.select();//阻塞，等待事件发生
            Set<SelectionKey>  selectionKeys = selector.selectedKeys();//返回已发生的注册事件
            selectionKeys.forEach(key ->{//判断事件类型，进行相应操作
                final SocketChannel client;
                try {
                    if (key.isAcceptable()){//根据key获得channel
                        //之所以转换ServerSocketChannel，因为前面注册的就是这个类
                        ServerSocketChannel serverChannel = (ServerSocketChannel) key.channel();
                        client = serverChannel.accept();//新的channel 和客户端建立了通道
                        client.configureBlocking(false);//非阻塞
                        client.register(selector,SelectionKey.OP_READ);//将新的channel和selector，绑定
                        String clientKey = "【"+ UUID.randomUUID() +"】";//用UUID，标识客户端client
                        clientMap.put(clientKey,client);
                        //完成客户端注册
                    }else if (key.isReadable()){//是否有数据可读
                        client = (SocketChannel) key.channel();
                        ByteBuffer readBuffer =  ByteBuffer.allocate(1024);
                        int count = client.read(readBuffer);
                        if (count>0){
                            readBuffer.flip();
                            Charset charset = StandardCharsets.UTF_8;
                            String receiveMassage = String.valueOf(charset.decode(readBuffer).array());
                            System.out.println(client +": "+receiveMassage);//显示哪个client发消息
                            String senderKey = null;
                            for (Map.Entry<String, SocketChannel> entry : clientMap.entrySet()){
                                if (client == entry.getValue()){
                                    senderKey = entry.getKey();//确定哪个client发送的消息
                                    break;
                                }
                            }
                            for (Map.Entry<String, SocketChannel> entry : clientMap.entrySet()){
                                SocketChannel channel = entry.getValue();
                                ByteBuffer writeBuffer = ByteBuffer.allocate(1024);
                                writeBuffer.put((senderKey + ": " + receiveMassage).getBytes());//告诉所有client ，谁发了消息，发了什么
                                writeBuffer.flip();
                                channel.write(writeBuffer);
                            }
                        }
                    }
                    //selectionKeys.clear();//处理完事件一定要移除
                }catch (Exception e){
                    e.printStackTrace();
                }finally {
                    selectionKeys.clear();//处理完事件一定要移除
                }
            });
        }
    }
}
```

**NIOClient**
```java
public class NioClient {
    public static void main(String[] args) throws Exception {
        SocketChannel socketChannel = SocketChannel.open();
        socketChannel.configureBlocking(false);
        Selector selector = Selector.open();
        socketChannel.register(selector, SelectionKey.OP_CONNECT);
        socketChannel.connect(new InetSocketAddress("127.0.0.1",8899));
        while (true){
            selector.select();//阻塞 等待事件发生
            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            selectionKeys.forEach(key ->{
                try {
                    if (key.isConnectable()){
                        SocketChannel channel = (SocketChannel) key.channel();
                        if (channel.isConnectionPending()){//是否正在连接
                            channel.finishConnect(); //结束正在连接
                            ByteBuffer writeBuffer = ByteBuffer.allocate(1024);
                            writeBuffer.put((LocalDateTime.now() + " 连接成功").getBytes());
                            writeBuffer.flip();
                            channel.write(writeBuffer);//将buffer写入channel
                            ExecutorService service = Executors.newSingleThreadExecutor(Executors.defaultThreadFactory());
                            service.submit(()->{//线程，从键盘读入数据
                               try {
                                   while (true){
                                       writeBuffer.clear();//清空buffer
                                       InputStreamReader input = new InputStreamReader(System.in);
                                       BufferedReader bufferedReader = new BufferedReader(input);
                                       String senderMessage = bufferedReader.readLine();
                                       writeBuffer.put(senderMessage.getBytes());
                                       writeBuffer.flip();
                                       channel.write(writeBuffer);
                                   }
                               }catch (Exception e){
                                   e.printStackTrace();
                               }
                            });
                        }
                        channel.register(selector,SelectionKey.OP_READ);//注册事件
                    }else if (key.isReadable()){//channel 有信息的输入
                        SocketChannel channel = (SocketChannel) key.channel();//哪个channel 触发了 read
                        ByteBuffer readBuffer = ByteBuffer.allocate(1024);
                        int count = channel.read(readBuffer);//server发来的
                        if (count > 0){
                            String receiveMessage = new String(readBuffer.array(),0,count);
                            System.out.println(receiveMessage);
                        }
                    }
                }catch (Exception e){
                    e.printStackTrace();
                }finally {
                    selectionKeys.clear();//移除已经发生的事件
                }
            });
        }
    }
}

```
## 3. AIO
