## 1. Zookeeper相关概念
Zookeeper基于观察者模式设计，负责存储和管理分布式数据，接收观察者的注册，负责将分布式数据变化通知给在Zookeeper上注册的观察者，**Zookeeper=文件系统+通知机制**

<img src="D:\Project\IT-notes\框架or中间件\Zookeeper\img\Zookeeper集群结构.png" style="width:700px;height:200px;" />

Zookeeper特点：
1. Zookeeper服务中，存在一个领导者leader、多个跟随者follower
2. 集群中只要有半数以上节点存活，则Zookeeper集群能正常服务，所以Zookeeper适合安装奇数台服务器
3. 全局数据一致，Zookeeper集群中的Server都保存了一份相同的数据副本，Client连接到哪个Server数据都是一致的
4. 来自同一个Client的更新请求按照发送顺序依次执行
5. 数据更新为原子性
6. 一定时间范围中，Client能读到最新数据

Zookeeper存在多个server，每个server都可以被多个client连接，server中存在命名空间，命名空间中的节点结构可以看作一棵树，每个节点为一个ZNode，每一个ZNode默认能存储1MB数据，每个ZNode拥有一个唯一路径标识
<img src="D:\Project\IT-notes\框架or中间件\Zookeeper\img\Zookeeper数据结构.png" style="width:500px;height:300px;" />

应用场景：
1. 统一命名服务，对应用服务进行统一命名，类似域名-IP映射关系
2. 统一配置管理，一般集群中的所有节点配置信息都存在一致的部分，针对这部分进行管理，修改后也能快速同步到各个节点；可将配置信息写到一个ZNode，每个客户端监听这个ZNode
3. 统一集群管理，可将服务器节点状态写入一个ZNode，每个客户端监听该ZNode获取集群状态
4. 服务器节点动态上下线，与统一集群管理功能重合
5. 软负载均衡，与统一命名服务功能重合

Zookeeper中存在三种角色：
- Leader：在集群中只有一个Leader节点，该节点是集群的中心，负责协调集群中的其他节点；Leader节点可以选择不接受客户端的连接
	- 作用：
		1. 发起与提交请求：所有的跟随者Follower与观察者Observer节点的写请求都会转交给领导者Leader执行。Leader接受到一个写请求后，首先会发送给所有的Follower，统计Follower写入成功的数量。当有超过半数的Follower写入成功后，Leader就会认为这个写请求提交成功，通知所有的Follower commit这个写操作，保证事后哪怕是集群崩溃恢复或者重启，这个写操作也不会丢失
		2. 与Learner（Follower、Observer统称）
		3. 崩溃恢复时负责恢复数据以及同步数据到Learner

- Follower：在集群中可以有多个Follower
	- 作用：
		1. 与Leader保持心跳连接
		2. 当Leader挂了，需要经过投票选举成为新Leader，Leader重新选举由Follower们投票决定
		3. 向Leader发送消息与请求
		4. 处理Leader发来的消息与请求
- Observer：为了在Zookeeper集群中Follower越来越多导致写性能越来越差的情况下提高写性能而存在
	- 负责：
		1. 与Leader同步数据
		2. 不参与Leader选举，也没有投票权，也不参与写操作的提议过程
		3. Observer只会把数据加载到内存中


## 2. Zookeeper配置文件相关参数
```cfg
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/tmp/zookeeper
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

## Metrics Providers
#
# https://prometheus.io Metrics Exporter
#metricsProvider.className=org.apache.zookeeper.metrics.prometheus.PrometheusMetricsProvider
#metricsProvider.httpPort=7000
#metricsProvider.exportJvmInfo=true

#zookeeper默认开启后占用8080端口，需要改变默认端口号
admin.serverPort=8887
```
1. `tickTime`：通信心跳时间，Zookeeper服务器与客户端心跳时间，单位毫秒
2. `initLimit`：Leader与Follower初始通信时限，初始连接能容忍的最多心跳次数，单位为一个心跳时间
3. `syncLimit`：Leader与Follower同步通信时限，之间的通信时间若超过`syncLimit * tickTime`，则认为Follower死掉并删除Follower，单位为一个心跳时间
4. `dataDir`：保存Zookeeper数据的文件路径
5. `clientPort`：客户端连接端口

## 3. Zookeeper集群安装
### 1. 新建若干台虚拟机

### 2. 在虚拟机上安装并配置Zookeeper环境
```
tickTime=2000

initLimit=10

syncLimit=5

dataDir=/home/lao/zookeeper/data
dataLogDir=/home/lao/zookeeper/logs

clientPort=2181

#server.A=B:C:D
#A表示第几号服务器
#B为这个服务器ip地址
#C为该服务器Follower与集群中的Leader服务器交换信息的端口
#D万一Leader服务器挂了，需要一个端口重新进行选举得出Leader，这个端口是执行选举时服务器相互通信的端口
server.1=10.0.2.15:2888:3888
server.2=10.0.2.4:2888:3888
server.3=10.0.2.5:2888:3888
```
注：需要在zoo.cfg配置文件中配置dataDir路径与dataLogDir路径，并新建相应dataDir与dataLogDir文件夹；在dataDir路径文件夹中创建myid文件，文件中添加与server对应编号

### 3. 集群脚本
```shell
#!/bin/bash

case $1 in 
"start") {
	for i in centos1 centos2 centos3 
	do 
		ssh $i "/home/lao/zookeeper/apache-zookeeper-bin/bin/zkServer.sh start"
	done
};;
"stop"){
	for i in centos1 centos2 centos3 
	do 
		ssh $i "/home/lao/zookeeper/apache-zookeeper-bin/bin/zkServer.sh stop"
	done
};;
"status"){
	for i in centos1 centos2 centos3 
	do 
		ssh $i "/home/lao/zookeeper/apache-zookeeper-bin/bin/zkServer.sh status"
	done
};;
esac
```

## 4. 选举机制
Zookeeper节点中的四种状态：
1. Leading
2. Looking：选举中
3. Following
4. Observing：不参加投票与选举，接受Leader选举后的结果

若希望形成Zookeeper集群，则需要至少三台机器，否则Zookeeper运行中不会选举Leader服务器，也不会作为Leader服务器运行

### 1. Zookeeper集群第一次启动选举
1. 服务器1启动，发起一次选举。服务器1投自己一票。此时服务器1票数一票，不够半数以上（3票），选举无法完成，服务器1状态保持为LOOKING;
2. 服务器2启动，发起一次选举。服务器1和服务器2分别投自己一票并交换投票信息；此时服务器1发现服务器2的myid比自己目前投票推举的（服务器1）大，更改投票为推举服务器2。此时服务器1票数0票，服务器2票数2票，没有半数以上结果，选举无法完成，服务器1,2保持LOOKING；
3. 服务器3启动，发起一次选举。此时服务器1和2都会更改选票为服务器3。此次投票结果：服务器1为0票，服务器2为0票，服务器3为3票。此时服务器3的票数已经超过半数，服务器3当选Leader。服务器1,2更改状态为FOLLOWING,服务器3更改为LEADING；
4. 服务器4启动，发起一次选举。此时1,2,3已经不是LOOKING状态；不会更改选票信息。交换选票结果：服务器3为3票，服务器4为1票。此时服务器4服从多数，更改选票信息为服务器3，并更改状态为FOLLOWING;
5. 服务器5启动，同4一样当小弟

### 2. Zookeeper集群非第一次启动选举
#### 1. 前提概念
1. `SID`：服务器ID。用来唯一标识一台ZooKeeper集群中的机器，每台机器不能重复，和myid一致
2. `ZXID`：事务ID。ZXID是一个事务ID，用来标识一次服务器状态的变更，Zookeeper中的每个变化都会产生一个全局唯一的ZXID。在某一时刻，集群中的每台机器的ZXID值不一定完全一致，这和ZooKeeper服务器对于客户端“更新请求”’的处理逻辑有关
3. `Epoch`：每个Leader任期的代号。没有Leader时同一轮投票过程中的逻辑时钟值是相同的。每投完一次票这个数据就会增加

#### 2. Zookeeper非第一次选举
1. 集群中本来就已经存在一个Leader，会被告知当前服务器的Leader信息，对于该机器来说，仅仅需要和Leader机器建立连接，并进行状态同步即可
2. 集群中确实不存在Leader，根据每台服务器中（EPOCH，ZXID，SID）三维信息对比得出Leader：先比较EPOCH，谁大谁胜出；若EPOCH都相同，则比较ZXID谁大；最后比较myid

## 5. 节点
使用`./zkCli.sh -server ip:clientPort`，让客户端连上对应IP地址的server

- 使用客户端连上服务器后，使用`ls /`可以查看命名空间中`/`路径下所存在的节点
<img src="D:\Project\IT-notes\框架or中间件\ZooKeeper\img\客户端ls命令.png" style="width:700px;height:400px;" />

- `ls -s /`查看`/`路径下的节点信息
<img src="D:\Project\IT-notes\框架or中间件\ZooKeeper\img\客户端ls -s命令.png" style="width:700px;height:400px;" />

### 节点属性信息
| ZNode状态信息      | 解释 |
| ----------- | ----------- |
| cZxid      | create ZXID，即该数据节点被创建时的事务id       |
| ctime   | create time，即该节点的创建时间        |
| mZxid   | modified ZXID，即该节点最终一次更新时的事务id        |
| mtime   | modified time，即该节点最后一次的更新时间        |
| pZxid   | 该节点的子节点列表最后一次修改时的事务id，只有子节点列表变更才会更新pZxid，子节点内容变更不会更新        |
| cversion   | 子节点版本号，当前节点的子节点每次变化时值增加1        |
| dataVersion   | 数据节点内容版本号，节点创建时为0，每更新一次节点内容(不管内容有无变化)该版本号的值增加1        |
| aclVersion   | 节点的ACL版本号，表示该节点ACL信息变更次数        |
| ephemeralOwner   | 创建该临时节点的会话的sessionId；如果当前节点为持久节点，则ephemeralOwner=0        |
| dataLength   | 数据节点内容长度        |
| numChildren   | 当前节点的子节点个数        |

节点类型：
- 根据节点生命周期
	1. 持久：客户端和服务端断开连接后，创建的节点不删除
	2. 短暂：客户端和服务端断开连接后，创建的节点自己删除
- 根据单调递增的序号
	1. 不带序号
	2. 带序号：在命名末尾会加上诸如`001 002`等序号

## 6. 客户端命令
| 操作      | 说明 |
| ----------- | ----------- |
| get \[\-w\] \[\-s\] path      | -w为可选参数，表示是否注册监听。若添加该参数，则其他客户端修改节点数据后，当前客户端可收到数据变更通知|
| stat path \[watch\]   | 用于查看节点状态信息，与get命令相似。如stat \/        |
| set path data \[version\]   | 对指定znode添加内容。version表示可指定dataVersion        |
| Is path\[watch\]   | 获取当前路径下的子节点列表（只包含直接子节点）        |
| Is2 path \[watch\]   | Is的增强版，获取当前路径下的子节点列表（只包含直接子节点）、和当前节点下的状态信息        |
| delete path \[version\]   | 删除指定节点，当指定节点下有子节点时，该节点无法删除；deleteall删除路径下的所有节点        |
| rmr path   | 代替delete命令，可递归删除指定节点        |
| create \[\-s\] \[\-e\] path data acl   | 创建节点，\-s表示顺序节点，\-e表示临时节点，acl表示设置ac权限控制        |
| sync path   | 强制同步，由于请求在半数以上的服务器上生效就表示此请求生效，那么就会有一些服务器上的数据是旧的。该命令则会强制同步所有的更新操作        |
| setAcl path acl   | 设置节点Acl，格式为：scheme：id：permissions        |
| getAcl path   | 获取节点的AcI信息，如getAcl /node1        |
| listquota path   | 显示节点配额        |
| Setquota \-n\|\-b val path   | 设置节点个数以及数据长度的配额        |
| delquota \[\-n\|\-b\] path   | 删除配额，\-n为子节点个数，\-b为节点数据长度        |

## 7. 监听器
1. 首先有个Main方法  
2. 在main线程中，创建Zookeeper客户端；这时就会创建两个线程，一个负责网络连接通信（connect），一个负责监听（listener）  
3. 通过connect线程将注册监听的事件发送给Zookeeper
4. Zookeeper的注册监听器列表中，将注册的监听事件，添加到列表中
5. Zookeeper监听到数据或者路径变化，就会将这个消息发送给listener线程
6. listener线程内部调用process方法

### 1. get path \[watch\]监听节点数据变化
get -s path注册一次监听器，只会监听一次数据变化
<img src="D:\Project\IT-notes\框架or中间件\ZooKeeper\img\get -w监听.png" style="width:600px;height:100px;" />

### 2. ls path \[watch\]监听子节点增减变化
<img src="D:\Project\IT-notes\框架or中间件\ZooKeeper\img\ls -w监听.png" style="width:600px;height:100px;" />

## 8. 客户端API
```xml
<dependency>
	<groupId>org.apache.zookeeper</groupId>
	<artifactId>zookeeper</artifactId>
	<version>3.5.5</version>
</dependency>
```

```java
package com.zookeeper.example;
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooKeeper;
import java.io.IOException;
import java.util.concurrent.CountDownLatch;

// 连接ZooKeeper服务器
public class ZooKeeperConnection {

    // ZooKeeper实例
    private ZooKeeper zoo;
    private final CountDownLatch connectedSignal = new CountDownLatch(1);
    // 连接服务器
    public ZooKeeper connect(String host) throws IOException, InterruptedException {
        zoo = new ZooKeeper(host, 5000, new Watcher() {
            public void process(WatchedEvent event) {
                Event.KeeperState state = event.getState();
                // 连接成功
                if (state == Event.KeeperState.SyncConnected) {
                    System.out.println("与ZooKeeper服务器连接成功");
                    connectedSignal.countDown();
                }
                // 断开连接
                else if (state == Event.KeeperState.Disconnected) {
                    System.out.println("与ZooKeeper服务器断开连接");
                }
            }
        });
        connectedSignal.await();
        return zoo;
    }

    // 断开与服务器的连接
    public void close() throws InterruptedException {
        zoo.close();
    }
}
```

```java
package com.zookeeper.example;
import org.apache.zookeeper.*;
import java.io.IOException;

// 创建Znode节点
public class ZKCreate {
    // 创建ZooKeeper实例
    private static ZooKeeper zk;

    // 创建ZooKeeperConnection实例
    private static ZooKeeperConnection conn;

    // 同步方式创建Znode节点
    public static void createNodeSync(String path, byte[] data) throws KeeperException, InterruptedException {
        String nodePath = zk.create(path, data, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
        System.out.println("NodePath: " + nodePath);
    }

    // 异步方式创建Znode节点
    public static void createNodeAsync(String path, byte[] data) {
        zk.create(path, data, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT,
            new AsyncCallback.StringCallback() {
                public void processResult(int resultCode, String path, Object ctx, String name) {
                    System.out.println("ResultCode: " + resultCode);
                    System.out.println("Path: " + path);
                    System.out.println("Ctx: " + ctx);
                    System.out.println("Name: " + name);
                }
            },
            "create_node_async"
        );
    }

    public static void main(String[] args) throws IOException, InterruptedException, KeeperException {
        // 路径
        String path1 = "/demo/node1";
        String path2 = "/demo/node2";
        // 数据
        byte[] data1 = "zookeeper node1".getBytes();
        byte[] data2 = "zookeeper node2".getBytes();
        // 建立连接
        conn = new ZooKeeperConnection();
        zk = conn.connect("localhost");
        // 同步方式创建节点
        createNodeSync(path1, data1);
        // 异步方式创建节点
        createNodeAsync(path2, data2);
        // 断开连接
        conn.close();
    }
}
```

## 9. 写数据
写入数据的请求可以发送给Zookeeper集群中的Leader Server或者Follower Server，而这两种发送方式中处理写请求的方式是不一样的

- 当写入请求直接发送到Leader节点：
	1. Client向Leader发出写请求
	2. Leader将数据写入到本节点，并将数据发送到所有的Follower节点
	3. 等待Follower节点返回写入成功的确认信号
	4. 当Leader接收到一半以上节点（包含自己）返回写成功的信息之后，返回写入成功消息给client
	5. 完成剩余的写入请求

- 当写入请求直接发送到Follower节点：
	1. Client向Follower发出写请求
	2. Follower节点将请求转发给Leader
	3. Leader将数据写入到本节点，并将数据发送到所有的Follower节点
	4. 等待Follower节点返回确认信号
	5. 当Leader接收到一半以上节点(包含自己)返回写成功的信息之后，返回写入成功消息给原来的Follower
	6. 原来的Follower返回写入成功消息给Client

## 10. 服务器动态上下线

**使用DistributedServer新建若干个有序短暂节点，随后新建一个DistributedClient对server目录中的子节点进行监听；随后无论是再使用DistributedServer新建节点，还是通过杀掉DistributedServer进程删除节点，DistributedClient都会监听到变化**

```java
package com.dpb.dynamic;
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooDefs.Ids;
import org.apache.zookeeper.ZooKeeper;
/**
 * 服务器端代码
 * @author dengp
 *
 */
public class DistributedServer {
	private static final String connectString = "192.168.88.121:2181,192.168.88.122:2181,192.168.88.123:2181";
	private static final int sessionTimeout = 2000;
	private static final String parentNode = "/servers";
	private ZooKeeper zk = null;
	/**
	 * 创建到zk的客户端连接
	 * 
	 * @throws Exception
	 */
	public void getConnect() throws Exception {
		zk = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
			@Override
						public void process(WatchedEvent event) {
				// 收到事件通知后的回调函数（应该是我们自己的事件处理逻辑）
				System.out.println(event.getType() + "---" + event.getPath());
				try {
					zk.getChildren("/", true);
				}
				catch (Exception e) {
				}
			}
		}
		);
	}
	/**
	 * 向zk集群注册服务器信息
	 * 
	 * @param hostname
	 * @throws Exception
	 */
	public void registerServer(String hostname) throws Exception {
		String create = zk.create(parentNode + "/server", hostname.getBytes(), Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
		System.out.println(hostname + "is online.." + create);
	}
	/**
	 * 业务功能
	 * 
	 * @throws InterruptedException
	 */
	public void handleBussiness(String hostname) throws InterruptedException {
		System.out.println(hostname + "start working.....");
		Thread.sleep(long.MAX_VALUE);
	}
	public static void main(String[] args) throws Exception {
		// 获取zk连接
		DistributedServer server = new DistributedServer();
		server.getConnect();
		// 利用zk连接注册服务器信息
		server.registerServer(args[0]);
		// 启动业务功能
		server.handleBussiness(args[0]);
	}
}
```

```java
package com.dpb.dynamic;
import java.util.ArrayList;
import java.util.List;
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.ZooKeeper;
/**
 * 客户端：通过zookeeper获取服务器地址
 * @author 波波烤鸭
 *
 */
public class DistributedClient {
	private static final String connectString = "192.168.88.121:2181,192.168.88.122:2181,192.168.88.123:2181";
	private static final int sessionTimeout = 2000;
	private static final String parentNode = "/servers";
	// 注意:加volatile的意义何在？
	private volatile List<String> serverList;
	private ZooKeeper zk = null;
	/**
	 * 创建到zk的客户端连接
	 * 
	 * @throws Exception
	 */
	public void getConnect() throws Exception {
		zk = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
			@Override
						public void process(WatchedEvent event) {
				// 收到事件通知后的回调函数（应该是我们自己的事件处理逻辑）
				try {
					//重新更新服务器列表，并且注册了监听
					getServerList();
				}
				catch (Exception e) {
				}
			}
		}
		);
	}
	/**
	 * 获取服务器信息列表
	 * 
	 * @throws Exception
	 */
	public void getServerList() throws Exception {
		// 获取服务器子节点信息，并且对父节点进行监听
		List<String> children = zk.getChildren(parentNode, true);
		// 先创建一个局部的list来存服务器信息
		List<String> servers = new ArrayList<String>();
		for (String child : children) {
			// child只是子节点的节点名
			byte[] data = zk.getData(parentNode + "/" + child, false, null);
			servers.add(new String(data));
		}
		// 把servers赋值给成员变量serverList，已提供给各业务线程使用
		serverList = servers;
		//打印服务器列表
		System.out.println(serverList);
	}
	/**
	 * 业务功能
	 * 
	 * @throws InterruptedException
	 */
	public void handleBussiness() throws InterruptedException {
		System.out.println("client start working.....");
		Thread.sleep(long.MAX_VALUE);
	}
	public static void main(String[] args) throws Exception {
		// 获取zk连接
		DistributedClient client = new DistributedClient();
		client.getConnect();
		// 获取servers的子节点信息（并监听），从中获取服务器信息列表
		client.getServerList();
		// 业务线程启动
		client.handleBussiness();
	}
}
```

## 11. 分布式锁
```java
// zk实现分布式锁
public class DisLockTest {
    public static void main(String[] args) {


        // 使用10个线程模拟分布式环境
        for (int i=0; i<10; i++)
        {
            Thread t = new Thread(new DisLockRunnable()).start();  // 启动线程
        }
    }

    static class DisLockRunnable implements Runnable{

        @Override
        public void run() {
            // 每个线程具体的任务，每个线程就是抢锁
            DisClient disClient = new DisClient();
            // 获取锁
            disClient.getDisLock();

            // 模拟获取锁后的其它动作
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            // 释放锁
            disClient.deleteLock();
        }
    }
}
```

```java
// 线程执行任务就是抢锁
// 实现思路：
// 1. 去zk中创建临时序列节点, 并获取到序号
// 2. 判断自己创建节点序号是否是当前节点最小序号，如果是则获取锁, 执行相关操作，最后要释放锁
// 3. 不是最小节点，当前线程需要等待，等待前一个序号的节点被删除，然后再次判断自己是否是最小节点
public class DisClient {

    public DisClient() {
        // 初始化zk的/distrilLock节点, 会出现线程安全问题
        // 使用同步锁，保证了检查和创建的安全性
        synchronized (DisClient.class)
        {
            if (!zkClient.exists("/distrilock"))
            {
                zkClient.createPersistent("/distrilock");
            }
        }

    }

    // 定义前一个节点
    String beforeNodePath;

    // 定义当前节点
    String currentNodePath;

    // 计数器, 用来使没有获得锁的线程进行等待
    CountDownLatch countDownLatch=null;

    // 获取到ZkClient, 连接zk集群
    private ZkClient zkClient = new ZkClient("linux121:2181, linux122:2181");

    // 把抢锁过程分为两部分，一部分是创建节点，比较序号。另一部分是等待锁

    // 完整获取锁方法
    public void getDisLock()
    {
        // 获取当前线程名称
        String threadName = Thread.currentThread().getName();
        // 首先调用tryGetLock
        if (tryGetLock())
        {
            // 说明获取到锁了
                System.out.println(threadName + ": 获取到了锁");
        }
        else{
                System.out.println(threadName + ": 获取锁失败, 进入等待状态");
            }
            // 没有获取到锁，则需要进行等待，同时监听其上一个节点
            waitForLock();
            // 递归获取锁。即等待结束后，可以调用getDisLock方法重新尝试获取锁
            getDisLock();
        }
    }

    // 尝试获取锁
    public boolean tryGetLock()
    {
        // 在指定目录下创建临时顺序节点
        if (null == currentNodePath || "".equals(currentNodePath))
        {
            currentNodePath = zkClient.createEphemeralSequential("/distrilock/", "lock");
        }

        // 获取到路径下所有的子节点
        List<String> childs = zkClient.getChildren("/distrilock");
        // 对节点信息进行排序
        Collections.sort(childs); // 默认是升序
        String minNode = childs.get(0); // 获取序号最小的节点
        // 判断自己创建节点是否与最小序号一致
        if (currentNodePath.equals("/distrilock/" + minNode))
        {
            // 说明当前线程创建的就是序号最小节点
            return true;
        }
        else{
            // 说明最小节点不是自己创建的
            // 此时，要监控自己当前节点序号前一个的节点
            int i = Collections.binarySearch(childs, currentNodePath.substring("/distrilock/".length()));
            // 前一个(lastNodeChild是不包括父节点)
            String lastNodeChild = childs.get(i - 1);
            beforeNodePath = "/distrilock/" + lastNodeChild;
        }
        return false;
    }

    // 等待之前节点释放锁, 如何判断锁被释放，需要唤醒线程，继续尝试tryGetLock
    public void waitForLock() {
        // 准备一个监听器
        IZkDataListener iZkDataListener = new IZkDataListener() {
            @Override
            public void handleDataChange(String s, Object o) throws Exception {

            }
            @Override
            // 监听数据被删除
            public void handleDataDeleted(String s) throws Exception {
                // 提醒当前线程再次获取锁
                // 这里即当有前一个节点被删除时，唤醒下一个节点的线程
                countDownLatch.countDown(); // 把值减一，变为0，则唤醒之前await的线程
            }
        };

        // 监控前一个节点
        zkClient.subscribeDataChanges(beforeNodePath, iZkDataListener);

        // 在监听的通知没来之前，该线程应该是等待状态, 先判断一次上一个节点是否还存在
        if (zkClient.exists(beforeNodePath))
        {
            // 开始等待, CountDownLatch: 线程同步计数器
            countDownLatch = new CountDownLatch(1);
            try {
                countDownLatch.await(); // 阻塞，直到countDownLatch值变为0
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

        }

        // 此时监听结束，可以解除监听
        zkClient.subscribeDataChanges(beforeNodePath, iZkDataListener);

    }

    // 释放锁
    public void deleteLock()
    {
        if (zkClient!=null)
        {
            zkClient.delete(currentNodePath);
            zkClient.close();
            System.out.println(Thread.currentThread().getName() + ": 释放了锁资源");
        }
    }
}
```

## 12. curator
```xml
<dependency>
	<groupId>org.apache.curator</groupId>
	<artifactId>curator-framework</artifactId>
	<version>4.3.0</version>
</dependency>

<dependency>
	<groupId>org.apache.curator</groupId>
	<artifactId>curator-recipes</artifactId>
	<version>4.3.0</version>
</dependency>

<dependency>
	<groupId>org.apache.curator</groupId>
	<artifactId>curator-client</artifactId>
	<version>4.3.0</version>
</dependency>
```

```java
public static CuratorFramework getClient() {
	return CuratorFrameworkFactory.builder()
	            .connectString("127.0.0.1:2181")
	            .retryPolicy(new ExponentialBackoffRetry(1000, 3))
	            .connectionTimeoutMs(15 * 1000) //连接超时时间，默认15秒
	.sessionTimeoutMs(60 * 1000) //会话超时时间，默认60秒
	.namespace("arch") //设置命名空间
	.build();
}
public static void create(final CuratorFramework client, final String path, 
						  final byte[] payload) throws Exception {
	client.create().creatingParentsIfNeeded().forPath(path, payload);
}
public static void createEphemeral(final CuratorFramework client, final String path, 
								   final byte[] payload) throws Exception {
	client.create().withMode(CreateMode.EPHEMERAL).forPath(path, payload);
}
public static String createEphemeralSequential(final CuratorFramework client, 
				final String path, final byte[] payload) throws Exception {
	return client.create().
	withProtection().withMode(CreateMode.EPHEMERAL_SEQUENTIAL).
	forPath(path, payload);
}
public static void setData(final CuratorFramework client, final String path, 
						   final byte[] payload) throws Exception {
	client.setData().forPath(path, payload);
}
public static void delete(final CuratorFramework client, 
						  final String path) throws Exception {
	client.delete().deletingChildrenIfNeeded().forPath(path);
}
public static void guaranteedDelete(final CuratorFramework client, 
									final String path) throws Exception {
	client.delete().guaranteed().forPath(path);
}
public static String getData(final CuratorFramework client, 
							 final String path) throws Exception {
	return new String(client.getData().forPath(path));
}
public static List<String> getChildren(final CuratorFramework client, 
									   final String path) throws Exception {
	return client.getChildren().forPath(path);
}
```

```java
public static void nodeCache() throws Exception {
	final String path = "/nodeCache";
	final CuratorFramework client = getClient();
	client.start();
	delete(client, path);
	create(client, path, "cache".getBytes());
	final NodeCache nodeCache = new NodeCache(client, path);
	nodeCache.start(true);
	nodeCache.getListenable()
	            .addListener(() -> System.out.println("node data change, new data is " + new String(nodeCache.getCurrentData().getData())));
	setData(client, path, "cache1".getBytes());
	setData(client, path, "cache2".getBytes());
	Thread.sleep(1000);
	client.close();
}

public static void pathChildrenCache() throws Exception {
	final String path = "/pathChildrenCache";
	final CuratorFramework client = getClient();
	client.start();
	final PathChildrenCache cache = new PathChildrenCache(client, path, true);
	cache.start(PathChildrenCache.StartMode.POST_INITIALIZED_EVENT);
	cache.getListenable().addListener((client1, event) -> {
		switch (event.getType()) {
			case CHILD_ADDED:
			    System.out.println("CHILD_ADDED:" + event.getData().getPath());
			break;
			case CHILD_REMOVED:
			    System.out.println("CHILD_REMOVED:" + event.getData().getPath());
			break;
			case CHILD_UPDATED:
			    System.out.println("CHILD_UPDATED:" + event.getData().getPath());
			break;
			case CONNECTION_LOST:
			    System.out.println("CONNECTION_LOST:" + event.getData().getPath());
			break;
			case CONNECTION_RECONNECTED:
			    System.out.println("CONNECTION_RECONNECTED:" + event.getData().getPath());
			break;
			case CONNECTION_SUSPENDED:
			    System.out.println("CONNECTION_SUSPENDED:" + event.getData().getPath());
			break;
			case INITIALIZED:
			    System.out.println("INITIALIZED:" + event.getData().getPath());
			break;
			default:
			    break;
		}
	}
	);
	//        client.create().withMode(CreateMode.PERSISTENT).forPath(path);
	Thread.sleep(1000);
	client.create().withMode(CreateMode.PERSISTENT).forPath(path + "/c1");
	Thread.sleep(1000);
	client.delete().forPath(path + "/c1");
	Thread.sleep(1000);
	client.delete().forPath(path);
	//监听节点本身的变化不会通知
	Thread.sleep(1000);
	client.close();
}

public static void treeCache() throws Exception {
	final String path = "/treeChildrenCache";
	final CuratorFramework client = getClient();
	client.start();
	final TreeCache cache = new TreeCache(client, path);
	cache.start();
	cache.getListenable().addListener((client1, event) -> {
		switch (event.getType()){
			case NODE_ADDED:
			    System.out.println("NODE_ADDED:" + event.getData().getPath());
			break;
			case NODE_REMOVED:
			    System.out.println("NODE_REMOVED:" + event.getData().getPath());
			break;
			case NODE_UPDATED:
			    System.out.println("NODE_UPDATED:" + event.getData().getPath());
			break;
			case CONNECTION_LOST:
			    System.out.println("CONNECTION_LOST:" + event.getData().getPath());
			break;
			case CONNECTION_RECONNECTED:
			    System.out.println("CONNECTION_RECONNECTED:" + event.getData().getPath());
			break;
			case CONNECTION_SUSPENDED:
			    System.out.println("CONNECTION_SUSPENDED:" + event.getData().getPath());
			break;
			case INITIALIZED:
			    System.out.println("INITIALIZED:" + event.getData().getPath());
			break;
			default:
			                break;
		}
	}
	);
	client.create().withMode(CreateMode.PERSISTENT).forPath(path);
	Thread.sleep(1000);
	client.create().withMode(CreateMode.PERSISTENT).forPath(path + "/c1");
	Thread.sleep(1000);
	setData(client, path, "test".getBytes());
	Thread.sleep(1000);
	client.delete().forPath(path + "/c1");
	Thread.sleep(1000);
	client.delete().forPath(path);
	Thread.sleep(1000);
	client.close();
}
```

可重入锁
`public InterProcessMutex(CuratorFramework client, String path)`

不可重入锁
`InterProcessSemaphoreMutex`

可重入读写锁
`InterProcessReadWriteLock InterProcessLock`


## 13. 相关算法理论
Zookeeper使用多种算法协议保证实现分布式中的一致性

### 1. Paxos算法
为快速正确地在一个分布式系统中对某个数据值达成一致，并且保证不论发生任何异常，都不会破坏整个系统的一致性

在Paxos系统中，所有节点会划分为三类：
1. Proposer：提议者，决策会议发起者，发起决策事件
2. Acceptor：接受者，对决策进行投票
3. Learner：学习者，执行决策

**Paxos流程**：
- **Prepare准备阶段**：Proposer生成全局唯一且递增的Proposal ID，向所有Acceptor发送Prepare请求，这里无需携带提案内容，只携带Proposal ID即可
	1. Proposer向多个Acceptor发起Prepare请求
	2. Acceptor针对收到的Prepare请求进行Promise，即“两个承诺，一个应答”：
		- 不再接受Proposal ID小于等于当前请求的Prepare请求
		- 不再接受Proposal ID小于当前请求的Propose请求
		- 不违背以前做出的承诺，再添加回复已经Accept过的提案中Proposal ID最大的提案中的Value和Proposal ID，没有则返回空值
- **Propose提案阶段**：Proposer收到多数Acceptors的Promise应答后，从应答中选择Proposal ID最大的提案Value，作为本次要发起的提案。如果所有应答的提案Value均为空值，则随意决定提案Value，然后携带当前Proposal ID，向所有Acceptors发送Propose请求
- **Accept接收阶段**：Acceptor收到Propose请求后，在不违背自己之前作出的承诺下，接受并持久化当前Proposal ID和提案Value
- **Learn**：Proposer收到多数Acceptors的Accept后，决议形成并发送给所有Learners

<img src="D:\Project\IT-notes\框架or中间件\ZooKeeper\img\Paxos流程.png" style="width:700px;height:400px;" />

当Proposer存在多个时，可能会因为Proposal ID（比如Proposal ID=timestamp.server）导致相互争夺Acceptor，导致提案无法达成一致。因此提出一次Paxos流程中只有一个Proposer的算法：**ZAB**

### 2. ZAB算法
ZAB是一种专门为Zookeeper设计的一种支持**崩溃恢复**的**原子广播协议**，是Zookeeper保证数据一致性的核心算法

基于ZAB协议，Zookeeper保持了集群中各副本之间的数据的⼀致性，表现形式就是**使⽤⼀个单⼀的主进程（Leader服务器）来接收并处理客户端的所有事务请求（写请求），并采⽤ZAB的原⼦⼴播协议，将服务器数据的状态变更为事务 Proposal的形式⼴播到所有的Follower进程中**

所有事务必须由一个**全局唯一的服务器来协调处理** ，这样的服务器被称为Leader服务器，余下的服务器则称为Follower服务器
1. Leader服务器负责将一个客户端事务请求转化为一个事务Proposal提案，并将该Proposal分发给集群中所有的Follower服务器
2. Leader服务器等待所有Follower服务器的反馈，一旦超过半数的Follower服务器进行了正确的反馈后，Leader就会向所有的Follower服务器发送Commit消息，要求将前一个Proposal进行提交

#### 1. ZAB两种模式之一：消息广播
**消息广播过程**：
1. 客户端发起一个写操作请求
2. Leader服务器将客户端的请求转化为事务Proposal提案，同时为每个Proposal分配一个全局ID，ZXID
3. Leader服务器为每个Follower服务器分配一个单独的队列，然后将需要广播的Proposal依次放到队列中去，并且根据FIFO策略进行消息发送
4. Follower接收到Proposal后，会首先将其以事务日志的形式写入本地磁盘，写入成功后向Leader反馈一个ACK响应
5. Leader接受到超过半数以上Follower的ACK响应后，即认为消息发送成功，可以发送commit消息
6. Leader向所有Follower广播commit消息，同时自身也会完成事务提交。Follower接受到commit消息后，会将上一条事务提交

<img src="D:\Project\IT-notes\框架or中间件\ZooKeeper\img\消息广播过程.jpg" style="width:700px;height:800px;" />

#### 2. ZAB两种模式之一：崩溃恢复
当Leader服务器出现崩溃或由于网络原因导致Leader服务器失去了与过半Follower的联系，就会进入**崩溃恢复模式**：
1. 需要选举出新的Leader
2. 需要过半Follower感知到这个新Leader的存在并与该Leader完成状态同步

如果发生故障：
1. 一个事务在Leader服务器上被提交，并且已经得到过半Follower服务器的Ack反馈，但是在它将commit消息发送给所有Follower之前，Leader服务器挂了
2. Leader服务器提出事务之后崩溃导致其他Follower服务器没有接收到这个事务
必须确保：
1. 提交成功的事务最终被所有服务器执行完毕
2. 未成功提交的则丢弃事务

ZAB实现数据同步：
1. 完成Leader选举（新的Leader具有最高的zxid）之后，在正式开始⼯作（接收客户端请求）之前，Leader服务器会⾸先确认事务⽇志中的所有Proposal是否都已经被集群中过半的机器提交了，即**是否完成数据同步**
2. Leader服务器需要确保所有的Follower服务器能够接收到每⼀条事务Proposal，并且能够正确地将所有已经提交了的事务Proposal应⽤到内存数据中。等到Follower服务器将所有其尚未同步的事务Proposal都从Leader服务器上同步过来并成功应⽤到本地数据库中后，Leader服务器就会将该Follower服务器加⼊到真正的可⽤Follower列表中，并开始之后的其他流程

一个Follower只能和一个Leader保持同步，Leader进程和所有的Follower进程之间通过心跳监测机制来感知彼此的情况
1. 若Leader能够在超时时间内正常的收到心跳检测，那么Follower就会一直与该Leader保持连接
2. 如果在指定时间内Leader无法从过半的Follower进程那里接收到心跳检测，或者TCP连接断开，那么Leader会放弃当前周期的领导，并转换为LOOKING状态；其他的Follower也会选择放弃这个Leader，同时转换为LOOKING状态，之后会进行新一轮的Leader选举

### 3. CAP理论
- 一致性C：多个副本之间是否能够保持数据一致
- 可用性A：系统提供的服务必须一致处于可用的状态
- 分区容错性P：在遇到任何网络分区故障时，仍需要能够保证对外提供满足一致性和可用性的服务

**Zookeeper保证的是CP**，比如：
1. 每次服务请求不能保证，可能会因为极端环境，Zookeeper丢弃一些请求
2. 进行Leader选举时集群是不可用的