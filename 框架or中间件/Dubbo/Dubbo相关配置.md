## 1. RPC(Remote Procedure Call)远程过程调用
本地调用可以理解为进程内的互相调用；而远程调用可以理解为进程间的互相调用

<img src="D:\Project\IT notes\框架or中间件\Dubbo\img\RPC过程调用.png" style="width:700px;height:600px;" />

1. 服务消费者（client客户端）通过本地调用的方式调用服务
2. 客户端存根（client stub）接收到请求后负责将方法、入参等信息序列化（组装）成能够进行网络传输的消息体
3. 客户端存根（client stub）找到远程的服务地址，并且将消息通过网络发送给服务端
4. 服务端存根（server stub）收到消息后进行解码（反序列化操作）
5. 服务端存根（server stub）根据解码结果调用本地的服务进行相关处理
6. 本地服务执行具体业务逻辑并将处理结果返回给服务端存根（server stub）
7. 服务端存根（server stub）将返回结果重新打包成消息（序列化）并通过网络发送至消费方
8. 客户端存根（client stub）接收到消息，并进行解码（反序列化）
9. 服务消费方得到最终结果

## 2. Dubbo特性
- 面向接口代理的高性能RPC调用
- 服务自动注册与发现
- 负载均衡
- 高度可扩展
- 流量调度
- 可视化服务治理

## 3. Dubbo架构
<img src="D:\Project\IT notes\框架or中间件\Dubbo\img\Dubbo架构.png" style="width:700px;height:400px;" />

## 4. Dubbo-admin管理控制台
1.  下载代码: `git clone https://github.com/apache/dubbo-admin.git`
2.  在 `dubbo-admin-server/src/main/resources/application.properties`中指定注册中心地址
3.  构建
    -   `mvn clean package -Dmaven.test.skip=true`
    -   构建jar包过程中会提示进入`dubbo-admin-develop\dubbo-admin-ui`目录并执行`npm install`
4.  启动
    -   `mvn --projects dubbo-admin-server spring-boot:run` 或者
    -   `cd dubbo-admin-distribution/target; java -jar dubbo-admin-0.1.jar`
5.  访问 `http://localhost:8080`

## 5. Dubbo-Spring整合
`Provider配置文件applicationProvider.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
    http://code.alibabatech.com/schema/dubbo
    http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
    <!-- 具体的实现bean -->
    <bean id="demoService" class="io.kuaibao.provider.service.impl.UserServiceImpl" />
    <!-- 提供方应用信息，用于计算依赖关系 -->
    <dubbo:application name="dubbo_provider"  />
    <!-- 使用zookeeper注册中心暴露服务地址 -->
    <dubbo:registry address="zookeeper://192.168.124.131:2181" />
    <!-- 用dubbo协议在20880端口暴露服务 -->
    <dubbo:protocol name="dubbo" port="20881" />
    <!-- 声明需要暴露的服务接口 -->
    <dubbo:service interface="io.kuaibao.provider.service.UserService" ref="demoService" timeout="10000" > <!-- 远程调用接口，首先比较精确程度；精确程度相同再比较消费者还是生产者：越精确越优先，次之消费者优先 -->
	    <!-- <dubbo:method name="findxxx" timeout="10000" /> -->
    </dubbo:service>
	
	<!-- 配置当前提供者的统一规则 -->
	<dubbo:provider timeout="10000" />
</beans>
```

`Consumer配置文件applicationConsumer.xml`
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://code.alibabatech.com/schema/dubbo
       http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
    <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
    <dubbo:application name="dubbo_consumer" />
    <!-- 使用zookeeper广播注册中心暴露发现服务地址 -->
    <dubbo:registry  protocol="zookeeper" address="zookeeper://192.168.124.131:2181" />
    <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
    <dubbo:reference id="demoService" interface="io.kuaibao.provider.service.UserService" check="false" timeout="10000" retries="3"> <!-- check启动检查 timeout服务调用等待超时时间 -->
	    <!-- <dubbo:method name="getUserAddressList" timeout="1000" /> -->
    </dubbo:reference>

	<!-- 配置当前消费者的统一规则 -->
	<dubbo:consumer check="false" timeout="10000"></dubbo:consumer>
	
	<!-- timeout属性 作用范围越精确则属性引用越优先 -->
	
	<!-- 注册中心统一规则 -->
	<dubbo:registry check="false"></dubbo:registry>
</beans>
```

## 6. Dubbo-SpringBoot整合
1. `导入依赖`
```xml
<dependency>
	<groupId>com.alibaba.boot</groupId>
	<artifactId>dubbo-spring-boot-starter</artifactId>
	<version>0.2.0</version>
<dependency/>
```

2. `Provider配置文件properties`
```properties
dubbo.application.name=boot-user-service-provider
dubbo.registry.address=127.0.0.1:2181
dubbo.registry.protocol=zookeeper

dubbo.protocol.name=dubbo
dubbo.protocol.port=20880

dubbo.monitor.protocol=registry
```

3. 在Provider包中暴露服务，相关服务实现类的上注释注解`@Service+@Component`，`@Service`为dubbo的注解，该注解作用于服务类接口实现类
4. 在Provider包的启动类注释注解`@EnableDubbo`

5. `Consumer配置文件properties`
```properties
dubbo.application.name=boot-order-service-consumer
dubbo.registry.address=zookeeper://127.0.0.1:2181

dubbo.monitor.protocol=registry
```

6. 在Consumer包中引用服务相关类`@Reference`，该注解作用于服务类接口
7. 在Consumer包中的启动类注释注解`@EnableDubbo`

## 7. Dubbo配置文件
Dubbo配置属性覆盖，优先级逐级递减：
1. `java -Ddubbo.protocol.port=20880`
2. `dubbo.xml <dubbo:protocol port="20880" />`
3. `dubbo.properties dubbo.protocol.port=20880`

## 8. 属性配置的覆盖规则
1. 首先比较配置的精确程度，配置范围越精确越优先
2. 若精确程度一致，则消费者配置优先

## 9. 多版本
```xml
<!-- 生产者新旧版本 -->
<dubbo:service interface="com.foo.BarService" version="1.0.0" ref="..." />
<dubbo:service interface="com.foo.BarService" version="2.0.0" ref="..." />

<!-- 消费者新旧版本 -->
<dubbo:reference id="barService" interface="com.foo.BarService" version="1.0.0" />
<dubbo:reference id="barService" interface="com.foo.BarService" version="2.0.0" />
```

## 9. 本地存根
在远程调用服务提供者的实现之前，如果需要做一些参数验证、缓存、判断、小功能等等，满足要求再调用服务提供者的远程服务，则我们可以通过编写一个本地存根来实现这种功能

```java
//在消费者项目中编写远程接口的本地存根实现
public class UserServiceStub implements UserService {
    private final UserService userService;  //存储远程接口的代理实现

    //必须有一个有参构造器，传入的是远程接口的代理实现
    public UserServiceStub(UserService userService) {
        //构造函数传入的是UserService的远程代理对象
        this.userService = userService;
    }

    @Override
    public void addUser(User user) {
        try {
            //在此处进行缓存、参数验证等等
            //如果通过验证则调用远程代理实现的方法
            if(user != null) {
                userService.addUser(user);
            }
        }
        catch (Exception e){
            //可以在此处进行容错等任何AOP拦截事项
        }
    }

    @Override
    public void delById(Integer id) {
        try {
            //在此处进行缓存、参数验证等等
            //如果通过验证则调用远程代理实现的方法
            if(id != 0 || id != null){
                userService.delById(id);
            }
        }
        catch (Exception e){
            //可以在此处进行容错等任何AOP拦截事项
        }
    }

    @Override
    public void modifyUser(User user) {
        try {
            //在此处进行缓存、参数验证等等
            //如果通过验证则调用远程代理实现的方法
            if(user != null){
                userService.modifyUser(user);
            }
        }
        catch (Exception e){
            //可以在此处进行容错等任何AOP拦截事项
        }
    }

    @Override
    public User getById(Integer id) {
        try {
            //在此处进行缓存、参数验证等等
            //如果通过验证则调用远程代理实现的方法
            if(id != 0 || id != null){
                return userService.getById(id);
            }
        }
        catch (Exception e){
            //可以在此处进行容错等任何AOP拦截事项
            return null;
        }
        return null;
    }

    @Override
    public List<User> getList() {
        try {
            //在此处进行缓存、参数验证等等
            //如果通过验证则调用远程代理实现的方法
            return userService.getList();
        }
        catch (Exception e){
            //可以在此处进行容错等任何AOP拦截事项
            return null;
        }
    }
}
```

```xml
<!--声明需要调用的远程服务接口，生成远程服务代理，可以和本地Bean一样使用-->
<dubbo:reference id="userService" interface="cn.coreqi.service.UserService" stub="cn.coreqi.service.stub.UserServiceStub"/>
```

## 10. 多种Dubbo与SpringBoot的整合方式
1. 引入`dubbo-starter`，添加包扫描`@EnableDubbo`，在`application.properties`配置属性，使用`@Service`暴露服务，使用`@Reference`引用服务
2. 保留`dubbo.xml`配置文件，`@ImportResource(locations="classpath:dubbo.xml")`
3. 使用JavaConfig
```java
@Configuration
public class DubboDemoConsumerConfig {

    public static final String APPLICATION_NAME = "consumer-of-helloworld-app";

    public static final String REGISTRY_ADDRESS = "zookeeper://127.0.0.1:2181";

    @Bean
    public ApplicationConfig applicationConfig() {
        ApplicationConfig applicationConfig = new ApplicationConfig();
        applicationConfig.setName(APPLICATION_NAME);
        return applicationConfig;
    }

    @Bean
    public RegistryConfig registryConfig() {
        RegistryConfig registryConfig = new RegistryConfig();
        registryConfig.setAddress(REGISTRY_ADDRESS);
        return registryConfig;
    }

    @Bean
    public ProtocolConfig protocolConfig() {
        ProtocolConfig protocolConfig = new ProtocolConfig();
        protocolConfig.setName("dubbo");
        protocolConfig.setPort(20882);
        return protocolConfig;
    }
	
	@Bean
    public ServiceConfig<UserService> userServiceConfig(UserService userService) {
        ServiceConfig userServiceConfig = new ServiceConfig();
        userServiceConfig.setInterface(UserService.class);
        userServiceConfig.serRef(userService);
        userServiceConfig.setVersion("1.0.0");
		
		MethodConfig methodConfig = new MethodConfig();
		methodConfig.setName("getUserAddressList");
		methodConfig.setTimeout(1000)
		
		List<MethodConfig> methods = new ArrayList<>();
		methods.add(methodConfig);
        return userServiceConfig;
    }
}
```

```java
@DubboComponentScan(basePackages = "...") or @EnableDubbo(scanBasePackages = "...")
@SpringBootApplication
...
```

## 11. Zookeeper宕机与Dubbo直连
-   监控中心宕掉不影响使用，只是丢失部分采样数据
-   数据库宕掉后，注册中心仍能通过缓存提供服务列表查询，但不能注册新服务
-   注册中心对等集群，任意一台宕掉后，将自动切换到另一台
-   **注册中心全部宕掉后，服务提供者和服务消费者仍能通过本地缓存通讯**
-   服务提供者无状态，任意一台宕掉后，不影响使用
-   服务提供者全部宕掉后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复

注册中心的作用就是向服务消费端告诉服务提供端的 url 地址及端口信息，当然也可以**绕过注册中心消费者直接调用生产者**

```xml
<dubbo:reference id="userService" interface="com.yz.dubbo.api.IUserService" check="false" version="1.0.0" url="dubbo://127.0.0.1:20882"></dubbo:reference>
```

## 12. 负载均衡
**负载均衡策略**
-   **Random LoadBalance**
    按权重随机，权重决定调用概率
-   **RoundRobin LoadBalance**
    按权重轮询，权重决定调用次数的占比
-   **LeastActive LoadBalance**
    最少活跃调用数，总是挑选服务响应时间最快的
-   **ConsistentHash LoadBalance**
    一致性 Hash，方法与参数作为Hash参数值进行Hash，结果为调用目标服务器

<img src="D:\Project\IT notes\框架or中间件\Dubbo\img\Dubbo负载均衡算法.png" style="width:700px;height:200px;" />

配置方式：
1. 服务端服务级别与方法级别
```xml
<dubbo:service interface="..." loadbalance="" />

<dubbo:service interface="..." loadbalance="" >
	<dubbo:method name="" loadbalance="" />
</dubbo:service>
```
2. 客户端服务级别与方法级别
```xml
<dubbo:reference interface="..." loadbalance="" />

<dubbo:reference interface="..." loadbalance="" >
	<dubbo:method name="" loadbalance="" />
</dubbo:reference>
```

## 13. 服务降级
当服务器压力剧增时，可以根据实际业务情况及流量，对一些服务和页面有策略的不处理或换种简单的方式处理，从而释放服务器资源以保证核心交易正常运作或高效运作
可以通过服务降级临时屏蔽某个出错的非关键服务，并定义降级后的返回策略

- `mock=force:return+null`：不实际调用远程服务，直接在消费者本地返回空
- `mock=fail:return+null`：等到服务实际调用失败后，再返回null，不抛异常

## 14. 容错
1. `Failover Cluster`：设置`retries`
2. `Failfast Cluster`：只发起一次调用，失败立即报错
3. `Failsafe Cluster`：出现异常直接忽略，计入审计日志
4. `Failback Cluster`：失败自动恢复，失败定时重发
5. `Forking Cluster`：并行调用同个服务中的多个服务器，只要一个成功即返回
6. `Broadcast Cluster`：广播调用，逐个调用服务生产者，只要一个报错则全局报错

配置：
```xml
<!-- 生产者配置 -->
<dubbo:service cluster="failsafe" />

<!-- 消费者配置 -->
<dubbo:reference cluster="failsafe" />
```

### 整合hystrix
1. 添加依赖
```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
	<version>1.4.4.RELEASE</version>
</dependency>
```

2. 添加Enable全局注解
```java
@EnableDubbo
@EnableHystrix
@SpringBootApplication
...
```

3. 在服务提供者实现类中方法上添加`@HystrixCommand`注解，让`Hystrix`代理实现类，出现`Hystrix`配置的错误情况，则调用消费者端配置的服务降级方法
```java
package cn.coreqi.service.impl;

import cn.coreqi.entities.User;
import cn.coreqi.service.UserService;
import com.alibaba.dubbo.config.annotation.Service;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;

@Component  //org.springframework.stereotype.Component
@Service    //com.alibaba.dubbo.config.annotation.Service
public class UserServiceImpl implements UserService {

    private static List<User> users = new ArrayList<>();

    static {
        users.add(new User(1,"fanqi","123456",1));
        users.add(new User(2,"zhangsan","123456",1));
        users.add(new User(3,"lisi","123456",1));
        users.add(new User(4,"wangwu","123456",1));
    }

    @HystrixCommand(commandProperties = {
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "2000") })
    @Override
    public void addUser(User user) {
        users.add(user);
    }

    @HystrixCommand(commandProperties = {
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "2000") })
    @Override
    public void delById(Integer id) {
        for (User s:users){
            if(s.getId() == id){
                users.remove(s);
                break;
            }
        }
    }

    @HystrixCommand(commandProperties = {
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "2000") })
    @Override
    public void modifyUser(User user) {
        delById(user.getId());
        addUser(user);
    }

    @HystrixCommand(commandProperties = {
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "2000") })
    @Override
    public User getById(Integer id) {
        for (User s:users){
            if(s.getId() == id){
                return s;
            }
        }
        return null;
    }

    @HystrixCommand(commandProperties = {
            @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "2000") })
    @Override
    public List<User> getList() {
        return users;
    }
}
```

4. 在服务消费者调用服务提供者的方法上添加`@HystrixCommand`注解并指定`fallbackMethod`属性（指定服务降级方法），重写`fallbackMethod`指定的方法（服务提供者的服务出现`HystrixCommand`所配置的错误后，则调用消费者端配置的`fallbackMethod`方法）
```java
package cn.coreqi.controller;

import cn.coreqi.entities.User;
import cn.coreqi.service.UserService;
import com.alibaba.dubbo.config.annotation.Reference;
import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.List;

@Controller
public class UserController {

    @Reference()
    private UserService userService;

    @HystrixCommand(fallbackMethod = "test1")
    @ResponseBody
    @RequestMapping("/users")
    public List<User> getUsers(){
        return userService.getList();
    }

    public List<User> test1(){
        return null;
    }
}
```