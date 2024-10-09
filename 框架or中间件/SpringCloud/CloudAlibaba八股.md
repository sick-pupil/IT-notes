<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\整体微服务架构.png" style="width:700px;height:450px;" />

## 1. Nacos
### 1. 关键特性
1. 服务发现和服务健康监测
2. 动态配置服务
3. 动态`DNS`服务
4. 服务及其元数据管理
### 2. 注册中心
<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\Nacos注册中心大致流程.png" style="width:700px;height:450px;" />

引入心跳机制后的注册中心流程：
<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\Nacos注册中心新增心跳流程.png" style="width:700px;height:500px;" />

- **服务注册：** `NacosClient`会通过发送`REST`请求的方式向`NacosServer`注册自已的服务，提供自身的元数据，比如`ip`地址、端口等信息。`NacosServer`接收到注册请求后，就会把这些元数据信息存储在一个双层的内存`Map`中
- **服务心跳：** 在服务注册后，`Nacos Client`会维护一个定时心跳来持续通知`Nacos Server`，说明服务一直处于可用状态，防止被剔除。默认`5s`发送一次心跳
- **服务同步：** `Nacos Server`集群之间会互相同步服务实例，用来保证服务信息的一致性
- **服务发现：** 服务消费者（`Nacos Client`）在调用服务提供者的服务时，会发送一个`REST`请求给`NacosServer`，获取上面注册的服务清单，并且缓存在`Nacos Client`本地，同时会在`Nacos Client`本地开启一个定时任务定时拉取服务端最新的注册表信息更新到本地缓存
- **服务健康检查：** `NacosServer`会开启一个定时任务用来检查注册服务实例的健康情况，对于超过`15s`没有收到客户端心跳的实例会将它的`healthy`属性置为`false`（客户端服务发现时不会发现），如果某个实例超过30秒没有收到心跳，直接剔除该实例
### 3. Nacos 1.X注册中心原理
1. 服务提供方使用`http`发送注册
2. 服务消费方查询服务提供方列表
3. 服务消费方定时10秒拉取
4. 检测服务提供方变化，`Nacos Server`基于`UDP`协议广播推送更新
5. `Nacos`客户端定时5秒发送心跳，提供自身服务状态，更新`Nacos Server`中的服务列表信息
6. 定时心跳任务检查，`Nacos server`发现当前时间距离上次心跳时间超过15秒，则置节点为不健康；超过30秒则置节点不可用，移除节点
7. 集群数据使用`Distro`实现同步任务

<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\Nacos1.x注册中心原理.png" style="width:700px;height:400px;" />

### 4. Nacos服务领域模型

| 模型名称      | 解释                                    |
| --------- | ------------------------------------- |
| Namespace | 实现环境隔离，默认public                       |
| Group     | 不同的service可以组成一个group，默认Default-Group |
| Service   | 服务名称                                  |
| Cluster   | 对指定的微服务虚拟划分，默认值Default                |
| Instance  | 某个服务的具体实例                             |

<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\Nacos服务领域模型.png" style="width:700px;height:300px;" />

### 5. Nacos-Distro协议
`Nacos`的`Distro`协议是一种基于`P2P`的数据分发协议，主要用于将`NacosServer`中的配置信息和元数据同步到所有的节点上。具体而言，`Distro`协议包括以下几个特点：
- 基于`P2P`的数据分发：`Distro`协议采用对等网络（`Peer-to-Peer`）结构进行数据分发，即每个节点都可以作为数据的源头和接收端。当一个节点有新的数据需要分发时，它会将数据传输给与其相连的其他节点，并让这些节点进一步将数据分发到其他节点上，直到所有节点能接收到该数据
- 数据增量同步：`Distro`协议支持增量同步方式，即每次只同步新增或修改的数据项，避免了全量同步的低效性和带宽浪费问题。在数据增量同步过程中，`Distro`协议使用了版本号机制来保证数据的一致性
- 冗余备份：`Distro`协议支持余备份，即当某个节点失效或网络故障时，其他节点仍能够继续工作并提供服务。在`Distro`协议中，每个数据项都会存储在多个节点上，从而保证数据的高可用性和可靠性

1. `Nacos`每个节点自己负责部分的写请求，**根据写请求过来的数据进行哈希求值并取余，获取最终写入的`Nacos`节点序号**
2. 每个节点会把自己负责的新增数据同步给其他节点，**其实类似于Kafka与ElasticSearch的分区与副本概念**
3. 每个节点定时发送自己负责数据的校验值到其他节点以保持数据一致性
4. 每个节点独立处理读请求，及时从本地发出响应
5. 新加入的`Distro`节点会进行全量数据拉取

<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\Nacos-Distro协议.png" style="width:700px;height:350px;" />
### 6. Nacos1.x配置中心长轮询机制
客户端会轮询向服务端发出一个长连接请求，这个长连接最多`30s`就会超时
服务端收到客户端的请求会先判断当前是否有配置更新，有则立即返回
如果没有，服务端会将这个请求拿住`“hold”29.5s`加入队列，最后`0.5s`再检测配置文件
无论有没有更新都进行正常返回，但等待的`29.5s`期间有配置更新可以提前结束并返回

<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\Nacos1.x配置中心长轮询机制.png" style="width:700px;height:200px;" />
### 7. Nacos配置中心配置优先级
```
# 作用、顺序
1. ${application.name}-${profile}.${file-extension} 项目应用名配置文件
2. ${application.name}.${file-extension}
3. extensionConfigs 扩展配置文件
4. sharedConfigs 多个微服务公共配置
5. application-${profile}.${file-extension}
6. application.yml
7. bootstrap.yml
```
### 8. Nacos2.x客户端探活机制
`Nacos`服务端会启动一个定时任务，每3秒执行一次，查看所有连接是否超过`20s`没有通信，如果超过20秒没有通信，服务端就会给客户端发送一个请求，进行探活，如果能正常返回就表示这个服务为正常服务，如果不能正常返回就将其连接删除

<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\Nacos2.x探活机制.png" style="width:700px;height:900px;" />

Nacos 提供了两种服务类型供用户注册实例时选择，分为非持久化实例（又称临时实例）和持久化实例（又称永久实例）
**临时实例与永久实例的健康检查：**
- 临时实例只是临时存在于注册中心中，会在服务下线或不可用时被注册中心剔除，临时实例会与注册中心保持心跳，注册中心会在一段时间没有收到来自客户端的心跳后会将实例设置为不健康，然后在一段时间后进行剔除
- 永久实例在被删除之前会永久的存在于注册中心，且有可能并不知道注册中心存在，不会主动向注册中心上报心跳，那么这个时候就需要注册中心主动进行探活

**临时实例健康检查**：
1. `OpenAPI`：`OpenAPI`的注册方式实际是用户根据自身需求调用`Http`接口对服务进行注册，然后通过`Http`接口发送心跳到注册中心。在注册服务的同时会注册一个全局的客户端心跳检测的任务。在服务一段时间没有收到来自客户端的心跳后，该任务会将其标记为不健康，如果在间隔的时间内还未收到心跳，那么该任务会将其剔除
2. `SDK`：`SDK`的注册方式实际是通过`RPC`与注册中心保持连接（`Nacos 2.x`版本中，旧版的还是仍然通过`OpenAPI`的方式），客户端会定时的通过`RPC`连接向`Nacos`注册中心发送心跳，保持连接的存活。如果客户端和注册中心的连接断开，那么注册中心会主动剔除该`client`所注册的服务，达到下线的效果。同时`Nacos`注册中心还会在注册中心启动时，注册一个过期客户端清除的定时任务，用于删除那些健康状态超过一段时间的客户端

**永久实例健康检查**：
1. 用的是注册中心探测机制，注册中心会在持久化服务初始化时根据客户端选择的协议类型注册探活的定时任务。`Nacos`现在内置提供了三种探测的协议，即`Http`、`TCP`以及`MySQL` 。一般而言`Http`和`TCP`已经可以涵盖绝大多数的健康检查场景
## 2. Feign
### 1. Ribbon底层实现不同服务的不同配置
为不同服务创建不同的`Spring`上下文
<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\Ribbon不同服务不同配置1.png" style="width:700px;height:150px;" />

<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\Ribbon不同服务不同配置2.png" style="width:700px;height:150px;" />

<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\Ribbon不同服务不同配置3.png" style="width:700px;height:200px;" />

<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\Ribbon不同服务不同配置4.png" style="width:700px;height:200px;" />

<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\Ribbon不同服务不同配置5.png" style="width:700px;height:350px;" />

<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\Ribbon不同服务不同配置6.png" style="width:700px;height:350px;" />
### 2. Feign第一次调用耗时长
`ribbon`默认是懒加载的，只有第一次调用的时候才会生成`ribbon`对应的组件，这就会导致首次调用的会很慢的问题

```yml
ribbon:
	eager-load:
		enabled: true
			clients: msb-stock
```
### 3. Ribbon属性配置与类配置优先级
因为存在`@ConditionalOnMissingBean`，所以类的优先级更高
<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\Ribbon属性配置与类配置优先级.png" style="width:700px;height:150px;" />
### 4. Feign性能优化
`Feign`底层默认是`JDK`自带的`HttpURLConnection`，它是单线程发送`HTTP`请求的，不能配置线程
池，我们使用`Okhttp`或者`HttpClient`来发送`http`请求，并且它们两个都支持线程池
常见`HTTP`客户端
- `HttpClient`
- `Okhttp`
- `HttpURLConnection`
- `RestTemplate`
### 5. Feign实现认证传递
实现接口`RequestInterceptor`，通过`header`实现认证传递
```java
package feign;

public interface RequestInterceptor {
	void apply(RequestTemplate template);
}
```
## 3. Sentinel
### 1. Sentinel中使用的限流算法
#### 1. 计数器滑动窗口算法
设定1秒内允许通过的请求是200个，但是在这里我们需要把1秒的时间分成多格，假设分成5格（格数越多，流量过渡越平滑），每格窗口的时间大小是200毫秒，每过200毫秒，就将窗口向前移动一格

在当前时间快（200毫秒）允许通过的请求数应该是70而不是200（只要超过70就会被限流），因为我们最终统计请求数时是需要把当前窗口的值进行累加，进而得到当前请求数来判断是不是需要进行限流

<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\滑动窗口限流算法.png" style="width:700px;height:350px;" />

#### 2. 漏桶算法
漏桶算法以一个常量限制了出口流量速率，因此漏桶算法可以平滑突发的流量。其中漏桶作为流量容
器我们可以看做一个FIFO的队列，当入口流量速率大于出口流量速率时，因为流量容器是有限的，
当超出流量容器大小时，超出的流量会被丢弃

<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\漏桶限流算法.png" style="width:700px;height:400px;" />

- 漏桶具有固定容量，出口流量速率是固定常量（流出请求）
- 入口流量可以以任意速率流入到漏桶中（流入请求）
- 如果入口流量超出了桶的容量，则流入流量会溢出（新请求被拒绝）
- 因为漏桶算法限制了流出速率是一个固定常量值，所以漏桶算法不支持出现突发流出流量
#### 3. 令牌桶算法
最开始，令牌桶是空的，我们以恒定速率往令牌桶里加入令牌，令牌桶被装满时，多余的令牌会被丢弃。当请求到来时，会先尝试从令牌桶获取令牌（相当于从令牌桶移除一个令牌），获取成功则请求被放行，获取失败则阻塞活拒绝请求

<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\令牌桶算法.png" style="width:700px;height:500px;" />

- 最多可以存发b个令牌。如果令牌到达时令牌桶已经满了，那么这个令牌会被丢弃
- 每当一个请求过来时，就会尝试从桶里移除一个令牌；如果没有令牌的话，请求无法通过
- 令牌桶算法限制的是平均流量，因此其允许突发流量（只要令牌桶中有令牌，就不会被限流）
### 2. Sentinel服务熔断过程
- 熔断关闭状态：服务没有故障时，熔断器所处的状态，对调用方的调用不做任何限制
- 熔断开启状态：后续对该服务接口的调用不再经过网络，直接执行本地的`fallback`方法
- 半熔断状态：尝试恢复服务调用，允许有限的流量调用该服务，并监控调用成功率。如果成功率达到预期，则说明服务已恢复，进入熔断关闭状态；如果成功率仍旧很低，则重新进入熔断关闭状态

<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\熔断开启与关闭过程.png" style="width:700px;height:250px;" />
## 4. Gateway
### 1. Gateway中实现路由的平滑迁移
使用`Weight`路由的断言工厂进行服务权重的配置，并将配置放到`Nacos`配置中心进行动态迁移
包含两个参数，`group`与`weight`，可以针对同一个路由的目的地址，设置多个`routes`但为同一个`group`，并给每个`routes`设置不同的权重

```yml
spring:
	cloud:
		gateway:
			routes:
			- id: weight_high
				uri: https://weighthigh.org
				predicates:
				- Weight=group1, 8
			- id: weight_low
				uri: https://weightlow.org
				predicates:
				- Weight=group1, 2
```
## 5. Seata
### 1. Seata支持的事务模式
`Seata`是一款开源的分布式事务解决方案，致力于提供高性能和简单易用的分布式事务服务。
`Seata`将为用户提供了`AT`、`TCC`、`SAGA`和`XA`事务模式
都为`2PC`两阶段提交
1. `AT`：提供无侵入自动补偿的事务模式，基于本地能支持事务的关系型数据库，然后
`java`代码可以通过`JDBC`访问数据库， 这里的无侵入：我们只需要加上对应的注解就可以开启全
局事务

2. `XA`：支持已实现`XA`接口的数据库的`XA`模式，需要数据库实现对应的XA模式的接
口，一般像`mysql oracle`都实现了`XA`

3. `TCC`：`TCC`则可以理解为在应用层面的`2PC`，是需要我们编写业务逻辑来实现

4. `SAGA`：为长事务提供有效的解决方案

### 2. 2PC流程
<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\2PC流程.png" style="width:700px;height:400px;" />

- **单点问题**：事务管理器在整个流程中扮演的角色很关键，如果其宕机，比如在第一阶段已经完
成，在第二阶段正准备提交的时候事务管理器宕机，资源管理器就会一直阻塞，导致数据库无法
使用
- **同步阻塞**：在准备就绪之后，资源管理器中的资源一直处于阻塞，直到提交完成，释放资源
- **数据不一致**：两阶段提交协议虽然为分布式数据强一致性所设计，但仍然存在数据不一致性的可
能，比如在第二阶段中，假设协调者发出了事务`commit`的通知，但是因为网络问题该通知仅被
一部分参与者所收到并执行了`commit`操作，其余的参与者则因为没有收到通知一直处于阻塞状
态，这时候就产生了数据的不一致性

### 3. Seata中的xid通过Feign进行全局传递
<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\Seata全局事务ID在Feign中的传递.png" style="width:700px;height:500px;" />
### 5. Seata的AT模式两阶段提交过程
1. 一阶段：开启全局事务、注册分支事务、储存全局锁、业务数据和回滚日志`undoLog`行提交
2. 二阶段：事务协调者根据所有分支的情况，决定本次全局事务是`Commit`还是`Rollback`（二阶段是完全异步删除undolog日志，全局事务、分支事务、储存的全局锁）

<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\Seata全局事务AT模式的执行流程.png" style="width:700px;height:500px;" />

1. 开启全局事务
	1. 向`TC`发起`GlobalBeginRequest`请求
	2. 生成全局事务信息`GlobalSession`存储到`global`表中
	3. 返回全局事务`xid`，绑定到`RootContext`
2. 开启分支事务
	1. 执行业务逻辑
	2. 获取前置镜像`beforeImage`，即`tableRecord`
	3. 执行业务`SQL`
	4. 获取后置镜像`afterImage`，即`tableRecord`
	5. 生成`undolog`并缓存
	6. 向`TC`注册分支事务，向`TC`发起`BranchRegisterRequest`请求，存储分支事务`BranchSession`到`branch`表中，并存储行锁到`lock`表中，返回`branchId`
	7. 保存`undolog`到`undo_log`表中
	8. 本地事务提交
3. 提交全局事务
	1. 向`TC`发起`GlobalCommitRequest`请求
	2. 更新`global`表中的全局事务记录状态（`global、branch、undo、lock`表存在定时任务定期删除完成的全局事务记录）
	3. 返回`GlobalStatus`