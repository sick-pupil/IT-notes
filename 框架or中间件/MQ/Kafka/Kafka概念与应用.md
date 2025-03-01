<img src="D:\Project\IT-notes\框架or中间件\MQ\Kafka\img\Kafka3大纲.png" style="width:700px;height:300px;" />

## 1. 概述
`kafka`是一个分布式基于发布订阅模式的消息队列，本质为一个`MQ`，应用场景有：多服务解耦、异步通信、缓冲
<img src="D:\Project\IT-notes\框架or中间件\MQ\Kafka\img\发布订阅模式.png" style="width:700px;height:200px;" />
**发布订阅模式**：生产者将消息发布到主题`topic`中，多个消费者订阅该主题，被消费的数据不会从主题中立即被清除

<img src="D:\Project\IT-notes\框架or中间件\MQ\Kafka\img\基本架构.png" style="width:700px;height:400px;" />

- `kafka`集群存在多个`kafka`节点，一个节点就是一个`broker`，每个节点存在多个分区`partition`。当存在一个比较大数据量的`topic`时，可以将大`topic`分割成多个`partition`分区进行分布式存储
- 而从`topic`分割出来的多个`partition`还会存在`leader`与`follower`的**数据一致性**关系，即`zookeeper`中的`leader`与`follower`关系：`leader`存在多个`follower`从节点作为读节点备份，当`leader`节点宕机可以在`follower`中进行投票选举

## 2. 安装
### 1. 原生部署
- 安装`zookeeper`
- 下载`kafka.tar`安装包
- 修改`server.properties`，每个节点都设置
```prop
broker.id=1
log.dirs=/Volumns/doc/tmp/kafka-log
zookeeper.connect=zk1_host:2181,zk2_host:2181,zk3_host:2181
```
- 启动`zookeeper`：`zookeeper-server-start.sh -daemon config/zookeeper.properties`
- 启动`kafka`：`kafka-server-start.sh -daemon config/server.properties`
- 显示已有`topic`：`kafka-topics.sh --list --zookeeper localhost:2181`
- 生产消息：`kafka-console-producer.sh --broker-list localhost:9092 --topic javaedge_ad_test_x`
- 消费消息：`kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic javaedge_ad_test_x --from-beginning`

### 2. docker部署
#### 1. `docker-zookeeper-compose.yml`
```yaml
version: '2.17.3'

networks:
	zk-net:
		name: zk-net

services:
	zk1:
	    image: zookeeper:3.6
	    hostname: zk1
	    container_name: zk1
	    ports:
			- 2181:2181
			- 8080:8080
		environment:
			ZOO_MY_ID: 1
			ZOO_SERVERS: server.1=0.0.0.0:2888:3888;2181 server.2=zk2:2888:3888;2181 server.3=zk3:2888:3888;2181
	volumes:
		- D:\Project\docker\zk1-data:/data
		- D:\Project\docker\zk1-datalog:/datalog
    networks:
		- zk-net

	zk2:
	    image: zookeeper:3.6
	    hostname: zk2
	    container_name: zk2
	    ports:
			- 2182:2181
			- 8081:8080
		environment:
			ZOO_MY_ID: 2
			ZOO_SERVERS: server.1=zk1:2888:3888;2181 server.2=0.0.0.0:2888:3888;2181 server.3=zk3:2888:3888;2181
		volumes:
			- D:\Project\docker\zk2-data:/data
			- D:\Project\docker\zk2-datalog:/datalog
		networks:
			- zk-net

	zk3:
	    image: zookeeper:3.6
	    hostname: zk3
	    container_name: zk3
	    ports:
			- 2183:2181
			- 8082:8080
		environment:
			ZOO_MY_ID: 3
			ZOO_SERVERS: server.1=zk1:2888:3888;2181 server.2=zk2:2888:3888;2181 server.3=0.0.0.0:2888:3888;2181
		volumes:
			- D:\Project\docker\zk3-data:/data
			- D:\Project\docker\zk3-datalog:/datalog
		networks:
			- zk-net
```

#### 2. `docker-kafka-compose.yml`
```yaml
version: '2.17.3'

networks:
	zk-net:
		external: true

services:

	kafka1:
	    image: wurstmeister/kafka:2.12-2.5.1
	    restart: unless-stopped
	    container_name: kafka1
	    ports:
			- "9092:9092"
		external_links:
			- zk1
			- zk2
			- zk3
		environment:
			KAFKA_BROKER_ID: 1
			KAFKA_ADVERTISED_HOST_NAME: 192.168.1.104                   ## 修改:宿主机IP
			KAFKA_ADVERTISED_PORT: 9092                                 ## 修改:宿主机映射port
			KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://192.168.1.104:9092    ## 绑定发布订阅的端口。修改:宿主机IP
			KAFKA_ZOOKEEPER_CONNECT: "zk1:2181,zk2:2181,zk3:2181"
		volumes:
			- D:\Project\docker\kafka1\docker.sock:/var/run/docker.sock
			- D:\Project\docker\kafka1\data:/kafka
		networks:
			- zk-net

	kafka2:
	    image: wurstmeister/kafka:2.12-2.5.1
	    restart: unless-stopped
	    container_name: kafka2
	    ports:
			- "9093:9092"
		external_links:
			- zk1
			- zk2
			- zk3
		environment:
			KAFKA_BROKER_ID: 2
			KAFKA_ADVERTISED_HOST_NAME: 192.168.1.104                 ## 修改:宿主机IP
			KAFKA_ADVERTISED_PORT: 9093                               ## 修改:宿主机映射port
			KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://192.168.1.104:9093   ## 修改:宿主机IP
			KAFKA_ZOOKEEPER_CONNECT: "zk1:2181,zk2:2181,zk3:2181"
		volumes:
			- D:\Project\docker\kafka2\docker.sock:/var/run/docker.sock
			- D:\Project\docker\kafka2\data:/kafka
		networks:
			- zk-net

	kafka3:
			image: wurstmeister/kafka:2.12-2.5.1
			restart: unless-stopped
			container_name: kafka3
		ports:
			- "9094:9092"
		external_links:
			- zk1
			- zk2
			- zk3
		environment:
			KAFKA_BROKER_ID: 3
			KAFKA_ADVERTISED_HOST_NAME: 192.168.1.104                 ## 修改:宿主机IP
			KAFKA_ADVERTISED_PORT: 9094                              ## 修改:宿主机映射port
			KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://192.168.1.104:9094   ## 修改:宿主机IP
			KAFKA_ZOOKEEPER_CONNECT: "zk1:2181,zk2:2181,zk3:2181"
		volumes:
			- D:\Project\docker\kafka3\docker.sock:/var/run/docker.sock
			- D:\Project\docker\kafka3\data:/kafka
		networks:
			- zk-net

	kafka-manager:
	    image: sheepkiller/kafka-manager:stable
	    restart: unless-stopped
	    container_name: kafka-manager
	    hostname: kafka-manager
	    ports:
			- "9000:9000"
		links:            # 连接本compose文件创建的container
			- kafka1
			- kafka2
			- kafka3
		external_links:   # 连接本compose文件以外的container
			- zk1
			- zk2
			- zk3
		environment:
			ZK_HOSTS: zk1:2181,zk2:2181,zk3:2181                 ## 修改:宿主机IP
			TZ: CST-8
		networks:
			- zk-net
```

## 3. 命令行操作
- 生产者：`kafka-console-producer.sh`
- 主题：`kafka-topics.sh`
- 消费者：`kafka-console-consumer.sh`

| `kafka-topics.sh`主题相关参数                           | 描述                      |
| ------------------------------------------------- | ----------------------- |
| --bootstrap-server <String: server toconnect to>  | 连接的Kafka Broker主机名称和端口号 |
| --topic <String: topic>                           | 操作的topic名称              |
| --create                                          | 创建主题                    |
| --delete                                          | 删除主题                    |
| --alter                                           | 修改主题                    |
| --list                                            | 查看所有主题                  |
| --describe                                        | 查看主题详细描述                |
| --partitions <Integer: # of partitions>           | 设置分区数                   |
| --replication-factor<Integer: replication factor> | 设置分区副本                  |
| --config <String: name=value>                     | 更新系统默认的配置               |

## 4. 存储机制
`kafka`消息队列支持大数据量的写入写出，因此即使是基于`scale`和`java`实现的，也不会在`JVM`上进行读写（需要开辟大量的堆空间、内存空间），因此使用磁盘存储数据
<img src="D:\Project\IT-notes\框架or中间件\MQ\Kafka\img\存储机制.png" style="width:700px;height:350px;" />

采用**分片**和**索引**机制，将每个`partition`分为多个`segment`，每个`segment`又分为两个文件：一个`.log`数据文件，一个`.index`索引文件

`partition`文件夹以**topic名称+分区序号**命名；而`partition`文件夹下存在多对`.log .index`文件对，一个文件对代表一个`segment`，文件对的命名方式以上一个`segment`的最后那一条消息的偏移量+1命名
```
kafka-topic
	kafka-topic-1
		0000000000000000000.log
		0000000000000000000.index
		0000000000000002584.log
		0000000000000002584.index
		0000000000000006857.log
		0000000000000006857.index
	kafka-topic-2
		...
	kafka-topic-3
		...
```

<img src="D:\Project\IT-notes\框架or中间件\MQ\Kafka\img\segment中的log和index.png" style="width:700px;height:250px;" />

## 5. 清理策略

`kafka`中默认的日志保存时间为7天，可以通过调整如下参数修改保存时间：
- `log.retention.hours`：最低优先级小时，默认7天
- `log.retention.minutes`：分钟
- `log.retention.ms`：最高优先级毫秒
- `log.retention.check.interval.ms`：负责设置检查周期，默认5分钟

日志超时后，`kafka`提供两种清理数据策略：`delete删除`和`compact压缩`：
1. `delete`过期日志数据删除
	- `log.cleanup.policy = delete`所有数据启用删除策略
		- 基于时间：默认打开，以`segment`中所有记录中的最大时间戳作为该文件时间戳，`segment`文件时间戳超时才删除
		- 基于大小：默认关闭，超过设置的所有日志总大小，删除最早的`segment`，`log.retention.bytes`默认-1即无穷大
2. `compact`日志压缩：对于相同`key`不同`value`，只保留最后一个版本
	- `log.cleanup.policy = compact`所有数据启用压缩策略
		- 压缩后的`offset`可能是不连续的，从这些压缩后的`offset`消费消息时，将会拿到比这个`offset`大的`offset`对应的消息，实际上会拿到`offset`为7的消息，并从这个位置开始消费

## 6. 页缓存PageCache与零拷贝

**`kafka`高效读写的原因**：
- `kafka`采用零拷贝技术，减少数据拷贝和上下文环境切换
- 使用多个分区存储一个`topic`，提高吞吐量
- 磁盘顺序读写，磁盘中的文件顺序读写增加读写速度
- 批量删除和复制，数据被消费后并非马上被删除，而是到达一定量后批量删除
- 使用页缓存，避免使用`jvm`，不需要垃圾回收

**传统拷贝**：
1. 发起读操作请求，CPU收到请求后给DMA发起调度命令，由DMA将磁盘数据写入内存缓冲区（第一次拷贝）取完成后给CPU发送读取完成消息
2. CPU再将内存缓冲区数据写入到用户缓冲区（第二次拷贝）
3. 将用户态数据写入Socket缓存区（第三次拷贝）
4. 完成后，CPU调度DMA，让DMA将Socket缓存区数据写入网卡缓存区（第四次拷贝），发送数据

<img src="D:\Project\IT-notes\框架or中间件\MQ\Kafka\img\传统拷贝.png" style="width:700px;height:400px;" />

**零拷贝**：
1. 发起sedfile()请求，首先会在PageCache查找数据，若存在则直接开始滴2步，不存在则使用DMA将数据从磁盘上拷贝至PageCache缓存区
2. 读取完成后，DMA给CPU发送信号，CPU将内存地址和页内偏移量传输给Socket
3. DMA将PageCache中的缓存数据写入网卡设备中
4. DMA发送写完信号给Socket，返回Seedfile()调用结束

<img src="D:\Project\IT-notes\框架or中间件\MQ\Kafka\img\零拷贝.png" style="width:700px;height:400px;" />

## 7. 生产者
<img src="D:\Project\IT-notes\框架or中间件\MQ\Kafka\img\生产者发送消息流程.png" style="width:700px;height:700px;" />

<img src="D:\Project\IT-notes\框架or中间件\MQ\Kafka\img\Kafka生产者发送消息流程.png" style="width:700px;height:350px;" />

<img src="D:\Project\IT-notes\框架or中间件\MQ\Kafka\img\kafka生产者原理.png" style="width:700px;height:400px;" />

1. 根据原始消息组装`ProducerRecord`进行发送
2. 经过`Serializer`进行序列化
3. 经过`Partitioner`分发到对应的`partition`
	- 如果已经指定了`partition`则分发到指定的分区中
	- 如果没有指定`partition`但存在消息`key`则根据`hash(key) % partitions.size`得到`partition`值
	- 也可以生成一个随机数`random`，后根据`random % partitions.size`得到`partition`值
4. 根据确定的`topic`与`partition`放置消息，存储在`partition`中的`batch`缓冲队列区域，等待积攒至区域空间阈值或者到达时间阈值，就会让额外的线程批量发送至`broker`中
5. 发送到`broker`前会由`sender`创建请求`request`并将`request`置于请求队列中
6. `broker`接受成功后返回消息记录的头信息`RecordMetadata`
<img src="D:\Project\IT-notes\框架or中间件\MQ\Kafka\img\发送ack机制.png" style="width:700px;height:300px;" />
- 成功发送：生产消息给`leader partition`，成功返回`ack`，并同步消息给`follower`副本，最后继续执行下一个消息的`send`
- 失败发送：生产消息并发送后并没有返回`ack`，则寻找`follower`副本发送，并让副本同步给`leader partition`

- `ISR`：`leader`会维护一个`ISR in-sync Replicas`，内部是所有和`leader`进行同步的`follower`，当`ISR`的所有`follower`完成同步后，`leader`会发送`ack`给`producer`。如果`follower`长时间未向`leader`同步数据，则会将该`follower`踢出`ISR`。如果`leader`宕机了，则会从`ISR`重新选举`leader`
- `OSR`：不能和`leader`保持同步的集合

应答机制：
```yaml
kafka:
    bootstrap-servers: 192.168.184.134:9092,192.168.184.135:9092,192.168.184.136:9092
    producer:
      # 值的序列化方式
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
      # acks=0 ： 生产者在成功写入消息之前不会等待任何来自服务器的响应。
      # acks=1 ： 只要集群的leader节点收到消息，生产者就会收到一个来自服务器成功响应。
      # acks=-1 or all ：只有当所有参与复制的节点全部收到消息时，生产者才会收到一个来自服务器的成功响应。
      acks: all
```

### 1. 异步发送
```java
	//配置
	Properties prop = new Properties();
	//连接地址
	prop.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka1:9092,kafka2:9092,kafka3:9092");
	//key value序列化
	prop.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
	prop.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
	
	//创建客户端对象
	KafkaProducer<String, String> kafkaProducer = new KafkaProducer<>(prop);
	
	for(int i = 0; i < 10; i++) {
		//回调函数异步发送
		kafkaProducer.send(new ProducerRecord<>("topic-1", "value-" + i), new Callback() {
			@Override
			public void onCompletion(RecordMetadata metadata, Exception exception) {
				if(exception == null) {
					System.out.println("主题: " + metadata.topic() + " 分区: " + metadata.partition());
				}
			}
		});
	}
	
	kafkaProducer.close();
```

### 2. 同步发送
```java
	//配置
	Properties prop = new Properties();
	//连接地址
	prop.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka1:9092,kafka2:9092,kafka3:9092");
	//key value序列化
	prop.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
	prop.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
	
	//创建客户端对象
	KafkaProducer<String, String> kafkaProducer = new KafkaProducer<>(prop);
	
	for(int i = 0; i < 10; i++) {
		//同步发布
		kafkaProducer.send(new ProducerRecord<>("topic-1", "value-" + i)).get();
	}
	
	kafkaProducer.close();
```

### 3. 分区
```java
public ProducerRecord(String topic, Integer partition, Long timestamp, K key, V value, Iterable<Header> headers)

public ProducerRecord(String topic, Integer partition, Long timestamp, K key, V value)

public ProducerRecord(String topic, Integer partition, K key, V value, Iterable<Header> headers)

public ProducerRecord(String topic, Integer partition, K key, V value)

public ProducerRecord(String topic, K key, V value)

public ProducerRecord(String topic, V value)
```

**自定义分区**
```java
//自定义分区器
public class MyPartitioner implements Partitioner {

	@Override
	public int partition(String topic, 
						Object key, byte[] keyBytes, 
						Object value, byte[] valueBytes, 
						Cluster cluster) {
		String msgValues = value.toString();
		int partition;
		
		if(msgValues.contains("0")) {
			partition = 0;
		} else {
			partition = 1;
		}
		
		return partition;
	}
}

//生产者配置
...
properties.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, "com. ... .MyPartitioner");
...
```

### 4. 提高吞吐量
提高`kafka`吞吐量，由四个配置参数决定
- `buffer.memory`：设置生产者内存缓冲区的大小，生产者用它缓冲要发送到`broker`服务器的消息；如果发送消息出去的速度小于写入消息进去的速度，就会导致缓冲区写满，此时生产消息就会阻塞住
- `compression.type`：消息被发送到`broker`之前使用压缩算法对消息进行压缩，通常存在`snappy gzp 1z4`压缩算法
- `batch.size`：多个消息被发送到同一分区，生产者则会它们放到同一个批次中；当批次被塞满则整个批次的消息会被发送出去
- `linger.ms`：生产者在发送批次之前等待更多消息加入批次的时间

```java
//kafka生产者配置
//发送缓冲区大小，默认32M
properties.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432);
//缓冲区中的批次大小，默认16k
properties.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
//在批次中的等待时间，默认0
properties.put(ProducerConfig.LINGER_MS_CONFIG, 1);
//压缩，默认none，可配置值qzip、snappy、lz4、zstd
properties.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "snappy");
```

### 5. 数据可靠
**数据可靠通过生产者ack三种应答级别实现**
```java
//发送确认
properties.put(ProducerConfig.ACKS_CONFIG, "all");
//发送重试
properties.put(ProducerConfig.RETRIES_CONFIG, 3);
```

### 6. 数据不重复与事务
`leader`分区的宕机可能导致已同步到`follower`的数据再次由生产者生产至`follower`，从而产生重复；批次数据请求可能也因为网络问题触发重试机制，因此需要**幂等**与**事务**
**幂等**保证单会话的数据不重复
**事务**保证多会话的数据不重复

开启幂等的前提是开启重试、开启`ack=all`、请求缓冲区请求批次数量不能大于5
开启事务的前提是开启幂等

```java
//幂等阻止单分区单会话数据重复，根据<pid, partition, seq>判断，pid为生产者会话id，partition为分区序号，seq为消息序号
properties.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, "true");
```

**开启事务必须开启幂等性**，幂等性由消息批次的主键：`<PID, Partition, SeqNumber>`起主要作用：
- `pid`：`ProducerID`，每个生产者启动时，`Kafka`都会分配生产者客户端一个`ID`，`Kafka`重启也会重新分配`pid`
- `partition`：消息需要发往的分区号
- `seqNumber`：生产者记录下来的消息的自增`ID`
`Kafka`不会持久化以上主键一致的数据，然而只能保证单分区内发送的批次数据不重复

**Kafka的事务机制可以实现对多个topic多个partition原子性的写入，即处于同一个事务内的所有消息，要么写入成功，要么写入失败**

<img src="D:\Project\IT-notes\框架or中间件\MQ\Kafka\img\kafka事务的工作流程.png" style="width:700px;height:500px;" />

1. 生产者启动，获取用户配置的事务`ID`；该生产者的事务信息会被存储到`__transaction_state`主题，该分区每个`broker`都有且默认有50个分区；根据事务`ID hash(transactional.id) % 50`计算出该事务属于哪个数据分区，这个数据分区的`leader`副本所在的`broker`节点即为该事务的事务协调器节点`transaction coordinator`
2. 生产者向`brokers`请求`producer id`生产者`ID`
3. 生产者获取到生产者`ID`后，`send`发送数据
4. 生产者发送完数据则`commit`提交请求
5. 生产者的事务相关的事务协调器会持久化本次`commit`请求
6. 事务协调器持久化`commit`成功后返回成功信号给生产者
7. 事务协调器向数据发送的目标分区发送`commit`请求
8. 数据目标分区返回成功则持久化本次发送事务到`__transaction_state`主题

```java
//初始化事务
void initTransactions();

//开启事务
void beginTransaction() throws ProducerFencedException;

//在事务内提交已经消费的偏移量(主要用于消费者)
void sendOffsetsToTransaction(Map<TopicPartition, OffsetAndMetadata> offsets, String consumerGroupId) throws ProducerFencedException;

//提交事务
void commitTransaction() throws ProducerFencedException;

//放弃事务，回滚
void abortTransaction() throws ProducerFencedException;
```

```java
//配置事务ID
properties.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, "transactional_id_01");

KafkaProducer<String, String> kafkaProducer = new KafkaProducer<>(properties);
kafkaProducer.initTransactions();
kafkaProducer.beginTransaction();

try {
	for(int i = 0; i < 5; i++) {
		kafkaProducer.send(new ProducerRecord<>("first", i));
	}
	kafkaProducer.commitTransaction();
} catch(Exception e) {
	kafkaProducer.abortTransaction();
} finally {
	kafkaProducer.close();
}
```

### 7. 有序乱序
生产者生产数据到多个分区，消费者希望消费有序数据，单分区内能实现有序；而多分区则需要消费者把分区中的数据全部拉去到消费者后，在消费者内进行排序

单分区中有可能出现因为某个消息发送失败导致最后接收到的数据乱序，存在多种解决方案
- `kafka 1.x`之前：`max.in.flight.requests.per.connection = 1`只允许生产者发送缓存里存在一个发送请求，因此发送失败则缓存中继续重试上一个请求
- `kafka 1.x`及之后，且未开启幂等：`max.in.flight.requests.per.connection = 1`
- `kafka 1.x`及之后，且开启幂等：`max.in.flight.requests.per.connection`设置为小于等于5，由于启用幂等后`kafka`会缓存生产者最近发送来的五个`request`的元数据；如果最近五个以内的`request`为乱序或者出现中间序号缺少的情况，会自动排序或者等待缺少的目标序号`request`出现并自动排序

## 8. Broker
<img src="D:\Project\IT-notes\框架or中间件\MQ\Kafka\img\kafka在zookeeper中的存储信息.png" style="width:700px;height:350px;" />

### 1. 总体工作流程
1. `broker`启动后会在`zookeeper`注册节点，注册即在`/brokers/ids`的`zookeeper`目录中添加节点信息
2. `zookeeper`中存在`kafka`的`/controller`目录，每个`broker`争先注册信息入`controller`，哪个`broker`抢占成功则哪个`broker`为`leader`选举的主持人
3. `broker`抢占`controller`目录成功后，对`/brokers/ids`目录的变化进行监听，开始`leader`的选举
4. 选举“在`isr`中存活且`broker`节点序号在`/brokers/ids`目录中排在前面的节点”作为`leader`，选举结束后，把选举结果：`leader`信息与节点信息上传`zookeeper`的`/brokers/topics/...`中
5. 其他从`broker`中的`controller`会从`/brokers/topics/...`拉去节点信息
6. 生产者生产数据，`kafka`进行主从同步

<img src="D:\Project\IT-notes\框架or中间件\MQ\Kafka\img\broker工作流程.png" style="width:700px;height:350px;" />

### 2. 节点服役与退役
向一个正在运行的旧集群中的`topic`新增一个新节点，或者重新分配正在使用的主题分区到新的`broker`列表，并重新同步原始数据到新的`broker`列表
#### 1. 服役
1. 在原有集群正常启动并运行的情况下，配置新的物理节点机器，配置`kafka`并正常启动连接至原有集群
2. 创建一个要负载均衡的主题，`vim topics-to-move.json`
```json
{
	"topic": [
		{"topic": "first"}
	],
	"version": 1
}
```
2. 生成一个负载均衡的计划，`bin/kafka-reassign-partitions.sh --bootstrap-server ip:port --topics-to-move-json-file topics-to-move.json --broker-list "0,1,2,3" --generate`
3. 将`kafka-reassign-partitions.sh`命令生成出来的新计划存入`increase-replication-factor.json`，并执行新计划`bin/kafka-reassign-partitions.sh --bootstrap-server ip:port --reassignment-json-file increase-replication-factor.json --execute`
4. 验证副本存储计划`bin/kafka-reassign-partitions.sh --bootstrap-server ip:port --reassignment-json-file increase-replication-factor.json --verify`

#### 2. 退役
1. 创建一个要负载均衡的主题，`vim topics-to-move.json`
```json
{
	"topic": [
		{"topic": "first"}
	],
	"version": 1
}
```
2. 生成一个负载均衡的计划`bin/kafka-reassign-partitions.sh --bootstrap-server hadoop102:9092 --topics-to-move-json-file topics-to-move.json --broker-list "0,1,2" --generate`，计划中已减少服役`broker`节点数量
3. 创建计划文件，`vim increase-replication-factor.json`
4. 执行新的计划，`bin/kafka-reassign-partitions.sh --bootstrap-server hadoop102:9092 --reassignment-json-file increase-replication-factor.json --execute`
5. 验证计划，`bin/kafka-reassign-partitions.sh --bootstrap-server hadoop102:9092 --reassignment-json-file increase-replication-factor.json --verify`

### 3. 副本
`kafka`副本，提高数据可靠性
1. `kafka`默认副本1个，生产环境一般配置2个，保证数据可靠性；太多副本会增加磁盘存储空间，增加网络上数据传输，降低效率
2. `kafka`副本分为`leader`与`follower`，生产者发送数据到`kafka`后，`follower`会从`leader`中同步数据
3. `kafka`分区中所有副本统称为`AR`，其中`AR = ISR + OSR`，`OSR`为长时间未向`leader`发送通信请求或同步数据的`follower`列表，`ISR`则为活跃的`follower`列表

### 4. Leader选举
<img src="D:\Project\IT-notes\框架or中间件\MQ\Kafka\img\broker工作流程.png" style="width:700px;height:350px;" />

### 5. Follower故障处理
<img src="D:\Project\IT-notes\框架or中间件\MQ\Kafka\img\LEO与HW.png" style="width:700px;height:300px;" />

`LEO log end offset`：每个副本的最后一个`offset`，`LEO = offset + 1`
`HW high watermark`：所有副本中最小的`LEO`，可以理解为`HW`为所有副本的`HW`，即分区的`HW`

如果`Follower`节点故障：
1. 将故障的`Follower`节点临时踢出`ISR`队列
2. 期间`Leader`与其他`Follower`继续接收数据
3. 故障的`Follower`节点恢复后，读取自己本地磁盘的故障前的`HW`记录，并将自己`log`文件高于`HW`的部分截取掉，从`HW`开始向`Leader`进行数据同步
4. 恢复后的`Follower`同步数据直至自己的`LEO`大于等于该分区所有副本的`HW`，即`Follower`追上`Leader`后，该`Follower`节点就可以重新加入`ISR`

### 6. Leader故障处理
1. `Leader`发生故障后，会从`ISR`中选出一个新的`Leader`
2. 为保证多个副本之间的数据一致性，其余的`Follower`会先将各自的`log`文件高于新`HW`的部分截掉，然后从新`Leader`同步数据

**Leader故障处理只能保证数据的一致性，并不能保证数据不丢失或者不重复**

### 7. 分区副本分配
例：有一个拥有四台机器的`kafka cluster`，创建一个名为`test`的`topic`，`topic`存在五个`partition`，每个`partition`一共拥有两个`replicate`：五个`partition`分别位于`broker-0、broker-1、broker-2、broker-3、broker-0`，每个`partition`的`follower replicate`于`leader replicate`所在`broker`的`id + 2`对应`broker id`的`broker`中

**默认宗旨：leader与follower均匀分配在多个机器中**

<img src="D:\Project\IT-notes\框架or中间件\MQ\Kafka\img\kafka默认分区副本分配方式例1.png" style="width:700px;height:350px;" />

<img src="D:\Project\IT-notes\框架or中间件\MQ\Kafka\img\kafka默认分区副本分配方式例2.png" style="width:700px;height:300px;" />

也会存在动态分配分区副本，即设置分区副本分配不均匀的场景：
1. 创建新`topic`：`bin/kafka-topics.sh --bootstrap-server ip:port --create --partitions 4 --replication-factor 2 --topic test`
2. 查看分区副本存储情况：`bin/kafka-topics.sh --bootstrap-server ip:port --describe --topic test`
3. 指定所有副本存储在`broker-0 broker-1`：`vim increase-replication-factor.json`
```json
{
	"version": 1, 
	"partitions": [
		{"topic":"test", "partition": 0, "replicas": [0,1]},
		{"topic":"test", "partition": 1, "replicas": [0,1]},
		{"topic":"test", "partition": 2, "replicas": [1,0]},
		{"topic":"test", "partition": 3, "replicas": [1,0]}
	]
}
```
4. 执行副本存储计划：`bin/kafka-reassign-partitions.sh --bootstrap-server ip:port --reassignment-json-file increase-replication-factor.json --execute`
5. 验证副本存储计划：`bin/kafka-reassign-partitions.sh --bootstrap-server ip:port --reassignment-json-file increase-replication-factor.json --verify`
6. 查看：`bin/kafka-topics.sh --bootstrap-server ip:port --describe --topic test`

### 8. 分区Leader的负载均衡
正常情况下，`kafka`本身会自动把`leader partition`均匀分散在各个机器上，来保证每台机器的读写吞吐量都是均匀的。但是如果某些`broker`宕机，会导致`leader partition`过于集中在其他少部分几台`broker`上，这会导致少数几台`broker`读写请求压力过高，其他宕机的`broker`重启之后都是`follower partition`，读写请求很低，造成集群负载不均衡

- `auto.leader.rebalance.enable`：默认为`true`，自动`leader partition`平衡
- `leader.imbalance.per.broker.percentage`：默认是10%，每个`broker`允许的不平衡的`leader`比率。如果每个`broker`超过这个值，控制器会触发`leader`的平衡
- `leader.imbalance.check.interval.seconds`：默认值300秒，检查`leader`负载是否平衡的间隔时间

### 9. 增加副本
需要对已经配置好分区数以及每个分区副本数量的topic，再次进行副本数量的配置修改
**手动添加副本**
1. 创建副本存储计划：`vim increase-replication-factor.json`
```json
{
	"version": 1, 
	"partitions": [
		{"topic":"test", "partition": 0, "replicas": [0,1]},
		{"topic":"test", "partition": 1, "replicas": [0,1]},
		{"topic":"test", "partition": 2, "replicas": [1,0]},
		{"topic":"test", "partition": 3, "replicas": [1,0]}
	]
}
```
2. 执行副本存储计划：`bin/kafka-reassign-partitions.sh --bootstrap-server ip:port --reassignment-json-file increase-replication-factor.json --execute`

## 9. 消费者
### 1. 消费方式
- `pull`：消费者主动拉取
- `push`：`broker`主动推送

如果`kafka`采用`push`的消费模式，则不一定所有消费者的消费速率都跟得上；而采用`push`，如果`kafka`中没有数据，则消费者可能会陷入循环中，一直返回空数据

### 2. 消费者组
**一个消费者可以消费一个分区的数据，一个消费者也可以消费多个分区的数据；而每个分区的数据只能由消费者组中的一个消费者消费，如果一个分区被一个消费者组中的多个消费者消费，则会需要进行数据去重，这是不被允许的**

当`kafka cluster`中出现某个`broker`忽然宕机，这个`broker`中的`leader replicate partition`被哪个消费者消费到偏移量为多少的记录上，都会由`offset`变量记录；这种情况下每个消费者都存在一个`offset`而且由消费者提交到系统主题`__consumer_offsets`保存，即落到磁盘上

`consumer group GC`：消费者组，由多个`consumer`组成，形成一个消费者组的条件，是所有消费者的`groupid`相同
- 消费者组内每个消费者负责消费不同分区的数据，一个分区只能由一个组内消费者消费
- 消费者组之间互不影响，所有消费者都属于某个消费者组，即消费者组是逻辑上的一个订阅者
- 每个消费者都隶属于一个消费者组，每个消费者都存在`groupid`，默认`groupid`不指定则自动生成
- 消费者组内的消费者数量多于分区数，则会存在若干消费者空闲

**消费者组初始化流程**：
1. 每个`broker`都存在一个`coordinator`组件辅助消费者组的消费`offset`提交（`offset`持久化），**消费者组需要选出该组对应的`coordinator`**：指定消费者组的`groupid`后，`hash(groupid) % 50`得到的`__consumer_offsets`主题具体的某个分区序号，序号对应的分区`broker`上的`coordinator`则作为这个消费者组的负责人，消费者组的所有消费者`offset`则向这个分区提交
2. 消费者组中的所有消费者主动向`coordinator`发送`joinGroup`入组请求，`coordinator`随机选出一个消费者作为消费者`leader`
3. `coordinator`会把收集到的所有消费者信息以及`topic`发送给`leader`消费者
4. 消费者`leader`制定消费计划，把消费计划发给`coordinator`
5. `coordinator`把消费计划发给其他消费者
（每个消费者都会与`coordinator`保持心跳，一旦超时则认为消费者挂了，需要移除消费者；移除消费者后需要将消费任务再平衡）

**消费者组详细消费流程**
1. 消费者组创建一个`consumerNetworkClient`客户端与`brokers`进行通信，`consumer`向`client`进行`sendFetches`发送消费请求，指定消费批次配置
	- `fetch.min.bytes`从`broker`中每批次最小抓取消费数据大小，默认一字节
	- `fetch.max.wait.ms`从`broker`中抓取一批消费数据的超时时间，默认500ms；批次实际内容大小未到达设置大小，只要超时则抓取
	- `fetch.max.bytes`每批次抓取消费数据的上限大小，默认50m
2. `consumerNetworkClient`向`broker`发送`send`请求，回调`onSuccess completedFetches`抓取数据，数据存入缓存队列`queue`中
3. 消费者组向缓存队列中进行数据拉取，可以设置`max.poll.records`配置批次拉取数据的一批次最大条数，先反序列化，再经过拦截器，最后处理数据

### 3. 消费一个主题
```java
//配置
Properties properties = new Properties();

//连接bootstrap.servers
properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "ip:port,ip:port");
//反序列化
properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
//消费者组id
properties.put(ConsumerConfig.GROUP_ID_CONFIG, "test");

//创建连接
KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<>(properties);

//订阅主题
ArrayList<String> topics = new ArrayList<>();
topics.add("test-topic");
kafkaConsumer.subscribe(topics);

//消费数据
while(true) {
	//消费间隔时间为1秒
	ConsumerRecords<String, String> consumerRecords = kafkaConsumer.poll(Duration.ofSeconds(1));
	
	for(ConsumerRecord<String, String> consumerRecord : consumerRecords) {
		log.info("{}", consumerRecord);
	}
}
```

### 4. 消费一个分区
```java
//配置
Properties properties = new Properties();

//连接bootstrap.servers
properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "ip:port,ip:port");
//反序列化
properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
//消费者组id
properties.put(ConsumerConfig.GROUP_ID_CONFIG, "test");

//创建连接
KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<>(properties);

//订阅主题对应的分区
ArrayList<TopicPartition> topicPartitions = new ArrayList<>();
topicPartitions.add(new TopicPartition("test-topic", 0));
kafkaConsumer.assign(topicPartitions);

//消费数据
while(true) {
	//消费间隔时间为1秒
	ConsumerRecords<String, String> consumerRecords = kafkaConsumer.poll(Duration.ofSeconds(1));
	
	for(ConsumerRecord<String, String> consumerRecord : consumerRecords) {
		log.info("{}", consumerRecord);
	}
}
```

### 5. 分区分配与再平衡
一个`consumer group`中有多个`consumer`组成，一个`topic`有多个`partition`组成，而消费组中的每个`consumer`具体来消费哪个`partition`，是存在策略配置的
又或者消费者组中的消费者与`coordinator`无法保持正常心跳，导致消费者被踢出消费者组，也会触发分区分配与再平衡

分区分配策略（默认`range + cooperativeSticky`，通过设置`partition.assignment.strategy`实现，`kafka`可以同时使用多种分区分配策略）：
- `range`：针对一个`topic`，按照`partitions / consumers`得到每个`consumer`需要消费的分区数，余数将一个一个按照`consumer`列表顺序分配给`consumer`
	- 容易使排在`consumer`列表前面的`consumer`消费过多分区，导致数据倾斜
	- 某个消费者被踢出后，再平衡，会把该被踢出消费者原本负责消费的分区分配给按照`consumer`列表顺序排在前面的`consumer`上
- `roundRobin`：与`range`不同，针对所有`topic`的所有`partition`，把所有的`partition`与所有`consumer`列出来并按照`hashcode`排序，最后通过轮询算法分配`partition`给每个消费者
	- 某个消费者被踢出后，再平衡，会把该被踢出消费者原本负责消费的分区轮询分配给剩余消费者
- `sticky`：尽量均匀而且随机的分配分区给消费者，尽量均匀地按照`partitions / consumers`得到每个`consumer`需要消费的分区数
	- 某个消费者被踢出后，再平衡，会把该被踢出消费者原本负责消费的分区尽量均匀随机分配给剩余消费者
- `cooperativeSticky`

| 参数名称 | 描述 |
| ----- | ----- |
| `heartbeat.interval.ms` | 消费者与`coordinator`之间的心跳时间，该配置项必须小于`session.timeout.ms`，也不应高于`session.timeout.ms`的1/3 |
| `session.timeout.ms` | 消费者和`coordinator`之间连接超时时间，超过该值，消费者被移除，消费者组执行再平衡 |
| `max.poll.interval.ms` | 消费者处理消息最大时长，超过该值，消费者被移除，消费者组执行再平衡 |
| `partition.assignment.strategy` | 分区分配策略 |

### 6. offset
消费者在消费分区时都会有一个`offset`记录消费者消费某个分区的记录偏移量，`offset`默认保存在`kafka`的内置`topic`：`__consumer_offsets`中

`__consumer_offsets`主题采用`key-value`方式存储数据。`key`是`groupid + topic + partitionSeq`，`value`则是`offset`的值。每隔一段时间，`kafka`内部会对这个`topic`进行`compact`压缩，以缩小`topic`占用空间并保留最新的记录
**可以设置exclude.internal.topics=false使系统主题可见，默认true系统主题不可见**

`kafka`自动提交`offset`配置：
- `enable.auto.commit`：是否开启自动提交`offset`功能，默认是`true`
- `auto.commit.interval.ms`：自动提交`offset`的时间间隔，默认`5s`

`kafka`手动提交`offset`分为两种：`commitSync`同步提交与`commitAsync`异步提交
- `commitSync`同步提交：必须等待`offset`提交完毕，再去消费下一批数据
- `commitAsync`异步提交：发送完提交`offset`请求后，就开始消费下一批数据

`commitSync`与`commitAsync`相同点与不同点：
- 相同：都会将本次一批数据中的最高的`offset`偏移量进行提交
- 不同：同步提交存在自动失败重试，异步提交没有失败重试机制

```java
//配置
Properties properties = new Properties();

//连接bootstrap.servers
properties.put(ConsumerConfig.BOOTSTRAP_SERVER_CONFIG, "ip:port,ip:port");
//反序列化
properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
//消费者组id
properties.put(ConsumerConfig.GROUP_ID_CONFIG, "test");

//手动提交offset
properties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);

//创建连接
KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<>(properties);

//订阅主题对应的分区
ArrayList<TopicPartition> topicPartitions = new ArrayList<>();
topicPartitions.add(new TopicPartition("test-topic", 0));
kafkaConsumer.assign(topicPartitions);

//消费数据
while(true) {
	//消费间隔时间为1秒
	ConsumerRecords<String, String> consumerRecords = kafkaConsumer.poll(Duration.ofSeconds(1));
	
	for(ConsumerRecord<String, String> consumerRecord : consumerRecords) {
		log.info("{}", consumerRecord);
	}
	
	//同步提交offset
	kafkaConsumer.commitSync();
	//异步提交offset
	kafkaConsumer.commitAsync();
}
```

指定`offset`位置进行消费，配置`auto.offset.reset`：
- `earliest`：自动将偏移量重置为最早的偏移量，即`--from-beginning`
- `latest`：默认值，自动将偏移量重置为最新的偏移量
- `none`

```java
//配置
Properties properties = new Properties();

//连接bootstrap.servers
properties.put(ConsumerConfig.BOOTSTRAP_SERVER_CONFIG, "ip:port,ip:port");
//反序列化
properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
//消费者组id
properties.put(ConsumerConfig.GROUP_ID_CONFIG, "test");

//创建连接
KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<>(properties);

//订阅主题
ArrayList<String> topics = new ArrayList<>();
topics.add("test");
kafkaConsumer.subscribe(topics);

//获取主题下所有分区
Set<TopicPartition> assignment = kafkaConsumer.assignment();

//保证消费者初始化broker分区信息完毕，即分区分配方案完成，消费者中已经有分区信息
while(assignment.size() == 0) {
	kafkaConsumer.poll(Duration.ofSeconds(1));
	assignment = kafkaConsumer.assignment();
}

//时间转化为offset，例如把offset置为一天前相同时刻的offset，即可将offset按照时间进行seek偏移
//HashMap<TopicPartition, Long> topicPartitionLongHashMap = new HashMap<>();
//for (TopicPartition topicPartition : assignment) {
	//topicPartitionLongHashMap.put(topicPartition, System.currentTimeMills() - 1 * 24 * 3600 * 1000);
//}
//
//
//Map<TopicPartition, OffsetAndTimestamp> map = kafkaConsumer.offsetsForTimes(topicPartitionLongHashMap);

for(TopicPartition topicPartition : assignment) {
	//指定主题、分区、offset
	kafkaConsumer.seek(topicPartition, 100);
	//OffsetAndTimestamp offsetAndTimestamp = map.get(topicPartition);
	//kafkaConsumer.seek(topicPartition, offsetAndTimestamp.offset());
}

//消费数据
while(true) {
	//消费间隔时间为1秒
	ConsumerRecords<String, String> consumerRecords = kafkaConsumer.poll(Duration.ofSeconds(1));
	
	for(ConsumerRecord<String, String> consumerRecord : consumerRecords) {
		log.info("{}", consumerRecord);
	}
}
```

### 7. 消费者事务
- **重复消费**：已经消费了数据，但是`offset`没提交（`offset`自动提交时在设置的自动提交时间间隔中消费者挂了，即使消费了数据也仍然从旧的`offset`重新开始消费）
- **漏消费**：先提交`offset`后消费，有可能会造成数据的漏消费（手动提交`offset`时消费的数据还在内存消费完毕但未落盘，而`offset`已被提交，此刻消费者挂了，则未落盘的消费数据丢失）

使用支持事务的消费者架构，如支持事务的数据库（`Mysql`），事务提交成功才提交`offset`

### 8. 数据积压
数据积压情况在生产者与消费者两端都可能出现：
- 消费能力不足，数据积压在消费者端，可以增加单个`topic`的分区数，并同时提升消费者组的消费者数量；也可以增大消费者单次拉取数据的批次大小
- 生产速度大于消费速度，也会出现生产者端的数据积压

## 10. kafka-eagle监控
如果是使用`zookeeper`模式的`kafka`，则可以使用`kafka-eagle`
如果是使用`kraft`模式的`kafka`，则可以使用`kafka-ui`进行监控

## 11. Kraft模式
**配置`./kafka/config/kraft/server.properties`**
```shell
process.roles=broker,controller

node.id=4

controller.quorum.voters=4@ip:9093,5@ip:9093,6@ip:9093

listeners=PLAINTEXT://:9092,CONTROLLER://:9093

advertised.listeners=PLAINTEXT://ip:port

controller_listener_names=CONTROLLER
```

## 12. springboot整合kafka
### 1. 生产者
```xml
<dependency>
	<groupId>org.springframework.kafka</groupId>
	<artifactId>spring-kafka</artifactId>
</dependency>
```

```yaml
server:
	port: 18081
spring:
	kafka:
	bootstrap-servers: 192.168.211.130:9092,192.168.211.130:9093,192.168.211.130:9094
	producer: # producer 生产者
		retries: 0 # 重试次数
		acks: 1 # 应答级别:多少个分区副本备份完成时向生产者发送ack确认(可选0、1、all/-1)
		batch-size: 16384 # 批量大小
		buffer-memory: 33554432 # 生产端缓冲区大小
		key-serializer: org.apache.kafka.common.serialization.StringSerializer
		value-serializer: org.apache.kafka.common.serialization.StringSerializer
```

```java
@RestController
@RequestMapping(value = "/producer")
public class SendController {

    @Autowired
    private KafkaTemplate kafkaTemplate;

    /***
     * 发送消息
     * topic:要发送的队列
     * msg:发送的消息
     */
    @GetMapping(value = "/send/{topic}/{msg}")
    public String send(@PathVariable(value = "topic")String topic,@PathVariable(value = "msg")String msg){
        //消息发送
        kafkaTemplate.send(topic,msg);
        return "SUCCESS";
    }
}

```

### 2. 消费者
```xml
<dependency>
	<groupId>org.springframework.kafka</groupId>
	<artifactId>spring-kafka</artifactId>
</dependency>
```

```yaml
server:
	port: 18082
spring:
	kafka:
	bootstrap-servers: 192.168.211.130:9092,192.168.211.130:9093,192.168.211.130:9094
	consumer: # consumer消费者
		group-id: mentugroup # 默认的消费组ID
		enable-auto-commit: true # 是否自动提交offset
		auto-commit-interval: 100  # 提交offset延时(接收到消息后多久提交offset)
		# earliest:当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费
		# latest:当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，消费新产生的该分区下的数据
		# none:topic各分区都存在已提交的offset时，从offset后开始消费；只要有一个分区不存在已提交的offset，则抛出异常
		auto-offset-reset: latest
		key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
		value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
```

```java
@Component
public class MessageListener {

    @KafkaListener(topics = {"itmentu"},groupId = "itmentuGroup")
    public void listener(ConsumerRecord<String,String> record){
        //获取消息
        String message = record.value();
        //消息偏移量
        long offset = record.offset();
        System.out.println("读取的消息："+message+"\n当前偏移量："+offset);
    }
}
```