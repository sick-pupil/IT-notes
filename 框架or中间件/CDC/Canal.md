## 1. canal-admin
1. 集群管理：就是我们集群下的`canal-server`的公共配置(`canal.properties`) 由以前各个服务器配配置改为数据库配置和`zk`地址,修改配置通知集群下的所有`server`
2. `Server`管理：注册到`Admin`的所有`Server`(针对单机可以配置各个`server`的`canal.properties`)
3. `instance`管理：对应以前的`example`下的`instance`配置，不同的`client`可以配置订阅不同的`instance`配置。不同的`instance`下维护各自的`binlog`读取的指针位置 和指定库如多个,号隔开需要像`example`一样创建对应的目录`canal.properties下的canal.destinations = example`

1. `conf/canal_manager.sql`创建`canal_manager`数据库
`mysql -u mantishell -p -h 127.0.0.1 -P 3506 my < ./canal_manager.sql`
2. `conf/application.yml`修改配置
```yml
server:
  port: 8089
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8

spring.datasource:
  address: 127.0.0.1:3306 # admin数据库地址
  database: canal_manager # 数据库名称
  username: mantishell # 数据库账号
  password: 123456     # 数据库密码
  driver-class-name: com.mysql.jdbc.Driver
  url: jdbc:mysql://${spring.datasource.address}/${spring.datasource.database}?useUnicode=true&characterEncoding=UTF-8&useSSL=false
  hikari:
    maximum-pool-size: 30
    minimum-idle: 1

canal:
  adminUser: admin   # 网页登录的账号
  adminPasswd: 123456 # 网页登录的默认密码
```
3. 等待`canal-deployer`配置并部署好后，进入`canal-admin`，位于菜单“集群管理”，按照`canal-deployer`中配置的集群信息，配置`canal-deployer`集群并启动
4. `canal-admin`中选择菜单"操作-主配置-载入模板-保存"
```
#################################################
######### 		common argument		#############
#################################################
# tcp bind ip
canal.ip =
# register ip to zookeeper
canal.register.ip = 
canal.port = 11111
canal.metrics.pull.port = 11112
# canal instance user/passwd
# canal.user = canal
# canal.passwd = E3619321C1A937C46A0D8BD1DAC39F93B27D4458

# canal admin config 
canal.admin.manager = 127.0.0.1:8089
canal.admin.port = 11110
canal.admin.user = admin
canal.admin.passwd = 4ACFE3202A5FF5CF467898FC58AAB1D615029441
# admin auto register
#canal.admin.register.auto = true
#canal.admin.register.cluster =
#canal.admin.register.name =

canal.zkServers =
# flush data to zk
canal.zookeeper.flush.period = 1000
canal.withoutNetty = false
# tcp, kafka, rocketMQ, rabbitMQ
canal.serverMode = tcp
# flush meta cursor/parse position to file
canal.file.data.dir = ${canal.conf.dir}
canal.file.flush.period = 1000
## memory store RingBuffer size, should be Math.pow(2,n)
canal.instance.memory.buffer.size = 16384
## memory store RingBuffer used memory unit size , default 1kb
canal.instance.memory.buffer.memunit = 1024 
## meory store gets mode used MEMSIZE or ITEMSIZE
canal.instance.memory.batch.mode = MEMSIZE
canal.instance.memory.rawEntry = true

## detecing config
canal.instance.detecting.enable = false
#canal.instance.detecting.sql = insert into retl.xdual values(1,now()) on duplicate key update x=now()
canal.instance.detecting.sql = select 1
canal.instance.detecting.interval.time = 3
canal.instance.detecting.retry.threshold = 3
canal.instance.detecting.heartbeatHaEnable = false

# support maximum transaction size, more than the size of the transaction will be cut into multiple transactions delivery
canal.instance.transaction.size =  1024
# mysql fallback connected to new master should fallback times
canal.instance.fallbackIntervalInSeconds = 60

# network config
canal.instance.network.receiveBufferSize = 16384
canal.instance.network.sendBufferSize = 16384
canal.instance.network.soTimeout = 30

# binlog filter config
canal.instance.filter.druid.ddl = true
canal.instance.filter.query.dcl = false
canal.instance.filter.query.dml = false
canal.instance.filter.query.ddl = false
canal.instance.filter.table.error = false
canal.instance.filter.rows = false
canal.instance.filter.transaction.entry = false
canal.instance.filter.dml.insert = false
canal.instance.filter.dml.update = false
canal.instance.filter.dml.delete = false

# binlog format/image check
canal.instance.binlog.format = ROW,STATEMENT,MIXED 
canal.instance.binlog.image = FULL,MINIMAL,NOBLOB

# binlog ddl isolation
canal.instance.get.ddl.isolation = false

# parallel parser config
canal.instance.parser.parallel = true
## concurrent thread number, default 60% available processors, suggest not to exceed Runtime.getRuntime().availableProcessors()
#canal.instance.parser.parallelThreadSize = 16
## disruptor ringbuffer size, must be power of 2
canal.instance.parser.parallelBufferSize = 256

# table meta tsdb info
canal.instance.tsdb.enable = true
canal.instance.tsdb.dir = ${canal.file.data.dir:../conf}/${canal.instance.destination:}
canal.instance.tsdb.url = jdbc:h2:${canal.instance.tsdb.dir}/h2;CACHE_SIZE=1000;MODE=MYSQL;
canal.instance.tsdb.dbUsername = canal
canal.instance.tsdb.dbPassword = canal
# dump snapshot interval, default 24 hour
canal.instance.tsdb.snapshot.interval = 24
# purge snapshot expire , default 360 hour(15 days)
canal.instance.tsdb.snapshot.expire = 360

#################################################
######### 		destinations		#############
#################################################
# 配置
canal.destinations = example
# conf root dir
canal.conf.dir = ../conf
# auto scan instance dir add/remove and start/stop instance
canal.auto.scan = true
canal.auto.scan.interval = 5
# set this value to 'true' means that when binlog pos not found, skip to latest.
# WARN: pls keep 'false' in production env, or if you know what you want.
canal.auto.reset.latest.pos.mode = false

canal.instance.tsdb.spring.xml = classpath:spring/tsdb/h2-tsdb.xml
#canal.instance.tsdb.spring.xml = classpath:spring/tsdb/mysql-tsdb.xml

canal.instance.global.mode = manager
canal.instance.global.lazy = false
canal.instance.global.manager.address = ${canal.admin.manager}
#canal.instance.global.spring.xml = classpath:spring/memory-instance.xml
canal.instance.global.spring.xml = classpath:spring/file-instance.xml
#canal.instance.global.spring.xml = classpath:spring/default-instance.xml

##################################################
######### 	      MQ Properties      #############
##################################################
# aliyun ak/sk , support rds/mq
canal.aliyun.accessKey =
canal.aliyun.secretKey =
canal.aliyun.uid=

canal.mq.flatMessage = false
canal.mq.canalBatchSize = 50
canal.mq.canalGetTimeout = 100
# Set this value to "cloud", if you want open message trace feature in aliyun.
canal.mq.accessChannel = local

canal.mq.database.hash = false
canal.mq.send.thread.size = 30
canal.mq.build.thread.size = 8

##################################################
######### 		     Kafka 		     #############
##################################################
kafka.bootstrap.servers = 127.0.0.1:6667
kafka.acks = all
kafka.compression.type = none
kafka.batch.size = 16384
kafka.linger.ms = 1
kafka.max.request.size = 1048576
kafka.buffer.memory = 33554432
kafka.max.in.flight.requests.per.connection = 1
kafka.retries = 0

kafka.kerberos.enable = false
kafka.kerberos.krb5.file = "../conf/kerberos/krb5.conf"
kafka.kerberos.jaas.file = "../conf/kerberos/jaas.conf"

##################################################
######### 		    RocketMQ	     #############
##################################################
rocketmq.producer.group = test
rocketmq.enable.message.trace = false
rocketmq.customized.trace.topic =
rocketmq.namespace =
rocketmq.namesrv.addr = 127.0.0.1:9876
rocketmq.retry.times.when.send.failed = 0
rocketmq.vip.channel.enabled = false
rocketmq.tag = 

##################################################
######### 		    RabbitMQ	     #############
##################################################
rabbitmq.host =
rabbitmq.virtual.host =
rabbitmq.exchange =
rabbitmq.username =
rabbitmq.password =
rabbitmq.deliveryMode =
```
5. 新增`instance`，并设置源库同步参数
```
# MySQL 集群配置中的 serverId 概念，需要保证和当前 MySQL 集群中 id 唯一 (v1.0.26+版本之后 canal 会自动生成，不需要手工指定) 
# canal.instance.mysql.slaveId=0 
# 源库地址 
canal.instance.master.address=11.17.6.185:4679 

# 方式一：binlog + postion 
# mysql源库起始的binlog文件 
# canal.instance.master.journal.name=mysql-bin.000008 
# mysql主库链接时起始的binlog偏移量 
# canal.instance.master.position=257890708 
# mysql主库链接时起始的binlog的时间戳 
# canal.instance.master.timestamp= 

# 方式二：gtid(推荐) 
# 启用 gtid 方式同步 
canal.instance.gtidon=true 
# gtid 
canal.instance.master.gtid=92572381-0b9c-11ec-9544-8cdcd4b157e0:1-1066141 

# 源库账号密码 
canal.instance.dbUsername=canal 
canal.instance.dbPassword=canal 

# MySQL 数据解析编码 
canal.instance.connectionCharset = UTF-8 

# MySQL 数据解析关注的表，Perl 正则表达式
canal.instance.filter.regex=acpcanaldb.* 
# MySQL 数据解析表的黑名单，过滤掉不同步的表 
# canal.instance.filter.black.regex=mysql\\.slave_.*
```
## 2. canal-deployer
1. 修改`conf/canal_local.properties`
```prop
#Canal Server 地址 
canal.register.ip = 11.8.36.104 
#Canal Admin 连接信息 
canal.admin.manager = 11.8.36.104:8089 
canal.admin.port = 11110 
canal.admin.user = admin 
#mysql5 类型 MD5 加密结果 -- admin 
canal.admin.passwd = 4ACFE3202A5FF5CF467898FC58AAB1D615029441 
#自动注册 
canal.admin.register.auto = true 
#集群名 
canal.admin.register.cluster = canal-cluster-1 
#Canal Server 名字 
canal.admin.register.name = canal-server-1



#Canal Server 地址 
canal.register.ip = 11.8.36.105
#Canal Admin 连接信息 
canal.admin.manager = 11.8.36.104:8089 
canal.admin.port = 11110 
canal.admin.user = admin 
#mysql5 类型 MD5 加密结果 -- admin 
canal.admin.passwd = 4ACFE3202A5FF5CF467898FC58AAB1D615029441 
#自动注册 
canal.admin.register.auto = true 
#集群名 
canal.admin.register.cluster = canal-cluster-1 
#Canal Server 名字 
canal.admin.register.name = canal-server-2
```
2. 启动`canal-deployer`，`./bin/startup.sh local`
## 3. canal-adapter
1. 修改`bootstrap.yml`配置
```yml
canal:
  manager:
    jdbc:
      url: jdbc:mysql://127.0.0.1:3506/canal_manager?useUnicode=true&characterEncoding=UTF-8&useSSL=false&allowPublicKeyRetrieval=true
      username: mantishell
      password: 123456

```
2. 修改`application.yml`配置
```yml
  srcDataSources:
    defaultDS:
      url: jdbc:mysql://127.0.0.1:3506/src_db?useUnicode=true&characterEncoding=UTF-8&useSSL=false&allowPublicKeyRetrieval=true
      username: root
      password: 123456
  canalAdapters:
  - instance: example # canal instance Name or mq topic name
    groups:
    - groupId: g1
      outerAdapters:
      - name: logger
      - name: rdb
        # key的值需要和rdb里yml文件里outerAdapterKey的值相同
        key: mysql1
        properties:
          jdbc.driverClassName: com.mysql.cj.jdbc.Driver
          # 目的地址
          jdbc.url: jdbc:mysql://192.168.10.10:3506/dest_db?useUnicode=true&characterEncoding=UTF-8&useSSL=false&allowPublicKeyRetrieval=true
          jdbc.username: root
          jdbc.password: 123456
```
3. 修改`rdb/*.yml`配置
```yml
dataSourceKey: defaultDS
destination: example
groupId: g1
outerAdapterKey: mysql1
concurrent: true
dbMapping:
  database: src_db
  table: user
  targetTable: dest_db.user
  targetPk:
    id: id
  mapAll: true
#  targetColumns:
#    id:
#    name:
#    role_id:
#    c_time:
#    test1:
#  etlCondition: "where c_time>={}"
  commitBatch: 3000 # 批量提交的大小
```