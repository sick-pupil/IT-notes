<img src="D:\Project\IT-notes\框架or中间件\Redis\img\基础.png" style="width:700px;height:600px;" />

## 1. 启用Redis服务
将`redis`服务注册到`windows`服务中：`redis-server.exe --service-install redis.windows.conf --loglevel verbose`
启动`redis`服务：`redis-server --service-start`
关掉`redis`服务：`redis-server --service-stop`

也可以将`redis`安装目录添加至用户环境变量或者系统环境变量，直接使用`redis-server.exe`命令启动`redis`
打开`redis-cli.exe`使用`ping`命令查看返回是否为`pong`，确定客户端与服务端是否成功连接

## 2. 常用数据类型
### 1. string
`string`字符串为一组字节，具有**二进制安全**特性：只关心二进制化的字符串，不关心具体的字符串格式，严格的按照二进制的数据存取
`redis`中的字符串类型是长度已知的，最多存储512MB的内容
```
使用set、get设置以及获取一个字符串类型键值对
set website "www.baidu.com"
get website

使用mset、mget设置以及获取多个字符串类型键值对
mset name www.baidu.com topic searchEngine
mget name topic
```

`Redis String`底层自定义了一个SDS动态字符串结构
```C
struct sdshdr{
     //记录buf数组中已使用字符的数量，等于 SDS 保存字符串的长度
     int len;
     //记录 buf 数组中未使用的字符数量
     int free;
     //字符数组，用于保存字符串
     char buf[];
```
此外`string`类型还会采用预先分配冗余空间的方式减少内存的频繁分配，初次分配的空间要大于字符串实际占用空间，刚开始字符串变大则无需申请内存空间
当空间不够时则会自动扩容，当字符串所占空间小于1MB，扩容是以成倍的方式增加；当所占空间超过1MB，每次扩容则只增加1MB

`string`命令格式
`SET key value [EX seconds|PX milliseconds] [NX|XX]`
例：`SET www.biancheng.net "hello编程帮" EX 60 NX`
- `EX seconds`：设置指定的过期时间，以秒为单位
- `PX milliseconds`：设置指定的过期时间，以毫秒为单位
- `NX`：先判断`key`是否存在，如果`key`不存在，则设置`key`与`value`
- `XX`：先判断`key`是否存在，如果`key`存在，则重新设置`value`

| 命令 | 说明 |
| ----- | ----- |
| set key value | 用于设定指定键的值 |
| get key | 用于检索指定键的值 |
| getrange key start end | 返回 key 中字符串值的子字符 |
| getset key value | 将给定 key 的值设置为 value，并返回 key 的旧值 |
| getbit key offset | 对 key 所存储的字符串值，获取其指定偏移量上的位（bit） |
| mget key1 \[key2..\] | 批量获取一个或多个 key 所存储的值，减少网络耗时开销 |
| setbit key offset value | 对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit) |
| setex key seconds value | 将值 value 存储到 key中 ，并将 key 的过期时间设为 seconds (以秒为单位) |
| setnx key value | 当 key 不存在时设置 key 的值 |
| setrange key offset value | 从偏移量 offset 开始，使用指定的 value 覆盖的 key 所存储的部分字符串值 |
| strlen key | 返回 key 所储存的字符串值的长度 |
| mset key value \[key value...\] | 该命令允许同时设置多个键值对 |
| msetnx key value \[key value...\] | 当指定的 key 都不存在时，用于设置多个键值对 |
| psetex key milliseconds value | 此命令用于设置 key 的值和有过期时间（以毫秒为单位） |
| incr key | 将 key 所存储的整数值加 1 |
| incrby key increment | 将 key 所储存的值加上给定的递增值（increment） |
| incrbyfloat key increment | 将 key 所储存的值加上指定的浮点递增值（increment） |
| decr key | 将 key 所存储的整数值减 1 |
| decrby key decrement | 将 key 所储存的值减去给定的递减值（decrement） |
| append key value | 该命令将 value 追加到 key 所存储值的末尾 |

### 2. hash
`hash`散列是由字符串类型的`field`和`value`组成的映射表，可以理解为由一个散列表名称与表中多个键值对组成，一般`hash`类型用来存储对象
```
使用hmset创建一个对象
hmset user name xiaoming age 18 sex male

使用hgetall获取对应ID的hash的所有键值对
hgetall user
```

`redis hash`底层
- 当存储数据量少，使用`ziplist`作为底层存储结构，但必须满足
	1. 哈希对象保存的所有键值对（键和值）的字符串长度总和小于 64 个字节
	2. 哈希对象保存的键值对数量要小于 512 个
- 当存储数据量大或者无法满足上述条件，底层则采用`dict`字典结构存储，即一个无序字典，类似java中的hashmap，原理上就是一个**数组+链表相结合的散列表**

`hash`命令格式

| 命令 | 说明 |
| ----- | ----- |
| hdel key field2 \[field2\] | 用于删除一个或多个哈希表字段 |
| hexists key field | 用于确定哈希表字段是否存在 |
| hget key field | 获取 key 关联的哈希字段的值 |
| hgetall key | 获取 key 关联的所有哈希字段值 |
| hincrby key field increment | 给 key 关联的哈希字段做整数增量运算 |
| hincrbyfloat key field increment | 给 key 关联的哈希字段做浮点数增量运算 |
| hkeys key | 获取 key 关联的所有字段和值 |
| hlen key | 获取 key 中的哈希表的字段数量 |
| hmset key field1 value1 \[field2 value2\] | 在哈希表中同时设置多个 field-value(字段-值） |
| hmget key field1 \[field2\] | 用于同时获取多个给定哈希字段（field）对应的值 |
| hset key field value | 用于设置指定 key 的哈希表字段和值（field/value） |
| hsetnx key field value | 仅当字段 field 不存在时，设置哈希表字段的值 |
| hvals key | 用于获取哈希表中的所有值 |
| hscan key cursor | 迭代哈希表中的所有键值对，cursor 表示游标，默认为 0 |

### 3. list
`list`为有序可重复的字符串列表
`list`中的元素都为字符串类型，其中的元素按照插入顺序进行排列，允许重复元素出现
用户可以选择将元素添加到列表的头部或者尾部
```
使用lpush创建并向列表中添加元素
lpush class student1
lpush class student2
lpush class student3
lpush class student4
lpush class student5

使用lrange查看列表元素，最后一个插入元素排在最前，第一个插入的元素排在最后
lrange class 0 4
```

`redis list`底层使用了`ziplist + quicklist`的存储方式，数据量小则为`ziplist`，数据量大则改为`quicklist`
`ziplist`：压缩列表，列表中元素较少时会使用一块连续的内存来存储这些元素，即连续结构
`quicklist`：快速链表，由多个压缩列表组成

`list`这种数据类型可以通过控制元素如何从左侧右侧出入列表，实现队列和斩

`list`命令格式

| 命令 | 说明 |
| ----- | ----- |
| lpush key value1 \[value2\] | 在列表头部插入一个或者多个值 |
| lrange key start stop | 获取列表指定范围内的元素 |
| rpush key value1 \[value2\] | 在列表尾部添加一个或多个值 |
| lpushx key value | 当储存列表的 key 存在时，用于将值插入到列表头部 |
| rpushx key value | 当储存列表的 key 存在时，用于将值插入到列表尾部 |
| lindex key index | 通过索引获取列表中的元素 |
| linsert key before\|after pivot value | 指定列表中一个元素在它之前或之后插入另外一个元素 |
| lrem key count value | 表示从列表中删除元素与 value 相等的元素。count 表示删除的数量，为 0 表示全部移除 |
| lset key index value | 表示通过其索引设置列表中元素的值 |
| ltrim key start stop | 保留列表中指定范围内的元素值 |
| lpop key | 从列表的头部弹出元素，默认为第一个元素 |
| rpop key | 从列表的尾部弹出元素，默认为最后一个元素 |
| llen key | 用于获取列表的长度 |
| rpoplpush source destination | 用于删除列表中的最后一个元素，然后将该元素添加到另一个列表的头部，并返回该元素值 |
| blpop key1 \[key2...\] timeout | 用于删除并返回列表中的第一个元素（头部操作），如果列表中没有元素，就会发生阻塞，直到列表等待超时或发现可弹出元素为止 |
| brpop key1 \[key2...\] timeout | 用于删除并返回列表中的最后一个元素（尾部操作），如果列表中没有元素，就会发生阻塞， 直到列表等待超时或发现可弹出元素为止 |
| brpoplpush source destination timeout | 从列表中取出最后一个元素，并插入到另一个列表的头部。如果列表中没有元素，就会发生阻塞，直到等待超时或发现可弹出元素时为止 |

### 4. set
`set`为无序不可重复的字符串集合
`set`中的元素都为字符串类型且具有唯一性不重复，其中的元素通过哈希映射表实现，因此无论添加、删除、查找，时间复杂度都为O(1)
```
使用sadd命令添加string元素到set集合中，成功添加则返回1，若元素已经存在则返回0
sadd class stu1
sadd class stu2
sadd class stu3
sadd class stu4
sadd class stu5

使用smembers查看集合中的元素
smembers class
```

`redis set`底层采用了`intset`整型数组与`hash table`哈希表
- 当`set`中存储的数据满足集合内保存的所有成员都为整数值，集合内成员数量不超过512个时，底层使用`intset`整型数组
- 不满足上述条件则使用`hash table`结构

`set`命令格式

| 命令 | 说明 |
| ----- | ----- |
| sadd key member1 \[member2\] | 向集合中添加一个或者多个元素，并且自动去重 |
| scard key | 返回集合中元素的个数 |
| sdiff key1 \[key2\] | 求两个或多个集合的差集 |
| sdiffstore destination key1 \[key2\] | 求两个集合或多个集合的差集，并将结果保存到指定的集合中 |
| sinter key1 \[key2\] | 求两个或多个集合的交集 |
| sinterstore destination key1 \[key2\] | 求两个或多个集合的交集，并将结果保存到指定的集合中 |
| sismember key member | 查看指定元素是否存在于集合中 |
| smembers key | 查看集合中所有元素 |
| smove source destination member | 将集合中的元素移动到指定的集合中 |
| spop key \[count\] | 弹出指定数量的元素 |
| srandmember key \[count\] | 随机从集合中返回指定数量的元素，默认返回 1个 |
| srem key member1 \[member2\] | 删除一个或者多个元素，若元素不存在则自动忽略 |
| sunion key1 \[key2\] | 求两个或者多个集合的并集 |
| sunionstore destination key1 \[key2\] | 求两个或者多个集合的并集，并将结果保存到指定的集合中 |
| sscan key cursor \[match pattern\] \[count count\] | 该命令用来迭代的集合中的元素 |

### 5. zset
`zset`为有序不可重复的字符串集合
`zset`中的元素都为字符串类型且具有唯一性不重复，其中的元素通过哈希映射表实现，因此无论添加、删除、查找，时间复杂度都为O(1)
与`set`类型的最大区别就是支持集合中的元素有序，以及每个元素还会关联一个`double`类型的分数`score`，该分数允许重复，`zset`按照这个`score`排序
```
zadd命令添加元素到集合，若元素存在于集合中，则不能添加成功
zadd class 99.9 stu1
zadd class 99.8 stu2
zadd class 99.7 stu3
zadd class 99.6 stu4
zadd class 99.5 stu5

查看指定成员的分值
zscore class stu3
zrange class 0 4
```

`redis zset`底层使用了两种不同的存储结构，`ziplist`压缩列表和`skiplist`跳跃列表
- 当`zset`中成员数量小于128且每个`member`字符串长度小于64字节，则使用`ziplist`存储
- 不满足则使用`skiplist`存储

`zset`命令格式

| 命令 | 说明 |
| ----- | ----- |
| zadd key score1 member1 \[score2 member2\] | 用于将一个或多个成员添加到有序集合中，或者更新已存在成员的 score 值 |
| zcard key | 获取有序集合中成员的数量 |
| zcount key min max | 用于统计有序集合中指定 score 值范围内的元素个数 |
| zincrby key increment member | 用于增加有序集合中成员的分值 |
| zinterstore destination numkeys key \[key ...\] | 求两个或者多个有序集合的交集，并将所得结果存储在新的 key 中 |
| zlexcount key min max | 当成员分数相同时，计算有序集合中在指定词典范围内的成员的数量 |
| zrange key start stop \[withscores\] | 返回有序集合中指定索引区间内的成员数量 |
| zrangebylex key min max \[limit offset count\] | 返回有序集中指定字典区间内的成员数量 |
| zrangebyscore key min max \[withscores\] \[limit\] | 返回有序集合中指定分数区间内的成员 |
| zrank key member | 返回有序集合中指定成员的排名 |
| zrem key member \[member...\] | 移除有序集合中的一个或多个成员 |
| zremrangebylex key min max | 移除有序集合中指定字典区间的所有成员 |
| zremrangebyrank key start stop | 移除有序集合中指定排名区间内的所有成员 |
| zremrangebyscore key min max | 移除有序集合中指定分数区间内的所有成员 |
| zrevrange key start stop \[withscores\] | 返回有序集中指定区间内的成员，通过索引，分数从高到低 |
| zrevrangebyscore key max min \[withscores\] | 返回有序集中指定分数区间内的成员，分数从高到低排序 |
| zrevrank key member | 返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序 |
| zscore key member | 返回有序集中，指定成员的分数值 |
| zunionstore destination numkeys key \[key...\] | 求两个或多个有序集合的并集，并将返回结果存储在新的 key 中 |
| zscan key cursor \[match pattern\] \[count count\] | 迭代有序集合中的元素（包括元素成员和元素分值） |

### 6. stream


### 7. 关于key键
- `key`的类型对应`value`的类型，使用`type key`命令查看类型
- 可以使用空字符串作为`key`值，`set "" value`，但不建议
- 可以对相同类型相同`key`值的键值对重新赋值，`value`会被覆盖
- 可以对`key`设置过期时间`expire`，到点自动删除，用于避免使用频率不高的`key`占用内存以及控制缓存失效时间
	1. `redis`底层把设置过过期时间的`key`存放在一个独立的字典中，定时遍历来删除过期`key`
	2. 使用“惰性策略”，客户端访问某`key`时会先对该`key`的过期时间进行检查，过期则立即删除
- 关于`key`的命令格式

| 命令 | 说明 |
| ----- | ----- |
| del key | 若键存在的情况下，该命令用于删除键 |
| dump key | 用于序列化给定 key ，并返回被序列化的值 |
| exists key | 用于检查键是否存在，若存在则返回 1，否则返回 0 |
| expire key | 设置 key 的过期时间，以秒为单位 |
| expireat key | 该命令与 EXPIRE 相似，用于为 key 设置过期时间，不同在于，它的时间参数值采用的是时间戳格式 |
| pexpire key | 设置 key 的过期，以毫秒为单位 |
| pexpireat key | 与 PEXPIRE 相似，用于为 key 设置过期时间，采用以毫秒为单位的时间戳格式 |
| keys pattern | 此命令用于查找与指定 pattern 匹配的 key |
| move key db | 将当前数据库中的 key 移动至指定的数据库中（默认存储为 0 库，可选 1-15中的任意库） |
| persist key | 该命令用于删除 key 的过期时间，然后 key 将一直存在，不会过期 |
| pttl key | 用于检查 key 还剩多长时间过期，以毫秒为单位 |
| ttl key | 用于检查 key 还剩多长时间过期，以秒为单位 |
| randomkey | 从当前数据库中随机返回一个 key |
| rename key newkey | 修改 key 的名称 |
| renamenx key newkey| 如果新键名不重复，则将 key 修改为 newkey |
| scan cursor | 基于游标的迭代器，用于迭代数据库中存在的所有键，cursor 指的是迭代游标 |
| type key | 该命令用于获取 value 的数据类型 |

## 3. 配置文件
使用`config`可以查看或者更改`redis`的相关配置信息：
- `redis 127.0.0.1:6379> config get 配置名称`
- `redis 127.0.0.1:6379> config set 配置名称 配置值`
如：
- `redis 127.0.0.1:6379> config get loglevel`
- `redis 127.0.0.1:6379> config set loglevel verbose`
- `redis 127.0.0.1:6379> config get *`

配置项说明

| 配置项 | 参数 | 说明 |
| ----- | ----- | ----- |
| daemonize | no/yes | 默认为 no，表示 Redis 不是以守护进程的方式运行，通过修改为 yes 启用守护进程 |
| pidfile | 文件路径 | 当 Redis 以守护进程方式运行时，会把进程 pid 写入自定义的文件中 |
| port | 6379 | 指定 Redis 监听端口，默认端口为 6379 |
| bind | 127.0.0.1 | 绑定的主机地址 |
| timeout | 0 | 客户端闲置多长秒后关闭连接，若指定为 0 ，表示不启用该功能 |
| loglevel | notice | 指定日志记录级别，支持四个级别：debug、verbose、notice、warning，默认为 notice |
| logfile | stdout | 日志记录方式，默认为标准输出 |
| databases | 16 | 设置数据库的数量（0-15个）共16个，Redis 默认选择的是 0 库，可以使用 SELECT 命令来选择使用哪个数据库储存数据 |
| save\[seconds\] \[changes\] | 可以同时配置三种模式：save 900 1;save 300 10;save 60 10000 | 表示在规定的时间内，执行了规定次数的写入或修改操作，Redis 就会将数据同步到指定的磁盘文件中。比如 900s 内做了一次更改，Redis 就会自动执行数据同步 |
| rdbcompression | yes/no | 当数据存储至本地数据库时是否要压缩数据，默认为 yes |
| dbfilename | dump.rdb | 指定本地存储数据库的文件名，默认为 dump.rdb |
| dir | ./ | 指定本地数据库存放目录 |
| slaveof \<masterip\> \<masterport\> | 主从复制配置选项 | 当本机为 slave 服务时，设置 master 服务的 IP 地址及端口，在 Redis 启动时，它会自动与 master 主机进行数据同步 |
| requirepass | foobared默认关闭 | 密码配置项，默认关闭，用于设置 Redis 连接密码。如果配置了连接密码，客户端连接 Redis 时需要通过\<password\> 密码认证 |
| maxmemory \<bytes\> | 最大内存限制配置项 | 指定 Redis 最大内存限制，Redis 在启动时会把数据加载到内存中，达到最大内存后，Redis 会尝试清除已到期或即将到期的 Key，当此方法处理 后，若仍然到达最大内存设置，将无法再进行写入操作，但可以进行读取操作 |
| appendfilename | appendonly.aof | 指定 AOF 持久化时保存数据的文件名，默认为 appendonly.aof |
| glueoutputbuf | yes | 设置向客户端应答时，是否把较小的包合并为一个包发送，默认开启状态 |

## 4. 常用命令
```
客户端远程连接
redis-cli -h host -p port -a password

配置客户端连接密码
config set requirepass password
验证密码是否与配置密码一致
auth password
切换库
select dbNumber
中断连接quit

查看是否设置密码验证
config get requirepass

避免一些危险指令例如flushdb、flushall造成的严重后果，可以在redis配置文件的security模块中重命名指令
rename-command flushdb 123flushdb123
rename-command flushall 123flushall123

在redis暴露到外网时，可以在配置文件中绑定要监听的IP地址
bing 192.167.1.1
还可以增加redis密码访问限制
requirepass yourpasswd
要求从机必须配置与主服务相同的密码才可以进行主从复制
masterauth yourpasswd

客户端常用命令
client list 以列表形式返回所有连接到服务器的客户端
client setname 设置当前连接名称
client getname 获取当前连接名称
client pause 挂起客户端连接
client kill 关闭客户端连接
client id 返回客户端ID
client reply 控制发送当前连接的回复，on|off|skip

服务端命令
太多了...
```

## 5. 发布与订阅
`redis pubsub`又称发布订阅模式，即`publish/subscribe`，实现消息传递与多播功能。发布者`publisher`发布消息，订阅者`subscriber`订阅消息。用来传递消息的链路称为`channel`，一个客户端可以订阅任意数量的`channel`

*消息多播：生产者生产一次消息，中间件负责将消息复制到多个消息队列中，每个消息队列由相应的消费组进行消费*

```
订阅channel
127.0.0.1:6379> subscribe a-channel
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "a-channel"
3) (integer) 1

发布消息到channel中
127.0.0.1:6379> publish a-channel "abcdefg"
(integer) 1
127.0.0.1:6379> publish a-channel "xxxx"
(integer) 1
127.0.0.1:6379> publish a-channel "dddd"
(integer) 1
```

本质上就是某些客户端充当发布者，另外某些客户端充当订阅者，消息在`redis`中的`channel`里传递

消息发布订阅格式：

| 命令 | 说明 |
| ----- | ----- |
| psubscribe pattern \[pattern...\] | 订阅一个或多个符合指定模式的频道 |
| pubsub subcommand \[argument \[argument...\]\] | 查看发布/订阅系统状态，可选参数；channel 返回在线状态的频道；numpat 返回指定模式的订阅者数量；numsub 返回指定频道的订阅者数量 |
| pubsub subcommand \[argument \[argument...\]\] | 将信息发送到指定的频道 |
| punsubscribe \[pattern \[pattern...\]\] | 退订所有指定模式的频道 |
| subscribe channel \[channel...\] | 订阅一个或者多个频道的消息 |
| unsubscribe \[channel \[channel...\]\] | 退订指定的频道 |

## 6. 命名空间
`redis`进行数据缓存，可以对多个键值对使用**命名空间**进行分类；以命名空间开头的方式存储数据，使不同类型的数据统一到一个命名空间下

- 键值以`namespace:key`命名，则在库中创建了一个以`namespace`命名的文件夹，文件夹下存在多个`key`
- 键值以`namespace::key`命名，则在库中创建了一个以`namespace`命名的文件夹，该文件夹下还存在一个无名文件夹，而无名文件夹下才存在多个`key`

## 7. Springboot+Redis整合
### 1. 使用Jedis
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!-- jedis 连接池依赖-->
<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-pool2</artifactId>
</dependency>

<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.1.0</version>
</dependency>
```

```properties
#ip地址
redis.host=192.168.25.131
#端口号
redis.port=6379
#如果有密码
redis.password=123456
#客户端超时时间单位是毫秒 默认是2000
redis.timeout=3000
#数据库，默认的是0
redis.database=0
#最大空闲数
redis.maxIdle=300  
#连接池的最大数据库连接数。设为0表示无限制,如果是jedis 2.4以后用redis.maxTotal
redis.maxActive=1000
#控制一个pool可分配多少个jedis实例,用来替换上面的redis.maxActive,如果是jedis 2.4以后用该属性
redis.maxTotal=1000
#最大建立连接等待时间。如果超过此时间将接到异常。设为-1表示无限制。
redis.maxWaitMillis=1000
#在空闲时检查有效性, 默认false
redis.testOnBorrow=false
#连接耗尽是否阻塞,false代表抛异常,true代表阻塞直到超时,默认为true
redis.blockWhenExhausted=false
#是否启用pool的jmx管理功能, 默认true
redis.JmxEnabled=true
 
#下面的不是必须的配置
#连接的最小空闲时间 默认1800000毫秒(30分钟)
minEvictableIdleTimeMillis=300000  
#每次释放连接的最大数目,默认3
numTestsPerEvictionRun=1024  
#逐出扫描的时间间隔(毫秒) 如果为负数,则不运行逐出线程, 默认-1
timeBetweenEvictionRunsMillis=30000  
#是否在从池中取出连接前进行检验,如果检验失败,则从池中去除连接并尝试取出另一个,数据量大的时候建议关闭
testWhileIdle=true
```

```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

@Configuration
@PropertySource("classpath:application.properties")
public class RedisConfig {
    @Value("${redis.host}")
    private String host;

    @Value("${redis.port}")
    private int port;

    @Value("${redis.password}")
    private int password;

    @Value("${redis.database}")
    private int database;

    @Value("${redis.timeout}")
    private int timeout;

    @Value("${redis.maxIdle}")
    private int maxIdle;

    @Value("${redis.maxWaitMillis}")
    private int maxWaitMillis;

    @Value("${redis.blockWhenExhausted}")
    private Boolean blockWhenExhausted;

    @Value("${redis.JmxEnabled}")
    private Boolean JmxEnabled;

    @Bean
    public JedisPool jedisPoolFactory() {
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        jedisPoolConfig.setMaxIdle(maxIdle);
        jedisPoolConfig.setMaxWaitMillis(maxWaitMillis);
        // 连接耗尽时是否阻塞, false报异常,true阻塞直到超时, 默认true
        jedisPoolConfig.setBlockWhenExhausted(blockWhenExhausted);
        // 是否启用pool的jmx管理功能, 默认true
        jedisPoolConfig.setJmxEnabled(JmxEnabled);
        JedisPool jedisPool = new JedisPool(jedisPoolConfig, host, port, timeout,password,database);
        return jedisPool;
    }
}
```

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.params.SetParams;

@Component
public class RedisUtil {

    @Autowired
    private JedisPool jedisPool;

    /**
     * 向Redis中存值，永久有效
     */
    public String set(String key, String value) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            return jedis.set(key, value);
        } catch (Exception e) {
            return "0";
        } finally {
            jedis.close();
        }
    }

    /**
     * 根据传入Key获取指定Value
     */
    public String get(String key) {
        Jedis jedis = null;
        String value;
        try {
            jedis = jedisPool.getResource();
            value = jedis.get(key);
        } catch (Exception e) {
            return "0";
        } finally {
            jedis.close();
        }
        return value;
    }

    /**
     * 校验Key值是否存在
     */
    public Boolean exists(String key) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            return jedis.exists(key);
        } catch (Exception e) {
            return false;
        } finally {
            jedis.close();
        }
    }

    /**
     * 删除指定Key-Value
     */
    public Long del(String key) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            return jedis.del(key);
        } catch (Exception e) {
            return 0L;
        } finally {
            jedis.close();
        }
    }

    /**
     * 分布式锁
     * @param key
     * @param value
     * @param time 锁的超时时间，单位：秒
     *
     * @return 获取锁成功返回"OK"，失败返回null
     */
    public String getDistributedLock(String key,String value,int time){
        Jedis jedis = null;
        String ret = "";
        try {
            jedis = jedisPool.getResource();

            ret = jedis.set(key, value, new SetParams().nx().ex(time));
            return ret;
        } catch (Exception e) {
            return null;
        } finally {
            jedis.close();
        }
    }
}
```

### 2. 使用SpringBoot中自带的redisTemplate
```xml
<!-- springboot整合redis -->  
<dependency>  
	<groupId>org.springframework.boot</groupId>  
	<artifactId>spring-boot-starter-data-redis</artifactId>  
</dependency>
```

```yml
redis:
	host: 127.0.0.1
	port: 6379
	pool:
		max-active: 8
		max-wait: 1
		max-idle: 8
		min-idle: 0
	timeout: 0
```

```java
package com.example.demo.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

/**
 * redis配置类
 *
 * @author zcc ON 2018/3/19
 **/
@Configuration
public class RedisConfig {
    // 以下两种redisTemplate自由根据场景选择
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);

        //使用Jackson2JsonRedisSerializer来序列化和反序列化redis的value值（默认使用JDK的序列化方式）
        Jackson2JsonRedisSerializer serializer = new Jackson2JsonRedisSerializer(Object.class);

        ObjectMapper mapper = new ObjectMapper();
        mapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        mapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        serializer.setObjectMapper(mapper);

		//value序列化方式采用jackson
        template.setValueSerializer(serializer);
		template.setHashValueSerializer(jackson2JsonRedisSerializer)

        //使用StringRedisSerializer来序列化和反序列化redis的key值
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        template.setKeySerializer(stringRedisSerializer;
		template.setHashKeySerializer(stringRedisSerializer);

        template.afterPropertiesSet();
        return template;
    }
    
    @Bean
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory factory) {
        StringRedisTemplate stringRedisTemplate = new StringRedisTemplate();
        stringRedisTemplate.setConnectionFactory(factory);
        return stringRedisTemplate;
    }
}
```

**注：如果不实现自定义redisTemplate序列化key和value，则redis相关的数据实体类都必须实现Serializable接口，因为redis中传输或者存储数据都会将数据序列化**
```java
//redis相关的数据实体类一定要实现序列化接口
public class User implements Serializable {
	
	private static final long serialVersionUID = -3946734305303957850L;
}
```

```java
/**
 * @Author liuzongqiang
 * @Date 2019/7/27 0027 19:05
 * @Version 1.0
 **/
@Component
public final class RedisUtil {

    @Resource
    private RedisTemplate<String, Object> redisTemplate;

    public Set<String> keys(String keys) {
        try {
            return redisTemplate.keys(keys);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 指定缓存失效时间
     *
     * @param key  键
     * @param time 时间(秒)
     * @return
     */
    public boolean expire(String key, long time) {
        try {
            if (time > 0) {
                redisTemplate.expire(key, time, TimeUnit.SECONDS);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 根据key 获取过期时间
     *
     * @param key 键 不能为null
     * @return 时间(秒) 返回0代表为永久有效
     */
    public long getExpire(String key) {
        return redisTemplate.getExpire(key, TimeUnit.SECONDS);
    }

    /**
     * 判断key是否存在
     *
     * @param key 键
     * @return true 存在 false不存在
     */
    public boolean hasKey(String key) {
        try {
            return redisTemplate.hasKey(key);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 删除缓存
     *
     * @param key 可以传一个值 或多个
     */
    @SuppressWarnings("unchecked")
    public void del(String... key) {
        if (key != null && key.length > 0) {
            if (key.length == 1) {
                redisTemplate.delete(key[0]);
            } else {
                redisTemplate.delete(CollectionUtils.arrayToList(key));
            }
        }
    }

    /**
     * 普通缓存获取
     *
     * @param key 键
     * @return 值
     */
    public Object get(String key) {
        return key == null ? null : redisTemplate.opsForValue().get(key);
    }

    /**
     * 普通缓存放入
     *
     * @param key   键
     * @param value 值
     * @return true成功 false失败
     */
    public boolean set(String key, Object value) {
        try {
            redisTemplate.opsForValue().set(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 普通缓存放入并设置时间
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒) time要大于0 如果time小于等于0 将设置无限期
     * @return true成功 false 失败
     */
    public boolean set(String key, Object value, long time) {
        try {
            if (time > 0) {
                redisTemplate.opsForValue().set(key, value, time, TimeUnit.SECONDS);
            } else {
                set(key, value);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 递增
     *
     * @param key   键
     * @param delta 要增加几(大于0)
     * @return
     */
    public long incr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递增因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, delta);
    }

    /**
     * 递减
     *
     * @param key   键
     * @param delta 要减少几(小于0)
     * @return
     */
    public long decr(String key, long delta) {
        if (delta < 0) {
            throw new RuntimeException("递减因子必须大于0");
        }
        return redisTemplate.opsForValue().increment(key, -delta);
    }

    /**
     * HashGet
     *
     * @param key  键 不能为null
     * @param item 项 不能为null
     * @return 值
     */
    public Object hget(String key, String item) {
        return redisTemplate.opsForHash().get(key, item);
    }

    /**
     * 获取hashKey对应的所有键值
     *
     * @param key 键
     * @return 对应的多个键值
     */
    public Map<Object, Object> hmget(String key) {
        return redisTemplate.opsForHash().entries(key);
    }

    /**
     * HashSet
     *
     * @param key 键
     * @param map 对应多个键值
     * @return true 成功 false 失败
     */
    public boolean hmset(String key, Map<String, Object> map) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * HashSet 并设置时间
     *
     * @param key  键
     * @param map  对应多个键值
     * @param time 时间(秒)
     * @return true成功 false失败
     */
    public boolean hmset(String key, Map<String, Object> map, long time) {
        try {
            redisTemplate.opsForHash().putAll(key, map);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key   键
     * @param item  项
     * @param value 值
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 向一张hash表中放入数据,如果不存在将创建
     *
     * @param key   键
     * @param item  项
     * @param value 值
     * @param time  时间(秒) 注意:如果已存在的hash表有时间,这里将会替换原有的时间
     * @return true 成功 false失败
     */
    public boolean hset(String key, String item, Object value, long time) {
        try {
            redisTemplate.opsForHash().put(key, item, value);
            if (time > 0) {
                expire(key, time);
            }
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 删除hash表中的值
     *
     * @param key  键 不能为null
     * @param item 项 可以使多个 不能为null
     */
    public void hdel(String key, Object... item) {
        redisTemplate.opsForHash().delete(key, item);
    }

    /**
     * 判断hash表中是否有该项的值
     *
     * @param key  键 不能为null
     * @param item 项 不能为null
     * @return true 存在 false不存在
     */
    public boolean hHasKey(String key, String item) {
        return redisTemplate.opsForHash().hasKey(key, item);
    }

    /**
     * hash递增 如果不存在,就会创建一个 并把新增后的值返回
     *
     * @param key  键
     * @param item 项
     * @param by   要增加几(大于0)
     * @return
     */
    public double hincr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, by);
    }

    /**
     * hash递减
     *
     * @param key  键
     * @param item 项
     * @param by   要减少记(小于0)
     * @return
     */
    public double hdecr(String key, String item, double by) {
        return redisTemplate.opsForHash().increment(key, item, -by);
    }

    /**
     * 根据key获取Set中的所有值
     *
     * @param key 键
     * @return
     */
    public Set<Object> sGet(String key) {
        try {
            return redisTemplate.opsForSet().members(key);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 根据value从一个set中查询,是否存在
     *
     * @param key   键
     * @param value 值
     * @return true 存在 false不存在
     */
    public boolean sHasKey(String key, Object value) {
        try {
            return redisTemplate.opsForSet().isMember(key, value);
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将数据放入set缓存
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSet(String key, Object... values) {
        try {
            return redisTemplate.opsForSet().add(key, values);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 将set数据放入缓存
     *
     * @param key    键
     * @param time   时间(秒)
     * @param values 值 可以是多个
     * @return 成功个数
     */
    public long sSetAndTime(String key, long time, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().add(key, values);
            if (time > 0)
                expire(key, time);
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 获取set缓存的长度
     *
     * @param key 键
     * @return
     */
    public long sGetSetSize(String key) {
        try {
            return redisTemplate.opsForSet().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 移除值为value的
     *
     * @param key    键
     * @param values 值 可以是多个
     * @return 移除的个数
     */
    public long setRemove(String key, Object... values) {
        try {
            Long count = redisTemplate.opsForSet().remove(key, values);
            return count;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }
    // ===============================list=================================

    /**
     * 获取list缓存的内容
     *
     * @param key   键
     * @param start 开始
     * @param end   结束 0 到 -1代表所有值
     * @return
     */
    public List<Object> lGet(String key, long start, long end) {
        try {
            return redisTemplate.opsForList().range(key, start, end);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 获取list缓存的长度
     *
     * @param key 键
     * @return
     */
    public long lGetListSize(String key) {
        try {
            return redisTemplate.opsForList().size(key);
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }

    /**
     * 通过索引 获取list中的值
     *
     * @param key   键
     * @param index 索引 index>=0时， 0 表头，1 第二个元素，依次类推；index<0时，-1，表尾，-2倒数第二个元素，依次类推
     * @return
     */
    public Object lGetIndex(String key, long index) {
        try {
            return redisTemplate.opsForList().index(key, index);
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @return
     */
    public boolean lSet(String key, Object value) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     * @return
     */
    public boolean lSet(String key, Object value, long time) {
        try {
            redisTemplate.opsForList().rightPush(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @return
     */
    public boolean lSet(String key, List<Object> value) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 将list放入缓存
     *
     * @param key   键
     * @param value 值
     * @param time  时间(秒)
     * @return
     */
    public boolean lSet(String key, List<Object> value, long time) {
        try {
            redisTemplate.opsForList().rightPushAll(key, value);
            if (time > 0)
                expire(key, time);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 根据索引修改list中的某条数据
     *
     * @param key   键
     * @param index 索引
     * @param value 值
     * @return
     */
    public boolean lUpdateIndex(String key, long index, Object value) {
        try {
            redisTemplate.opsForList().set(key, index, value);
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 移除N个值为value
     *
     * @param key   键
     * @param count 移除多少个
     * @param value 值
     * @return 移除的个数
     */
    public long lRemove(String key, long count, Object value) {
        try {
            Long remove = redisTemplate.opsForList().remove(key, count, value);
            return remove;
        } catch (Exception e) {
            e.printStackTrace();
            return 0;
        }
    }
}
```

### 3. Springboot整合redis订阅发布
