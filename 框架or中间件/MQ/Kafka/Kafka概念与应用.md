<img src="D:\Project\IT notes\框架or中间件\MQ\Kafka\img\Kafka3大纲.png" style="width:700px;height：400px;" />

## 1. 概述


## 2. 安装
### 1. 原生部署


### 2. docker部署
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