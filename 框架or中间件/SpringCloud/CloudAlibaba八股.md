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

## 4. Gateway

## 5. Seata

