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

