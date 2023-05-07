### 1. 入门基础
**数据库中间件**

使用`mycat`原因：
1. `Java`与数据库紧耦合
2. 高访问量高并发对数据库的压力
3. 读写请求数据不一致

`mycat`官网：`http://www.mycat.org.cn`

#### 读写分离
<img src="D:\Project\IT notes\框架or中间件\Mycat2\img\Mycat读写分离.png" style="width:700px;height:350px;" />

#### 数据分片
<img src="D:\Project\IT notes\框架or中间件\Mycat2\img\Mycat数据分片.png" style="width:700px;height:400px;" />

#### 多数据源整合
<img src="D:\Project\IT notes\框架or中间件\Mycat2\img\Mycat多数据源整合.png" style="width:700px;height:400px;" />

`Mycat`原理中比较重要的一点：**拦截**，拦截了用户发送过来的`SQL`语句，对语句做了一定的分析，如分片分析、路由分析、读写分离分析、缓存分析等，然后将语句送往特定真实数据库执行，结果作适当处理返回给用户

### 2. 安装使用
下载`mycat2-install-template`与对应`jar`包，将`jar`包放入解压后`mycat2-install-template`中的`lib`目录；并将`bin`目录中的文件权限设置成管理者权限，避免运行时由于权限不足导致报错

创建`mycat`连接`mysql`数据库所使用的用户
```
username:创建的用户名称
host:指定该用户在哪个主机上可以登录，可以配置具体域名或IP地址，也可以配置localhost或%
password:用户登录密码，可以为空
create user 'username'@'host' identified by 'password'

privileges:用户操作权限，如select、insert、update、all
databasename:数据库名
tablename:表名，可以填写具体表名或*
grant privileges on databasename.tablename to 'username'@'host'
```

`mycat`配置`mycat2\conf\datasources\prototypeDs.datasource.json`
```json
{
	"dbType":"mysql",
	"idleTimeout":60000,
	"initSqls":[],
	"initSqlsGetConnection":true,
	"instanceType":"READ_WRITE",
	"maxCon":1000,
	"maxConnectTimeout":3000,
	"maxRetryCount":5,
	"minCon":1,
	"name":"prototypeDs",
	//连接物理库的密码
	"password":"56962438",
	"type":"JDBC",
	//连接物理库的url
	"url":"jdbc:mysql://localhost:3306/mycat_test?useUnicode=true&serverTimezone=Asia/Shanghai&characterEncoding=UTF-8",
	//连接物理库的用户名
	"user":"mycat",
	"weight":0
}
```

`mycat`配置`mycat2\conf\users\mycat.user.json`
```json
{
	"dialect":"mysql",
	"ip":null,
	"password":"56962438",
	"transactionType":"xa",
	"username":"mycat"
}
```

**`prototypeDs.datasource.json`与`${username}.user.json`中的用户名密码要一致**

`mycat`命令
```
./mycat start 启动
./mycat stop 停止
./mycat console 前台运行
./mycat install 添加到系统自动启动
./mycat remove 取消系统自动启动
./mycat restart 重启服务
./mycat pause 暂停
./mycat status 查看启动状态

linux
./mycat install
./mycat start
./mycat status

win
mycat install
mycat start
mycat status
```

`mysql`登录与`mycat`登录有点类似
```
mysql登录命令
mysql -umysql -p123456 -P3306

mycat登录命令，9066为mycat管理窗口端口用于管理维护mycat；8066为mycat数据查询端口用于通过mycat查询数据
mysql -umycat -p123456 -P8066
```

### 3. Mycat概念
- 分库分表：按照一定规则把数据库中的表拆分成多个带有数据库实例、物理库、物理表访问路径的分表
	分库：一个电商项目，分开有用户库、订单库，原本一个数据库中存在用户表与订单表，现在根据业务拆分开来
	分表：一张订单表数据百万，分到多个数据库中的多张表
- 逻辑库：数据库代理中的数据库，可以包含多个逻辑表
	`Mycat`中定义的库，在逻辑上存在，物理上不存在`mysql`中，有可能是多个`mysql`库共同组成一个逻辑库
- 逻辑表：数据库代理中的表，可以映射代理连接的数据库中的物理表
	`Mycat`中定义的表，在逻辑上存在，可以映射为真实的`mysql`库中的表，可以一对一，一对多
- 物理库：数据库代理连接的数据库中真实的库
- 物理表：数据库代理连接的数据库中真实的表
- 拆分键：即分片键，描述拆分逻辑表的数据规则的字段
	如订单表可以按照订单所属的用户`ID`拆分订单表，用户`ID`则为拆分键
- 物理分表：已经进行数据拆分，数据库上的物理表，是分片表的一个分区，多个物理分表汇总就是一个逻辑表
- 物理分库：包含物理分表的库，参与了表数据分片的实际数据库
- 分库：通过多个数据库拆分出分片表
- 分片表：存在水平分片与垂直分片
- 单表：没有存在拆分的表
- 全局表、广播表：所有的分片数据源中都存在的表，表结构和表中的数据在每个数据库中完全一致
- ER表
	狭义：父子表中的子表，分片键指向父表的分片键，而且两表分片算法相同
	广义：具有相同的数据分布的一组表
- 集群：多个数据节点组成的逻辑节点
- 数据源
- 原型库：`Mycat`背后所连接的真实`mysql`数据库

### 4. 配置文件
1. 用户：`mycat2\conf\users\${username}.user.json`，用来配置`mycat`登录用户
```json
{
	"dialect":"mysql", //数据库类型
	"ip":null, //客户端ip白名单，对客户端访问进行限制
	"password":"56962438", //mycat用户密码，与mysql的用户密码无关
	"transactionType":"xa", //事务类型
	"username":"root", //mycat用户名，与mysql的用户密码无关
	"isolation":3 //设置初始化的事务隔离级别
	/*
		read_uncommitted: 1
		read_committed: 2
		repeated_read: 3 默认
		serializable: 4
	*/
}
```
2. 数据源：`mycat2\conf\datasources\${datasourcename}.datasource.json`，用来配置`mycat`连接后端物理库的数据源
```json
{
	"dbType":"mysql", //数据源类型
	"idleTimeout":60000,
	"initSqls":[],
	"initSqlsGetConnection":true,
	"instanceType":"READ_WRITE",
	"maxCon":1000,
	"maxConnectTimeout":3000,
	"maxRetryCount":5,
	"minCon":1,
	"name":"prototypeDs", //数据源名称
	"password":"123456", //连接Mysql库的用户密码
	"type":"JDBC",
	"url":"jdbc:mysql://localhost:3306/mysql?useUnicode=true&serverTimezone=Asia/Shanghai&characterEncoding=UTF-8",
	"user":"root", //连接Mysql库的用户名称
	"weight":0 //数据源负载均衡使用权重
}
```
3. 集群：`mycat2\conf\clusters\${集群名称}.cluster.json`，用来配置集群信息
```json
{
	"clusterType":"MASTER_SLAVE", //集群类型，single_node单一集群、master_slave主从集群等
	"heartbeat":{
		"heartbeatTimeout":1000,
		"maxRetry":3,
		"minSwitchTimeInterval":300,
		"slaveThreshold":0
	},
	"masters":[
		"prototypeDs"
	],
	"maxCon":200,
	"name":"prototype",
	"readBalanceType":"BALANCE_ALL", //查询负载均衡策略
	"switchType":"SWITCH" //主从故障转移
}
```
4. 逻辑库、逻辑表：`mycat2\conf\schemas\${库名}.schema.json`，用来配置`mycat`与`mysql`对应的逻辑表，进行分库分表
```json
{
	"customTables":{}, //自定义表
	"globalTables":{}, //全局表
	"normalTables":{...}, //默认表
	"schemaName":"mysql", //逻辑库名
	"shardingTables":{}, //分片表
	"targetName":"prototype" //目的数据源名，也可以为目的集群名
}
```
5. 序列号：`mycat2\conf\sequences\${数据库名字}_${表名字}.sequence.json`，使用序列号的分片表，对自增主键要在建表SQL中体现
6. 服务：`mycat2\server.json`

### 5. Mycat读写分离
1. 先实现`Mysql`主从复制配置
```
主数据库配置
server-id=1
log_bin=mysql-bin 打开日志
#binlog-do-db=czc 这个是需要同步的数据库 ，czc是一个数据库，自行先创建，不加则同步所有库
binlog-ignore-db=mysql 不给从机同步的数据库
binlog-ignore-db=performance_schema 不给从机同步的数据库
binlog-ignore-db=information_schema 不给从机同步的数据库
expire_logs_days=2 自动清理两天前的log文件
执行show master status，可以查看到file以及position的值，这两个值随后的从服务器会使用到
还需要在主机上创建用户并授权从机，让从机可以使用该用户复制binlog
create user 'slave1'@'%' identified by '123456';
grant replication slave on *.* to 'slave1'@'%';
alter user 'slave1'@'%' identified with mysql_native_password by '123456';
flush privileges;
执行show master status查看主机状态


从数据库配置1
当mysql版本大于5.5
server-id=2
master-host=192.168.1.1  主数据库的ip
master-user=slave1       主机中创建授权账号的用户名
master-password=123456   主机中创建授权账号的密码
master-port=3306
master-connect-retry=60
#replicate-do-db=czc    要同步的数据库,要同步多个数据库，就多加几个replicate-db-db=数据库名，不加则同步所有库
从数据库配置2
当mysql版本小于5.5，不能使用修改配置文件的方式直接配置，只能使用命令行的方式配置
配置文件中添加 server-id=3
执行以下change命令
change master to master_=host='118.25.2437.342',master_port=3306,master_user='slave1',master_password='123456',master_log_file='mysql-bin.000015',master_log_pos=606;
change master to这条命令中的master_log_file与master_log_pos可在主机的show mater status查看
最后从mysql服务器执行start slave

执行show slave status查看从机状态，可以查看到slave_io_running与slave_sql_running，都为yes则算是配置成功
```

2. 使用命令`mysql -uroot -p56962438 -P8066`远程连接上`mycat`后，向`mycat`添加数据源
```
/*+ mycat:createDataSource{
	"dbType":"mysql",
	"idleTimeout":60000,
	"initSqls":[],
	"initSqlsGetConnection":true,
	"instanceType":"READ_WRITE",
	"maxCon":1000,
	"maxConnectTimeout":3000,
	"maxRetryCount":5,
	"minCon":1,
	"name":"mycat_ds1",
	"password":"56962438",
	"type":"JDBC",
	"url":"jdbc:mysql://10.0.2.15:3306/master_slave_db?useUnicode=true&serverTimezone=Asia/Shanghai&characterEncoding=UTF-8",
	"user":"root",
	"weight":0
}*/;

/*+ mycat:createDataSource{
	"dbType":"mysql",
	"idleTimeout":60000,
	"initSqls":[],
	"initSqlsGetConnection":true,
	"instanceType":"READ",
	"maxCon":1000,
	"maxConnectTimeout":3000,
	"maxRetryCount":5,
	"minCon":1,
	"name":"mycat_ds2",
	"password":"56962438",
	"type":"JDBC",
	"url":"jdbc:mysql://10.0.2.4:3306/master_slave_db?useUnicode=true&serverTimezone=Asia/Shanghai&characterEncoding=UTF-8",
	"user":"root",
	"weight":0
}*/;

/*+ mycat:createDataSource{
	"dbType":"mysql",
	"idleTimeout":60000,
	"initSqls":[],
	"initSqlsGetConnection":true,
	"instanceType":"READ",
	"maxCon":1000,
	"maxConnectTimeout":3000,
	"maxRetryCount":5,
	"minCon":1,
	"name":"mycat_ds3",
	"password":"56962438",
	"type":"JDBC",
	"url":"jdbc:mysql://10.0.2.5:3306/master_slave_db?useUnicode=true&serverTimezone=Asia/Shanghai&characterEncoding=UTF-8",
	"user":"root",
	"weight":0
}*/;
```

3. 可以使用命令`/*+ mycat:showDataSources{} */`查看`mycat`中的数据源
4. 创建集群
```
/*! mycat:createCluster{
	"clusterType":"MASTER_SLAVE",
	"heartbeat":{
		"heartbeatTimeout":1000,
		"maxRetry":3,
		"minSwitchTimeInterval":300,
		"slaveThreshold":0
	},
	"masters":[
		"mycat_ds1"
	],
	"maxCon":2000,
	"name":"prototype",
	"readBalanceType":"BALANCE_ALL",
	"replicas":[
		"mycat_ds2",
		"mycat_ds3"
	],
	"switchType":"SWITCH"
}*/;
```
5. 查看集群`/*+ mycat:showClusters{}*/`
6. 创建逻辑库代表上述集群`create database mycat_logic_db default character set utf8mb4 collate utf8mb4_general_ci`;，之后更改`mycat/schemas/mycat_logic_db.schema.json`，向其中添加`"targetName":"集群name"`
7. 重启`mycat`

**数据源代表连接具体IP与端口的Mysql机器，若干数据源组成一个集群，再之后创建的逻辑库会根据schema配置文件中的targetName指向集群，并在集群中创建与逻辑库对应的物理库。因此对逻辑库作操作就是对集群中对应的物理库作操作**

### 6. Mycat双主双从
<img src="D:\Project\IT notes\框架or中间件\Mycat2\img\Mycat双主双从架构图.png" style="width:700px;height:300px;" />

| 编号 | 角色 | IP地址 | 机器名 |
| ----- | ----- | ----- | ----- |
| 1 | `master1` | `192.168.200.132` | `mycat01` |
| 2 | `slave1` | `192.168.200.133` | `mycat02` |
| 3 | `master2` | `192.168.200.134` | `mycat03` |
| 4 | `slave2` | `192.168.200.135` | `mycat04` |

1. `master1`配置
```
[mysqld]
datadir=/home/lao/mysql8
socket=/home/lao/mysql8/mysql.sock

log-error=/home/lao/mysql8/log/mysqld.log
pid-file=/home/lao/mysql8/mysqld.pid
[必须] 主服务器唯一ID
server-id=1
[必须]启用二进制日志，指明路径。比如：自己本地的路径 /log/mysqlbin
log-bin=/home/lao/mysql8/mysql-bin
设置不要复制的数据库(可设置多个)
binlog-ignore-db=mysql
binlog-ignore-db=information_schema
设置需要复制的数据库
binlog-do-db=mydb1
设置logbin格式
binlog_format=STATEMENT
在作为从数据库的时候，有写入操作也要更新二进制日志文件
log-slave-updates=1
表示自增长字段每次递增的量，指自增字段的起始值，其默认值是1，取值范围是1 .. 65535
auto-increment-increment=2
表示自增长字段从哪个数开始，指字段一次递增多少，他的取值范围是1 .. 65535
auto-increment-offset=1
```
2. `master2`配置
```
[mysqld]
datadir=/home/lao/mysql8
socket=/home/lao/mysql8/mysql.sock

log-error=/home/lao/mysql8/log/mysqld.log
pid-file=/home/lao/mysql8/mysqld.pid
[必须] 主服务器唯一ID
server-id=3
[必须]启用二进制日志，指明路径。比如：自己本地的路径 /log/mysqlbin
log-bin=/home/lao/mysql8/mysql-bin
设置不要复制的数据库(可设置多个)
binlog-ignore-db=mysql
binlog-ignore-db=information_schema
设置需要复制的数据库
binlog-do-db=mydb1
设置logbin格式
binlog_format=STATEMENT
在作为从数据库的时候，有写入操作也要更新二进制日志文件
log-slave-updates=1
表示自增长字段每次递增的量，指自增字段的起始值，其默认值是1，取值范围是1 .. 65535
auto-increment-increment=2
表示自增长字段从哪个数开始，指字段一次递增多少，他的取值范围是1 .. 65535
auto-increment-offset=2
```
3. `slave1`配置
```
[mysqld]
datadir=/home/lao/mysql8
socket=/home/lao/mysql8/mysql.sock

log-error=/home/lao/mysql8/log/mysqld.log
pid-file=/home/lao/mysql8/mysqld.pid
[必须] 主服务器唯一ID
server-id=2
启用中继日志
relay-log=mysql-relay
```
4. `slave2`配置
```
[mysqld]
datadir=/home/lao/mysql8
socket=/home/lao/mysql8/mysql.sock

log-error=/home/lao/mysql8/log/mysqld.log
pid-file=/home/lao/mysql8/mysqld.pid
[必须] 主服务器唯一ID
server-id=4
启用中继日志
relay-log=mysql-relay
```
5. 重启所有`mysql`服务，关闭防火墙，并在所有主机上配置`slave`用户
```
create user 'slave2'@'%' identified by '56962438';
grant replicatioin slave on *.* to 'slave2'@'%';
alter user 'slave2'@'%' identified with mysql_native_password by '56962438';
```
6. 在所有从机上设置需要复制的主机
```
slave1
CHANGE MASTER TO MASTER_HOST='192.168.200.132',
MASTER_USER='slave2',
MASTER_PASSWORD='123123',
MASTER_LOG_FILE='mysql-bin.000008',MASTER_LOG_POS=157;

slave2
CHANGE MASTER TO MASTER_HOST='192.168.200.134',
MASTER_USER='slave2',
MASTER_PASSWORD='123123',
MASTER_LOG_FILE='mysql-bin.000001',MASTER_LOG_POS=157;

slave1与slave2上分别执行start slave与show slave status
```
7. 设置两个主机互为主从
```
master1
CHANGE MASTER TO MASTER_HOST='192.168.200.134',
MASTER_USER='slave2',
MASTER_PASSWORD='123123',
MASTER_LOG_FILE='mysql-bin.000001',MASTER_LOG_POS=157;

master2
CHANGE MASTER TO MASTER_HOST='192.168.200.132',
MASTER_USER='slave2',
MASTER_PASSWORD='123123',
MASTER_LOG_FILE='mysql-bin.000008',MASTER_LOG_POS=157;

master1与master2分别执行start slave与show slave status
```
8. `mycat2`配置
为`master1`、`master2`、`slave1`、`slave2`四台主从机创建`mycat`数据源`dataSource`，并把这四个数据源添加入同一个集群`cluster`，之后创建一个逻辑库`schema`并将逻辑库与集群构成映射关系
9. 在逻辑库与逻辑表上操作，就是在逻辑库表对应的物理库表集群操作

### 7. 分库分表
- 分库（原则：将紧密关联关系的表划分在一个库里，没有关联关系的表可以分到不同的库里）
	- 水平分库：把同一个表的数据按一定规则拆到不同的数据库中
	- 垂直分库：按照业务、功能模块将表进行分类，不同功能模块对应的表分到不同的库中
- 分表（原则：减少节点数据库的访问，分表字段尤为重要，其决定了节点数据库的访问量）
	- 水平分表：在同一个数据库内，把同一个表的数据按一定规则拆到多个表中
	- 垂直分表：将一个表按照字段分成多表，每个表存储其中一部分字段

**切分的主要目的，都是为了减少数据库存储压力与访问压力**

**切记：默认的自动分片机制（包括广播表）要求设置的集群名字，必须以c为前缀数字为后缀，分片后写入操作均为写入分片集群的主机中去，不写从机**

*以下分片表均在已经创建集群c0、c1但集群并未在`schema`配置文件设置映射逻辑库的前提下*
#### 1. 广播表
1. 创建成功逻辑库`create database mydb`，此时无需将逻辑库与集群节点进行映射，即schema配置文件映射
2. 创建广播表，物理上会在所有集群节点都创建广播表
```sql
create table `mydb`.`broadcast_tb` (
	`id` bigint not null auto_increment,
	`name` varchar(50) default null,
	primary key (`id`),
	key `id` (`id`)
) engine=InnoDB default charset=utf8 broadcast;
```
3. 查看`mycat schema`中的配置
```json
{
	"customTables":{},
	"globalTables":{
		"team":{
			"broadcast":[
				{"targetName":"c0"},
				{"targetName":"c1"}
			],
			"createTableSQL":"create table mydb.`broadcast_tb` (...)..."
		}
	},
	"normalProcedures":{},
	"normalTables":{},
	"schemaName":"mydb",
	"shardingTables":{},
	"views":{}
}
```

#### 2. 分片表
创建分片表
```sql
create table `mydb`.`orders` (
	`id` bigint not null auto_increment,
	`order_type` int...,
	`customer_id` int...,
	`amount` decimal(10.2)...,
	primary key (`id`),
	key `id` (`id`)
) engine=InnoDB default charset=utf8 
	dbpartition by mod_hash(customer_id)
	tbpartition by mod_hash(customer_id)
	tbpartitions 1 dbpartitions 2;
```
配置信息修改可查看`schema`配置文件

#### 3. ER表
`mycat2`在涉及这两个表的join分片字段等价关系的时候可以完成join的下推，如`order`表与`order_detail`表，`order`表的`id`字段与`order_detail`表的`order_id`关联

`mycat1`需要自己创建`ER`表；`mycat2`在表与表关联时，无需自行创建`ER`关系支持关联。在同一个`schema`中所有的表都存在`ER`关系，即互相关联的两表，即使物理数据不在同一个库中，但只要两表的逻辑表都在同一个逻辑库中，则支持互相关联

#### 4. 分片算法
<img src="D:\Project\IT notes\框架or中间件\Mycat2\img\分片算法.png" style="width:700px;height:600px;" />


### 8. 全局ID，snowflake雪花ID算法
在复杂的分布式系统中，需要对大量的数据和消息进行唯一标识。如在阿里，淘宝，支付，等系统中，数据日渐增长，对数据分库分表后需要有一个唯一`ID`来标识一条数据或消息，还有如美团和饿了吗的骑手`ID`商家`ID`优惠券`ID`等，从以上可以得出，一个能够生成全局唯一`ID`的系统是非常必要的

在`MyCAT2`中，自动默认使用雪花片法生成全局序列号

如果不需要`MyCAT`默认全局序列，可以通过配置关闭自动加全局序列；建表语句方式关闭全局序列。如果不需要使用`MyCAT`的自增序列，而使用`MySQL`本身的自增主键的功能。需要在配置中更改对应的建表`SQL`。不设置`auto increment`关键字，这样`MyCAT`就不认为这个表有自增主键的功能。就不会使用`MyCAT`全局序列号，这样，对应的插入`SQL`在`MySQL`中去处理，由`MySQL`的自增主键功能补全

<img src="D:\Project\IT notes\框架or中间件\Mycat2\img\雪花算法.png" style="width:700px;height:250px;" />

### 9. Mycat2 UI客户端工具 assistant.jar

