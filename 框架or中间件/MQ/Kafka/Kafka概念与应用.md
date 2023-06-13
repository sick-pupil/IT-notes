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
- 主题：`kafka-topic.sh`
- 消费者：`kafka-console-consumer.sh`

| 参数 | 描述 |
| ----- | ----- |
| --bootstrap-server <String: server toconnect to> | 连接的Kafka Broker主机名称和端口号 |
| --topic <String: topic> | 操作的topic名称 |
| --create | 创建主题 |
| --delete | 删除主题 |
| --alter | 修改主题 |
| --list | 查看所有主题 |
| --describe | 查看主题详细描述 |
| --partitions <Integer: # of partitions> | 设置分区数 |
| --replication-factor<Integer: replication factor> | 设置分区副本 |
| --config <String: name=value> | 更新系统默认的配置 |

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

## 5. 生产者
<img src="D:\Project\IT-notes\框架or中间件\MQ\Kafka\img\生产者发送消息流程.png" style="width:700px;height:700px;" />

<img src="D:\Project\IT-notes\框架or中间件\MQ\Kafka\img\Kafka生产者发送消息流程.png" style="width:700px;height:350px;" />

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

### 6. 数据不重复
```java
//幂等阻止单分区单会话数据重复，根据<pid, partition, seq>判断，pid为生产者会话id，partition为分区序号，seq为消息序号
properties.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, "true");
```

**开启事务必须开启幂等性**，幂等性由消息的主键：`<PID, Partition, SeqNumber>`起主要作用：
- `pid`：`ProducerID`，每个生产者启动时，`Kafka`都会分配生产者客户端一个`ID`，`Kafka`重启也会重新分配`pid`
- `partition`：消息需要发往的分区号
- `seqNumber`：生产者记录下来的消息的自增`ID`
`Kafka`不会持久化以上主键一致的数据，然而只能保证单分区内数据不重复

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

## 6. Broker
