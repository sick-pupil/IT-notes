<img src="D:\Project\IT-notes\框架or中间件\ShardingSphere\ShardingSphere-JDBC\img\程序代理.jpg" style="width:700px;height:400px;" />

分布式系统涉及读写操作，只能保证一致性`Consistence`、可用性`Availability`、分区容错性`Partition Tolerance`三者中的两个，另一个必须被牺牲，即**保证`CAP`**
- **C 一致性（Consistency）**：对某个指定的客户端来说，读操作保证能够返回最新的写操作结果
- **A 可用性（Availability）**：非故障的节点在合理的时间内返回合理的响应（不是错误和超时的响应）
- **P 分区容忍性（Partition Tolerance）**：当出现网络分区后（可能是丢包，也可能是连接中断，还可能是拥塞），系统能够继续履行职责

不可能所有系统都能保持`CAP`三种特性，要不选择`CP`，要不选择`AP`，分布式系统理论上不可能选择`CA`架构
- **CP**：系统为了保持一致性，在某一节点发生数据变化后，变化会被同步到其他节点，但数据变化可能因为信道问题无法复制成功，此时其他未同步成功的节点被访问时，会提示错误、未同步成功
- **AP**：系统不保持一致性，集群中某节点发生数据变化，而因为信道问题变化未被同步到其余节点时，访问其余节点会返回旧数据

*而数据同步复制过程中，节点之间不可能实现完美的强一致性，但可以做到最终一致性，如：保证部分核心可用、一致性过程存在中间状态*
## 1. 读写分离
- 主库负责处理事务性的增删改操作，从库负责处理查询操作，能够有效避免由数据更新导致的行锁，使得整个系统的查询性能得到极大的改善
- 读写分离是根据`SQL`语义的分析，将读操作和写操作分别路由至主库与从库
- 通过**一主多从**的配置方式，可以将查询请求均匀的分散到多个数据副本，能够进一步的提升系统的处理能力
- 使用**多主多从**的方式，不但能够提升系统的吞吐量，还能够提升系统的可用性，可以达到在任何一个数据库宕机，甚至磁盘物理损坏的情况下仍然不影响系统的正常运行

### 1. MySQL主从复制
<img src="D:\Project\IT-notes\框架or中间件\ShardingSphere\ShardingSphere-JDBC\img\MySQL主从同步原理.png" style="width:700px;height:300px;" />

**Slave会主动从Master读取binlog进行数据同步**
1. `master`将数据改变记录到二进制日志`binlog`中
2. 当`slave`上执行 `start slave` 命令之后，slave会创建一个`IO 线程`用来连接`master`，请求`master`中的`binlog`
3. 当`slave`连接`master`时，`master`会创建一个`log dump`线程，用于发送`binlog`的内容。在读取`binlog`的内容的操作中，会对主节点上的`binlog`加锁，当读取完成并发送给从服务器后解锁
4. `IO`线程接收主节点`binlog dump`进程发来的更新之后，保存到中继日志`relay log` 中
5. `slave`的`SQL`线程，读取`relay log`日志，并解析成具体操作，从而实现主从操作一致，最终数据一致
#### 示例
- 主服务器：容器名`mysql-master`，端口`3306`
- 从服务器：容器名`mysql-slave1`，端口`3307`
- 从服务器：容器名`mysql-slave2`，端口`3308`

##### 1. Master
```shell
#关闭docker
systemctl stop docker
#关闭防火墙
systemctl stop firewalld
#启动docker
systemctl start docker
```

```shell
docker run -d \
-p 3306:3306 \
-v /mysql/master/conf:/etc/mysql/conf.d \
-v /mysql/master/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
--name mysql-master \
mysql:8.0.29
```

```shell
vim /mysql/master/conf/my.cnf
```

```properties
[mysqld]
# 服务器唯一id，默认值1
server-id=1
# 设置日志格式，默认值ROW
binlog_format=STATEMENT
# 二进制日志名，默认binlog
# log-bin=binlog
# 设置需要复制的数据库，默认复制全部数据库
#binlog-do-db=mytestdb
# 设置不需要复制的数据库
#binlog-ignore-db=mysql
#binlog-ignore-db=infomation_schema
```

```shell
docker restart mysql-master
```

```shell
#进入容器：env LANG=C.UTF-8 避免容器中显示中文乱码
docker exec -it atguigu-mysql-master env LANG=C.UTF-8 /bin/bash
#进入容器内的mysql命令行
mysql -uroot -p
#修改默认密码校验方式
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';

# 创建slave用户
CREATE USER 'atguigu_slave'@'%';
# 设置密码
ALTER USER 'atguigu_slave'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
# 授予复制权限
GRANT REPLICATION SLAVE ON *.* TO 'atguigu_slave'@'%';
# 刷新权限
FLUSH PRIVILEGES;
```

```shell
SHOW MASTER STATUS;
# 需要记下File以及Position的值，用于配置slave从机
```
##### 2. Salve
```shell
docker run -d \
-p 3307:3306 \
-v /mysql/slave1/conf:/etc/mysql/conf.d \
-v /mysql/slave1/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
--name mysql-slave1 \
mysql:8.0.29
```

```shell
vim /mysql/slave1/conf/my.cnf
```

```properties
[mysqld]
# 服务器唯一id，每台服务器的id必须不同，如果配置其他从机，注意修改id
server-id=2
# 中继日志名，默认xxxxxxxxxxxx-relay-bin
# relay-log=relay-bin
```

```shell
docker restart mysql-slave1

#进入容器：
docker exec -it atguigu-mysql-slave1 env LANG=C.UTF-8 /bin/bash
#进入容器内的mysql命令行
mysql -uroot -p
#修改默认密码校验方式
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';

# 配置主从关系
CHANGE MASTER TO MASTER_HOST='192.168.100.201', 
MASTER_USER='atguigu_slave',MASTER_PASSWORD='123456', MASTER_PORT=3306,
MASTER_LOG_FILE='binlog.000003',MASTER_LOG_POS=1357;

START SLAVE;
# 查看状态（不需要分号）
SHOW SLAVE STATUS\G
# 如果出现Slave_IO_Running、Slave_SQL_Running均为Yes，则从机启动并配置成功
```

### 2. ShardingJDBC读写分离
```xml
<dependency>
	<groupId>org.apache.shardingsphere</groupId>
	<artifactId>shardingsphere-jdbc-core-spring-boot-starter</artifactId>
	<version>5.1.1</version>
</dependency>
```

```yml
# 应用名称
spring: 
	application: 
		name: sharging-jdbc-demo
	profiles: 
		active: dev
	shardingsphere: 
		# 打印SQl
		props:
			sql-show: true
		mode: 
			# 内存模式
			type: Memory
		datasource:
			# 配置真实数据源
			names: master,slave1,slave2
			master: 
				type: com.zaxxer.hikari.HikariDataSource
				driver-class-name: com.mysql.jdbc.Driver
				jdbc-url: jdbc:mysql://192.168.100.201:3306/db_user
				username: root
				password: 123456
			slave1:
				type: com.zaxxer.hikari.HikariDataSource
				driver-class-name: com.mysql.jdbc.Driver
				jdbc-url: jdbc:mysql://192.168.100.201:3307/db_user
				username: root
				password: 123456
			slave2:
				type: com.zaxxer.hikari.HikariDataSource
				driver-class-name: com.mysql.jdbc.Driver
				jdbc-url: jdbc:mysql://192.168.100.201:3308/db_user
				username: root
				password: 123456
		rules:
			readwrite-splitting:
				data-sources:
					myds:
						# 读写分离类型，Static Dynamic
						type: Static
						props: 
							# 写数据源名称
							write-data-source-name: master
							# 读数据源名称，多个从数据源用逗号分隔
							read-data-source-names: slave1,slave2
						load-balancer-name: alg_round
				# 负载均衡算法配置
				# 负载均衡算法类型
				load-balancers:
					alg_round:
						type: ROUND_ROBIN
					alg_random: 
						type: RANDOM
					alg_weight:
						type: WEIGHT
						props:
							slave1: 1
							slave2: 2
```

**ShardingJDBC的主从模型中，事务中的数据读写均用主库**
## 2. 垂直分库
```yml
# 应用名称
spring: 
	application: 
		name: sharging-jdbc-demo
	profiles: 
		active: dev
	shardingsphere: 
		# 打印SQl
		props:
			sql-show: true
		mode: 
			# 内存模式
			type: Memory
		datasource:
			# 配置真实数据源
			names: server-user,server-order
			server-user: 
				type: com.zaxxer.hikari.HikariDataSource
				driver-class-name: com.mysql.jdbc.Driver
				jdbc-url: jdbc:mysql://192.168.100.201:3301/db_user
				username: root
				password: 123456
			server-order:
				type: com.zaxxer.hikari.HikariDataSource
				driver-class-name: com.mysql.jdbc.Driver
				jdbc-url: jdbc:mysql://192.168.100.201:3302/db_order
				username: root
				password: 123456
		# 路由规则
		rules:
			sharding:
				tables:
					# 逻辑表
					t_user:
						# 数据源与实际物理表
						actual-data-nodes: server-user.t_user
					t_order:
						actual-data-nodes: server-order.t_order
```

## 3. 水平分库分表
```yml
# 应用名称
spring: 
	application: 
		name: sharging-jdbc-demo
	profiles: 
		active: dev
	shardingsphere: 
		# 打印SQl
		props:
			sql-show: true
		mode: 
			# 内存模式
			type: Memory
		datasource:
			# 配置真实数据源
			names: server-user,server-order0,server-order1
			server-user: 
				type: com.zaxxer.hikari.HikariDataSource
				driver-class-name: com.mysql.jdbc.Driver
				jdbc-url: jdbc:mysql://192.168.100.201:3301/db_user
				username: root
				password: 123456
			server-order0:
				type: com.zaxxer.hikari.HikariDataSource
				driver-class-name: com.mysql.jdbc.Driver
				jdbc-url: jdbc:mysql://192.168.100.201:3310/db_order
				username: root
				password: 123456
			server-order1:
				type: com.zaxxer.hikari.HikariDataSource
				driver-class-name: com.mysql.jdbc.Driver
				jdbc-url: jdbc:mysql://192.168.100.201:3311/db_order
				username: root
				password: 123456
		rules:
			sharding:
				tables:
					# 逻辑表
					t_user:
						# 数据源与物理表
						actual-data-nodes: server-user.t_user
#					t_order:
#						actual-data-nodes: server-order0.t_order0,server-order0.t_order1,server-order1.t_order0,server-order1.t_order1
					# 逻辑表
#					t_order:
						# 数据源与物理表
#						actual-data-nodes: server-order$->{0..1}.t_order0
					# 逻辑表
					t_order:
						# 数据源与物理表
						actual-data-nodes: server-order$->{0..1}.t_order$->{0..1}
						# 分库策略
						database-strategy:
							standard:
								# 分片列
								sharding-column: user_id
#								sharding-column: order_no
								# 分片算法名称
								sharding-algorithm-name: alg_inline_userid
#								sharding-algorithm-name: alg_mod
#								sharding-algorithm-name: alg_hash_mod
						# 分表策略
						table-strategy:
							standard:
								# 分片列
								sharding-column: order_no
								# 分片算法名称
								sharding-algorithm-name: alg_hash_mod
						# 分布式id生成目标列与生成算法名称
						key-generate-strategy:
							# 分布式id生成目标列
							column: id
							# 生成算法名称
							key-generator-name: alg_snowflake
				# 分布式id生成策略
				key-generators:
					# 生成算法名称
					alg_snowflake:
						# 实际生成算法
						type: SNOWFLAKE
				# 分片算法类型
				sharding-algorithms: 
					# 分片算法名称
					alg_inline_userid: 
						# 分片算法，普通计算
						type: INLINE
						# 分片算法参数
						props:
							algorithm-expression: server-order$->{user_id % 2}
					# 分片算法名称
					alg_mod:
						# 分片算法：取模
						type: MOD
						# 分片算法参数
						props:
							sharding-count: 2
					# 分片算法名称
					alg_hash_mod:
						# 分片算法，hash
						type: HASH_MOD
						# 分片算法参数
						props:
							sharding-count: 2
```

```java
//修改order表的id生成策略

//@TableId(type = IdType.AUTO)//依赖数据库的主键自增策略
@TableId(type = IdType.ASSIGN_ID)//分布式id，使用mybatis-plus的主键生成策略

//如果配置了ShardingSphereJDBC的分布式id生成策略，则使用数据库自身的主键自增策略
@TableId(type = IdType.AUTO)
```

## 4. 多表关联
```yml
#------------------------标准分片表配置（数据节点配置）
spring:
	shardingsphere:
		rules:
			sharding:
				tables:
					t_order_item:
						actual-data-nodes: server-order$->{0..1}.t_order_item$->{0..1}
						database-strategy:
							standard:
								sharding-column: user_id
								sharding-algorithm-name: alg_mod
						table-strategy:
							standard:
								sharding-column: order_no
								sharding-algorithm-name: alg_hash_mod
						key-generate-strategy:
							column: id
							key-generator-name: alg_snowflake
```

**多表关联时，尽量让多表之间的分库分表策略保持一致，这样产生实际的SQL语句数量就不会出现笛卡尔积，保证分库分表后的多表关联不会产生跨库SQL**

**可以使用绑定表关系，实现分库分表后的多表关联尽量在同一个库中关联**
**绑定表**：指分片规则一致的一组分片表。 使用绑定表进行多表关联查询时，必须使用分片键进行关联，否则会出现笛卡尔积关联或跨库关联，从而影响查询效率
```yml
#------------------------绑定表
spring:
	shardingsphere:
		rules:
			sharding:
				binding-tables[0]: t_order,t_order_item
```
## 5. 广播表
可以将字典表常量表作为广播表，让多个分出来的分库都有一份广播表，避免其余表关联广播表产生过多跨库SQL
在其中一个分库中对广播表进行增删改，其余分库的同一张广播表也会作同样的增删改

```yml
#数据节点可不配置，默认情况下，向所有数据源广播
spring:
	shardingsphere:
		rules:
			sharding:
				tables:
					t_dict:
						actual-data-nodes: server-user.t_dict,server-order$->{0..1}.t_dict
				broadcast-tables[0]: t_dict
```