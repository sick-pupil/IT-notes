## 1. 安装启动
### 1. 二进制：
1. 解压二进制包`apache-shardingsphere-5.1.1-shardingsphere-proxy-bin.tar.gz`
2. 将`MySQL`驱动`jar`放到解压目录中的`ext-lib`目录
3. 修改配置`conf/server.yaml`
```yml
rules:
	- !AUTHORITY
		users:
			- root@%:root
	provider:
		type: ALL_PRIVILEGES_PERMITTED
props:
	sql-show: true
```
4. 启动`proxy`，`./start.sh`或者`start.bat`，指定端口`./start.sh ${proxy_port} ${proxy_conf_directory}`
5. 远程连接`mysql -h192.168.100.1 -P3307 -uroot -p`
6. `show databases;`

### 2. Docker
1. 启动`docker`容器
```shell
docker run -d \
-v /server/proxy-a/conf:/opt/shardingsphere-proxy/conf \
-v /server/proxy-a/ext-lib:/opt/shardingsphere-proxy/ext-lib \
-e ES_JAVA_OPTS="-Xmx256m -Xms256m -Xmn128m" \
-p 3321:3307 \
--name server-proxy-a \
apache/shardingsphere-proxy:5.1.1
```
2. 上传`MySQL`驱动到`/server/proxy-a/ext-lib`目录
3. 修改配置`server.yaml`，上传到`/server/proxy-a/conf`目录
```yaml
rules:
	- !AUTHORITY
		users:
			- root@%:root
	provider:
		type: ALL_PRIVILEGES_PERMITTED
props:
	sql-show: true
```
4. 重启容器`docker restart server-proxy-a`
5. 远程连接`proxy`，`mysql -h192.168.100.201 -P3321 -uroot -p`
6. `show databases`

*实时查看日志*
```shell
docker exec -it server-proxy-a env LANG=C.UTF-8 /bin/bash
tail -f /opt/shardingsphere-proxy/logs/stdout.log 
```
## 2. 读写分离
修改配置`/server/proxy-a/conf/config-readwrite-splitting.yaml`：
```yaml
schemaName: readwrite_splitting_db

dataSources:
	write_ds:
		url: jdbc:mysql://192.168.100.201:3306/db_user?serverTimezone=UTC&useSSL=false
		username: root
		password: 123456
		connectionTimeoutMilliseconds: 30000
		idleTimeoutMilliseconds: 60000
		maxLifetimeMilliseconds: 1800000
		maxPoolSize: 50
		minPoolSize: 1
	read_ds_0:
		url: jdbc:mysql://192.168.100.201:3307/db_user?serverTimezone=UTC&useSSL=false
		username: root
		password: 123456
		connectionTimeoutMilliseconds: 30000
		idleTimeoutMilliseconds: 60000
		maxLifetimeMilliseconds: 1800000
		maxPoolSize: 50
		minPoolSize: 1
	read_ds_1:
		url: jdbc:mysql://192.168.100.201:3308/db_user?serverTimezone=UTC&useSSL=false
		username: root
		password: 123456
		connectionTimeoutMilliseconds: 30000
		idleTimeoutMilliseconds: 60000
		maxLifetimeMilliseconds: 1800000
		maxPoolSize: 50
		minPoolSize: 1

rules:
- !READWRITE_SPLITTING
	dataSources:
		readwrite_ds:
			type: Static
			props:
				write-data-source-name: write_ds
				read-data-source-names: read_ds_0,read_ds_1
```

测试：
```sql
mysql> show databases;
mysql> use readwrite_splitting_db;
mysql> show tables;
mysql> select * from t_user;
mysql> select * from t_user;
mysql> insert into t_user(uname) values('wang5');
```

`SpringBoot`连接`Proxy`，**将proxy当作普通的数据库进行连接**：
```yml
# 应用名称
spring:
	application:
		name: sharding-proxy-demo
# 开发环境设置
	profiles:
		active: dev
# mysql数据库连接（proxy）
	datasource:
		driver-class-name: com.mysql.jdbc.Driver
		url: jdbc:mysql://192.168.100.201:3321/readwrite_splitting_db?serverTimezone=GMT%2B8&useSSL=false
		username: root
		password: root

#mybatis日志
mybatis-plus:
	configuration:
		log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

## 3. 垂直分片
```yml
schemaName: sharding_db

dataSources:
	ds_0:
		url: jdbc:mysql://192.168.100.201:3301/db_user?serverTimezone=UTC&useSSL=false
		username: root
		password: 123456
		connectionTimeoutMilliseconds: 30000
		idleTimeoutMilliseconds: 60000
		maxLifetimeMilliseconds: 1800000
		maxPoolSize: 50
		minPoolSize: 1
	ds_1:
		url: jdbc:mysql://192.168.100.201:3302/db_order?serverTimezone=UTC&useSSL=false
		username: root
		password: 123456
		connectionTimeoutMilliseconds: 30000
		idleTimeoutMilliseconds: 60000
		maxLifetimeMilliseconds: 1800000
		maxPoolSize: 50
		minPoolSize: 1

rules:
- !SHARDING
	tables:
		t_user:
			actualDataNodes: ds_0.t_user
		t_order:
			actualDataNodes: ds_1.t_order
```

## 4. 水平分片
```yml
schemaName: sharding_db

dataSources:
	ds_user:
		url: jdbc:mysql://192.168.100.201:3301/db_user?serverTimezone=UTC&useSSL=false
		username: root
		password: 123456
		connectionTimeoutMilliseconds: 30000
		idleTimeoutMilliseconds: 60000
		maxLifetimeMilliseconds: 1800000
		maxPoolSize: 50
		minPoolSize: 1
	ds_order0:
		url: jdbc:mysql://192.168.100.201:3310/db_order?serverTimezone=UTC&useSSL=false
		username: root
		password: 123456
		connectionTimeoutMilliseconds: 30000
		idleTimeoutMilliseconds: 60000
		maxLifetimeMilliseconds: 1800000
		maxPoolSize: 50
		minPoolSize: 1
	ds_order1:
		url: jdbc:mysql://192.168.100.201:3311/db_order?serverTimezone=UTC&useSSL=false
		username: root
		password: 123456
		connectionTimeoutMilliseconds: 30000
		idleTimeoutMilliseconds: 60000
		maxLifetimeMilliseconds: 1800000
		maxPoolSize: 50
		minPoolSize: 1

rules:
- !SHARDING
	tables:
		t_user:
			actualDataNodes: ds_user.t_user

		t_order:
			actualDataNodes: ds_order${0..1}.t_order${0..1}
			databaseStrategy:
				standard:
					shardingColumn: user_id
					shardingAlgorithmName: alg_mod
			tableStrategy:
				standard:
					shardingColumn: order_no
					shardingAlgorithmName: alg_hash_mod
			keyGenerateStrategy:
				column: id
				keyGeneratorName: snowflake
		t_order_item:
			actualDataNodes: ds_order${0..1}.t_order_item${0..1}
			databaseStrategy:
				standard:
					shardingColumn: user_id
					shardingAlgorithmName: alg_mod
			tableStrategy:
				standard:
					shardingColumn: order_no
					shardingAlgorithmName: alg_hash_mod
			keyGenerateStrategy:
				column: id
				keyGeneratorName: snowflake
	bindingTables:
		- t_order,t_order_item
	broadcastTables:
		- t_dict

	shardingAlgorithms:
		alg_inline_userid:
			type: INLINE
			props:
				algorithm-expression: server-order$->{user_id % 2}
		alg_mod:
			type: MOD
			props:
				sharding-count: 2
		alg_hash_mod:
			type: HASH_MOD
			props:
				sharding-count: 2
  
	keyGenerators:
		snowflake:
			type: SNOWFLAKE
```
