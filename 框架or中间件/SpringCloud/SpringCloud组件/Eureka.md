## 1. 基本概念

| 术语      | 定义 |
| ----------- | ----------- |
| 服务注册 Register      | 当Eureka客户端向Eureka Server注册时，它提供自身的元数据，比如IP地址、端口，运行状况指示符URL，主页等       |
| 服务续约 Renew   | Eureka客户会每隔30秒(默认情况下)发送一次心跳来续约。 通过续约来告知Eureka Server该Eureka客户仍然存在，没有出现问题。 正常情况下，如果Eureka Server在90秒没有收到Eureka客户的续约它会将实例从其注册表中删除        |
| 获取注册列表信息 Fetch Registries   | Eureka客户端从服务器获取注册表信息，并将其缓存在本地。客户端会使用该信息查找其他服务，从而进行远程调用。该注册列表信息定期（每30秒钟）更新一次。每次返回注册列表信息可能与Eureka客户端的缓存信息不同, Eureka客户端自动处理。如果由于某种原因导致注册列表信息不能及时匹配，Eureka客户端则会重新获取整个注册表信息。Eureka服务器缓存注册列表信息，整个注册表以及每个应用程序的信息进行了压缩，压缩内容和没有压缩的内容完全相同Eureka 客户端和Eureka服务器可以使用JSON / XML格式进行通讯。在默认的情况下Eureka客户端使用压缩JSON格式来获取注册列表的信息        |
| 服务下线 Cancel   | Eureka客户端在程序关闭时向Eureka服务器发送取消请求。 发送请求后，该客户端实例信息将从服务器的实例注册表中删除。该下线请求不会自动完成，它需要调用以下内容：`DiscoveryManager.getInstance().shutdownComponent()`        |
| 服务剔除 Eviction   | 在默认的情况下，当Eureka客户端连续90秒(3个续约周期)没有向Eureka服务器发送服务续约，即心跳，Eureka服务器会将该服务实例从服务注册列表删除，即服务剔除        |

<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\Eureka架构图.png" style="width:700px;height:500px;" />

## 2. Eureka常用配置
#### 添加依赖
```xml
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>  
</dependency>
```

### 1. Eureka服务端
```yml
# Eureka服务端配置
eureka:
	instance:
		hostname: localhost   # 指定Eureka主机
	server: 
		enable-self-preservation: true # Eureka 的自我保护机制，默认开启。如果出现网络延迟导致Eureka客户端心跳无法准时被Eureka服务端接收，Eureka服务端也不会把相应的Eureka客户端信息从注册中心列表删除
	client:
		register-with-eureka: false  # 是否向服务中心注册自己
		fetch-registry: false        # 是否能够获取Eureka注册信息
		service-url:    # 暴露自己的服务中心地址
			defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka
			# 做Eureka集群则在defaultZone配置多个ip并用逗号分隔，不需要写自己的ip地址，写集群中其他Eureka服务端ip
```

在启动类上添加注解`@EnableEurekaServer`

### 2. Eureka客户端
```yml
# Eureka客户端配置
eureka:
	client:
		service-url:    # 暴露服务中心地址
			defaultZone: http://xxxxxx   # Eureka服务端配置的defaultZone
	instance:
		instance-id: xxxxx # 指定当前客户端在注册中心的名称
		prefer-ip-address: true  # 显示访问路径的ip地址
```

在启动类上添加注解`@EnableEurekaClient`