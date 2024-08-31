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

## 5. 分片策略与算法
### 1. 分片策略
`shardingproxy`对外提供五种分片策略
- `standard`标准
- `complex`复合
- `hint`
- `none`无

#### 1. 标准分片
```yml
spring:
  shardingsphere:
    rules:
      sharding:
        tables:
          t_order: # 逻辑表名称
            # 数据节点：数据库.分片表
            actual-data-nodes: db$->{0..1}.t_order_${1..10}
            # 分库策略
            databaseStrategy: # 分库策略
              standard: # 用于单分片键的标准分片场景
                shardingColumn: order_id # 分片列名称
                shardingAlgorithmName: # 分片算法名称
           tableStrategy: # 分表策略，同分库策略
```
#### 2. 复合分片
```yml
spring:
  shardingsphere:
    rules:
      sharding:
        tables:
          t_order: # 逻辑表名称
            # 数据节点：数据库.分片表
            actual-data-nodes: db$->{0..1}.t_order_${1..10}
            # 分库策略
            databaseStrategy: # 分库策略
              complex: # 用于多分片键的复合分片场景
                shardingColumns: order_id，user_id # 分片列名称，多个列以逗号分隔
                shardingAlgorithmName: # 分片算法名称
            tableStrategy: # 分表策略，同分库策略
```
#### 3. `HINT`
```yml
spring:
  shardingsphere:
    rules:
      sharding:
        tables:
          t_order: # 逻辑表名称
            # 数据节点：数据库.分片表
            actual-data-nodes: db$->{0..1}.t_order_${1..10}
            # 分库策略
            databaseStrategy: # 分库策略
              hint: # Hint 分片策略
                shardingAlgorithmName: # 分片算法名称
            tableStrategy: # 分表策略，同分库策略
```
#### 4. 不分片
```yml
spring:
  shardingsphere:
    rules:
      sharding:
        tables:
          t_order: # 逻辑表名称
            # 数据节点：数据库.分片表
            actual-data-nodes: db$->{0..1}.t_order_${1..10}
            # 分库策略
            databaseStrategy: # 分库策略
              none: # 不分片
           tableStrategy: # 分表策略，同分库策略
```
### 2. 分片算法
#### 1. `MOD`
`分片键value % sharding-count`
```yml
spring:
  shardingsphere:
    rules:
      sharding:
        # 自动分片表规则配置
        auto-tables:
          t_order:
            actual-data-sources: db$->{0..1}
            sharding-strategy:
              standard:
                sharding-column: order_date
                sharding-algorithm-name: t_order_table_mod
        # 分片算法定义
        sharding-algorithms:
          t_order_table_mod:
            type: MOD # 取模分片算法
            props:
              # 指定分片数量
              sharding-count: 6
        tables:
          t_order: # 逻辑表名称
            actual-data-nodes: db$->{0..1}.t_order_${0..2}
            # 分库策略
            database-strategy:
            ....
            # 分表策略
            table-strategy:
              standard:
                sharding-column: order_id
                sharding-algorithm-name: t_order_table_mod
```
#### 2. `HASH_MOD`
`hash(分片键value) % sharding-count`
```yml
spring:
  shardingsphere:
    rules:
      sharding:
        # 自动分片表规则配置
        auto-tables:
          t_order:
            actual-data-sources: db$->{0..1}
            sharding-strategy:
              standard:
                sharding-column: order_date
                sharding-algorithm-name: t_order_table_hash_mod
        # 分片算法定义
        sharding-algorithms:
          t_order_table_hash_mod:
            type: HASH_MOD # 哈希取模分片算法
            props:
              # 指定分片数量
              sharding-count: 6
        tables:
          t_order: # 逻辑表名称
            actual-data-nodes: db$->{0..1}.t_order_${0..2}
            # 分库策略
            database-strategy:
            ....
            # 分表策略
            table-strategy:
              standard:
                sharding-column: order_id
                sharding-algorithm-name: t_order_table_hash_mod
```
#### 3. `VOLUME_RANGE`
`range-lower`与`range-upper`选择进行分片的范围，为`long`类型，因此分片范围也是整形数值范围；`sharding-volume`决定每个分片的容量，即每个分片的记录数量
```yml
# 分片算法定义
spring:
  shardingsphere:
    rules:
      sharding:
        # 自动分片表规则配置
        auto-tables:
          t_order:
            actual-data-sources: db$->{0..1}
            sharding-strategy:
              standard:
                sharding-column: order_date
                sharding-algorithm-name: t_order_table_volume_range
        sharding-algorithms:
          t_order_table_volume_range:
            type: VOLUME_RANGE
            props:
              range-lower: 2 # 范围下界，超过边界的数据会报错
              range-upper: 20 # 范围上界，超过边界的数据会报错
              sharding-volume: 10 # 分片容量
        tables:
          t_order: # 逻辑表名称
            actual-data-nodes: db$->{0..1}.t_order_${0..2}
            # 分库策略
            database-strategy:
            ....
            # 分表策略
            table-strategy:
              standard:
                sharding-column: order_id
                sharding-algorithm-name: t_order_table_volume_range
```
#### 4. `BOUNDARY_RANGE`
`sharding-ranges`标记每个分片的上下边界
```yml
spring:
  shardingsphere:
    rules:
      sharding:
        # 自动分片表规则配置
        auto-tables:
          t_order:
            actual-data-sources: db$->{0..1}
            sharding-strategy:
              standard:
                sharding-column: order_date
                sharding-algorithm-name: t_order_table_boundary_range
        sharding-algorithms:
          # 基于分片边界的范围分片算法
          t_order_table_boundary_range:
            type: BOUNDARY_RANGE
            props:
              sharding-ranges: 10,20,30,40 # 分片的范围边界，多个范围边界以逗号分隔
        tables:
          t_order: # 逻辑表名称
            actual-data-nodes: db$->{0..1}.t_order_${0..2}
            # 分库策略
            database-strategy:
            ....
            # 分表策略
            table-strategy:
              standard:
                sharding-column: order_id
                sharding-algorithm-name: t_order_table_boundary_range
```
#### 5. `AUTO_INTERVAL`
`datetime-lower`与`datetime-upper`选择进行分片的范围，使用`yyyy-MM-dd HH:mm:ss`标记时间上下边界，`sharding-seconds`决定每个分片的容量，为整形类型，单位为秒
```yml
spring:
  shardingsphere:
    rules:
      sharding:
        # 自动分片表规则配置
        auto-tables:
          t_order:
            actual-data-sources: db$->{0..1}
            sharding-strategy:
              standard:
                sharding-column: order_date
                sharding-algorithm-name: t_order_table_auto_interval
        # 分片算法定义
        sharding-algorithms:
          # 自动时间段分片算法
          t_order_table_auto_interval:
            type: AUTO_INTERVAL
            props:
              datetime-lower: '2023-01-01 00:00:00' # 分片的起始时间范围，时间戳格式：yyyy-MM-dd HH:mm:ss
              datetime-upper: '2025-01-01 00:00:00' #  分片的结束时间范围，时间戳格式：yyyy-MM-dd HH:mm:ss
              sharding-seconds: 31536000 # 单一分片所能承载的最大时间，单位：秒，允许分片键的时间戳格式的秒带有时间精度，但秒后的时间精度会被自动抹去
        tables:
          # 逻辑表名称
          t_order:
            # 数据节点：数据库.分片表
            actual-data-nodes: db$->{0..1}.t_order_${0..2}
            # 分库策略
            database-strategy:
              standard:
                sharding-column: order_date
                sharding-algorithm-name: t_order_table_auto_interval
            # 分表策略
#            table-strategy:
#              standard:
#                sharding-column: order_date
#                sharding-algorithm-name: t_order_table_auto_interval
```
#### 6. `INLINE`
`algorithm-expression`为分片行表达式；**`INLINE`该算法只支持`SQL`使用`=`与`IN`操作符，`allow-range-query-with-inline-sharding`为`true`，则`SQL`可以使用`=`与`IN`操作符，但开启范围查询后会无视分片策略进行全库表路由**
```yml
spring:
  shardingsphere:
    # 具体规则配置
    rules:
      sharding:
        # 分片算法定义
        sharding-algorithms:
          # 标准分片算法
          # 行表达式分片算法
          t_order_table_inline:
            type: INLINE
            props:
              algorithm-expression:	t_order_$->{order_id % 3} # 分片算法的行表达式
              allow-range-query-with-inline-sharding: false # 是否允许范围查询。注意：范围查询会无视分片策略，进行全路由，默认 false
        tables:
          # 逻辑表名称
          t_order:
            # 数据节点：数据库.分片表
            actual-data-nodes: db$->{0..1}.t_order_${0..2}
            # 分库策略
            database-strategy:
              standard:
                sharding-column: order_id
                sharding-algorithm-name: t_order_database_algorithms
            # 分表策略
            table-strategy:
              standard:
                sharding-column: order_id
                sharding-algorithm-name: t_order_table_inline
```
#### 7. `INTERVAL`
`datetime-pattern`：分片键值时间格式，如“`yyyy-MM-dd HH:mm:ss yyyy-MM-dd`”
`datetime-lower datetime-upper`：分片键值上下边界，格式必须与`datetime-pattern`
`sharding-suffix-pattern`：分片表后缀名，格式必须与`datetime-interval-unit`表达的一致，如`MONTH`对应`yyyy-MM`
`datetime-interval-unit`：分片间隔单位，每个分片的时间大小单位，如`MONTH`、`DAYS`
`datetime-interval-amount`：分片间隔数量，为整形类型，配合`datetime-interval-unit`形成分片容量
```yml
spring:
  shardingsphere:
    rules:
      sharding:
        # 分片算法定义
        sharding-algorithms:
          t_order_database_mod:
            type: MOD
            props:
              sharding-count: 2 # 指定分片数量
          t_order_table_interval:
            type: INTERVAL
            props:
              datetime-pattern: "yyyy-MM-dd HH:mm:ss"  # 分片字段格式
              datetime-lower: "2024-01-01 00:00:00"  # 范围下限
              datetime-upper: "2024-06-30 23:59:59"  # 范围上限
              sharding-suffix-pattern: "yyyyMM"  # 分片名后缀，可以是MM，yyyyMMdd等。
              datetime-interval-amount: 1  # 分片间隔，这里指一个月
              datetime-interval-unit: "MONTHS" # 分片间隔单位
        tables:
          # 逻辑表名称
          t_order:
            # 数据节点：数据库.分片表
            actual-data-nodes: db$->{0..1}.t_order_${202401..202406}
            # 分库策略
            database-strategy:
              standard:
                sharding-column: order_id
                sharding-algorithm-name: t_order_database_mod
            # 分表策略
            table-strategy:
              standard:
                sharding-column: interval_value
                sharding-algorithm-name: t_order_table_interval
            keyGenerateStrategy:
              column: id
              keyGeneratorName: t_order_snowflake
```
#### 8. `COMPLEX_INLINE`
`sharding-columes`决定复合的多个分片键，`allow-range-query-with-inline-sharding`是否允许范围查询，`algorithm-expression`分片行表达式
```yml
spring:
  shardingsphere:
    # 具体规则配置
    rules:
      sharding:
        # 分片算法定义
        sharding-algorithms:
          t_order_database_complex_inline_algorithms:
            type: COMPLEX_INLINE
            props:
              sharding-columns: order_id, user_id # 分片列名称，多个列用逗号分隔。
              algorithm-expression: db$->{(order_id + user_id) % 2} # 分片算法的行表达式
              allow-range-query-with-inline-sharding: false # 是否允许范围查询。注意：范围查询会无视分片策略，进行全路由，默认 false
          # 11、复合行表达式分片算法
          t_order_table_complex_inline:
            type: COMPLEX_INLINE
            props:
              sharding-columns: order_id, user_id # 分片列名称，多个列用逗号分隔。
              algorithm-expression: t_order_$->{ (order_id + user_id) % 3 } # 分片算法的行表达式
              allow-range-query-with-inline-sharding: false # 是否允许范围查询。注意：范围查询会无视分片策略，进行全路由，默认 false
        tables:
          # 逻辑表名称
          t_order:
            # 数据节点：数据库.分片表
            actual-data-nodes: db$->{0..1}.t_order_${0..2}
            # 分库策略
            database-strategy:
              complex:
                shardingColumns: order_id, user_id
                sharding-algorithm-name: t_order_database_complex_inline_algorithms
            # 分表策略
            table-strategy:
              complex:
                shardingColumns: order_id, user_id
                sharding-algorithm-name: t_order_table_complex_inline
            keyGenerateStrategy:
              column: id
              keyGeneratorName: t_order_snowflake
```
#### 9. `HINT_INLINE`
强制路由分片，使用`Groovy`表达式实现分片逻辑，并通过`HintManager`传入分库分表值
```yml
spring:
  shardingsphere:
    rules:
      sharding:
        # 分片算法定义
        sharding-algorithms:
          # Hint 行表达式分片算法
          t_order_database_hint_inline:
            type: HINT_INLINE
            props:
              algorithm-expression: db$->{Integer.valueOf(value) % 2} # 分片算法的行表达式，默认值${value}
          t_order_table_hint_inline:
            type: HINT_INLINE
            props:
              algorithm-expression: t_order_$->{Integer.valueOf(value) % 3} # 分片算法的行表达式，默认值${value}
        tables:
          # 逻辑表名称
          t_order:
            # 数据节点：数据库.分片表
            actual-data-nodes: db$->{0..1}.t_order_${0..2}
            # 分库策略
            database-strategy:
              hint:
                sharding-algorithm-name: t_order_database_hint_inline
            # 分表策略
            table-strategy:
              hint:
                sharding-algorithm-name: t_order_table_hint_inline
            keyGenerateStrategy:
              column: id
              keyGeneratorName: t_order_snowflake
```

```java
@DisplayName("测试 hint_inline 分片算法插入数据")
@Test
public void insertHintInlineTableTest() {
      HintManager hintManager = HintManager.getInstance();
      hintManager.clearShardingValues();
      // 设置逻辑表 t_order 的分库值
      hintManager.addDatabaseShardingValue("t_order", 0);
      // 设置逻辑表 t_order 的分表值
      hintManager.addTableShardingValue("t_order", 1);
      // 1%3 = 1 所以放入 db0.t_order_1 分片表
      jdbcTemplate.execute("INSERT INTO `t_order`(`id`,`order_date`,`order_id`, `order_number`, `customer_id`, `total_amount`, `interval_value`, `user_id`) VALUES (1, '2024-03-20 00:00:00', 1, '1', 1, 1.00, '2024-01-01 00:00:00', 1);");
      hintManager.close();
}
```

## 6. 主从负载均衡
主从模式中的从机设置负载均衡
- `ROUND_ROBIN`：轮询负载均衡算法
- `RANDOM`：随机负载均衡算法
- `WEIGHT`：权重负载均衡算法

```yml
rules:
- !READWRITE_SPLITTING
  dataSourceGroups:
    readwrite_ds:
      writeDataSourceName: write_ds
      readDataSourceNames:
        - read_ds_0
        - read_ds_1
      loadBalancerName: random
      transactionalReadQueryStrategy: PRIMARY
  loadBalancers:
    random:
      type: RANDOM
      props:
```