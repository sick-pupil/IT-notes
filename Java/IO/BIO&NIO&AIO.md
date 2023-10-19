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

<img src="D:\Project\IT-notes\Java\IO\img\buffer类型.png" style="width:700px;height:300px;" />

特点：
1. 每个`channel`对应一个`buffer`
2. `selector`对应一个线程，一个线程对应多个`channel`
3. 多个`channel`注册到一个`selector`
4. 程序切换到哪个`channel`由`event`决定
5. `selector`根据不同的事件在各个通道上切换
6. `buffer`为一个内存块，底层为一个数组
7. 数据读取写入都是通过`buffer`，即`buffer`为全双工，需要使用`flip()`切换读写模式

<img src="D:\Project\IT-notes\Java\IO\img\NIO模型2.png" style="width:700px;height:700px;" />
### 1. 文件读写
```java
// 获取文件通道
FileChannel.open(Paths.get(fileName), StandardOpenOption.READ);
// 分配字节缓存
ByteBuffer buf = ByteBuffer.allocate(10);

//读取
while (channel.read(buf) != -1){ // 读取通道中的数据，并写入到 buf 中
    buf.flip(); // 缓存区切换到读模式
    while (buf.position() < buf.limit()){ // 读取 buf 中的数据
        text.append((char)buf.get());
    }
    buf.clear(); // 清空 buffer，缓存区切换到写模式
}

//写入
for (int i = 0; i < text.length(); i++) {
    buf.put((byte)text.charAt(i)); // 填充缓冲区，需要将 2 字节的 char 强转为 1 自己的 byte
    if (buf.position() == buf.limit() || i == text.length() - 1) { // 缓存区已满或者已经遍历到最后一个字符
        buf.flip(); // 将缓冲区由写模式置为读模式
        channel.write(buf); // 将缓冲区的数据写到通道
        buf.clear(); // 清空缓存区，将缓冲区置为写模式，下次才能使用
    }
}

//将数据刷出到物理磁盘
channel.force(false);
//关闭通道
channel.close();
```

### 2. 网络读写
```java
//打开通道，连接到服务端
SocketChannel channel = SocketChannel.open(); // 打开通道，此时还没有打开 TCP 连接
channel.connect(new InetSocketAddress("localhost", 9090)); // 连接到服务端

//分配缓冲区
ByteBuffer buf = ByteBuffer.allocate(10); // 分配一个 10 字节的缓冲区，不实用，容量太小

//配置是否为阻塞方式
channel.configureBlocking(false); // 配置通道为非阻塞模式

//交互
...

//关闭连接
channel.close();          // 关闭通道
```

```java
//打开通道，提供连接给客户端
ServerSocketChannel server = ServerSocketChannel.open(); // 打开通道
//绑定端口
server.bind(new InetSocketAddress(9090)); // 绑定端口
//等待客户端连接到来
SocketChannel client = server.accept(); // 阻塞，直到有连接过来

//交互
...

//关闭连接
client.close();
```
### 3. buffer
<img src="D:\Project\IT-notes\Java\IO\img\buffer数据流程.png" style="width:700px;height:500px;" />

```java
put(byte b) //将给定单个字节写入缓冲区的当前位置
put(byte[] src) //将 src 中的字节写入缓冲区的当前位置
put(int index, byte b) //将指定字节写入缓冲区的索引位置(不会移动 position)

get() //读取单个字节
get(byte[] dst) //批量读取多个字节到 dst 中
get(int index) //读取指定索引位置的字节(不会移动 position)
```
### 4. selector
<img src="D:\Project\IT-notes\Java\IO\img\channel轮询.png" style="width:700px;height:250px;" />
```java
//获取选择器
Selector selector = Selector.open(); // 获取一个选择器实例

//获取可选择信道
SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress("localhost", 9090)); // 打开 SocketChannel 并连接到本机 9090 端口
socketChannel.configureBlocking(false); // 配置通道为非阻塞模式

//注册通道到选择器
socketChannel.register(selector, SelectionKey.OP_READ | SelectionKey.OP_WRITE); // 将套接字通过到注册到选择器，关注 read 和 write 事件

//轮询select就绪事件
while (selector.select() > 0){ // 轮询，且返回时有就绪事件
	Set<SelectionKey> keys = selector.selectedKeys(); // 获取就绪事件集合
	for(SelectionKey key : keys){
		if(key.isWritable()){ // 可写事件
			if("Bye".equals( (line = scanner.nextLine()) )){
				socketChannel.shutdownOutput();
				socketChannel.close();
				break;
			}
			buf.put(line.getBytes());
			buf.flip();
			socketChannel.write(buf);
			buf.compact();
		}
	}
}
```

### 5. 完整实例
**NIOServer**
```java
public static void main(String[] args) throws  Exception{
        //创建ServerSocketChannel，-->> ServerSocket
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        InetSocketAddress inetSocketAddress = new InetSocketAddress(5555);
        serverSocketChannel.socket().bind(inetSocketAddress);
        serverSocketChannel.configureBlocking(false); //设置成非阻塞
 
        //开启selector,并注册accept事件
        Selector selector = Selector.open();
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
 
        while(true) {
            selector.select(2000);  //监听所有通道
            //遍历selectionKeys
            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            Iterator<SelectionKey> iterator = selectionKeys.iterator();
            while (iterator.hasNext()) {
                SelectionKey key = iterator.next();
                if(key.isAcceptable()) {  //处理连接事件
                    SocketChannel socketChannel = serverSocketChannel.accept();
                    socketChannel.configureBlocking(false);  //设置为非阻塞
                    System.out.println("client:" + socketChannel.getLocalAddress() + " is connect");
                    socketChannel.register(selector, SelectionKey.OP_READ); //注册客户端读取事件到selector
                } else if (key.isReadable()) {  //处理读取事件
                    ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
                    SocketChannel channel = (SocketChannel) key.channel();
                    channel.read(byteBuffer);
                    System.out.println("client:" + channel.getLocalAddress() + " send " + new String(byteBuffer.array()));
                }
                iterator.remove();  //事件处理完毕，要记得清除
            }
        }
 
    }
```

**NIOClient**
```java
public class NIOClient {
	public static void main(String[] args) throws Exception{
	        SocketChannel socketChannel = SocketChannel.open();
	        socketChannel.configureBlocking(false);
	        InetSocketAddress inetSocketAddress = new InetSocketAddress("127.0.0.1", 5555);
	 
	        if(!socketChannel.connect(inetSocketAddress)) {
	            while (!socketChannel.finishConnect()) {
	                System.out.println("客户端正在连接中，请耐心等待");
	            }
	        }
	 
	        ByteBuffer byteBuffer = ByteBuffer.wrap("mikechen的互联网架构".getBytes());
	        socketChannel.write(byteBuffer);
	        socketChannel.close();
	}
}
```
## 3. AIO
