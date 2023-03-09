## 1. 概念
负载均衡，就是将用户的请求平摊分配到多个服务器上运行，以达到扩展服务器带宽、增强数据处理能力、增加吞吐量、提高网络的可用性和灵活性

负载均衡常见两种方式：
- 服务端负载均衡
- 客户端负载均衡

服务端负载均衡：在客户端与服务端之间建立一个独立的负载均衡服务器，比如Nginx，负载均衡服务器通过心跳机制维护着一份可用服务端清单
这种服务端的负载均衡方式，会把客户端的所有请求集中在负载均衡处理处，再由负载均衡服务器按照某种算法从可用服务清单选择一个进行请求转发

而客户端负载均衡则是把负载均衡的逻辑封装在客户端上，让客户端从注册中心获取可用服务清单，然后负载均衡器通过某种算法挑选出清单中的某个可用服务作为发出请求的目标
这种方式则不需要搭建特定的负载均衡服务器，也不会出现负载均衡处堵塞过多的请求

| 区别点      | 服务端负载均衡 | 客户端负载均衡 |
| ----------- | ----------- | ----------- |
| 是否需要建立负载均衡服务器      |   需要在客户端和服务端之间建立一个独立的负载均衡服务器     | 将负载均衡的逻辑以代码的形式封装到客户端上，因此不需要单独建立负载均衡服务器 |
| 是否需要服务注册中心   |    不需要服务注册中心     | 需要服务注册中心。在客户端负载均衡中，所有的客户端和服务端都需要将其提供的服务注册到服务注册中心上 |
| 可用服务清单存储的位置   |   可用服务清单存储在位于客户端与服务器之间的负载均衡服务器上      | 所有的客户端都维护了一份可用服务清单，这些清单都是从服务注册中心获取的 |
| 负载均衡的时机   |  先将请求发送到负载均衡服务器，然后由负载均衡服务器通过负载均衡算法，在多个服务端之间选择一个进行访问；即在服务器端再进行负载均衡算法分配。简单点说就是，先发送请求，再进行负载均衡       | 在发送请求前，由位于客户端的服务负载均衡器（例如 Ribbon）通过负载均衡算法选择一个服务器，然后进行访问。简单点说就是，先进行负载均衡，再发送请求 |
| 客户端是否了解服务提供方信息   |   由于负载均衡是在客户端发送请求后进行的，因此客户端并不知道到底是哪个服务端提供的服务      | 负载均衡是在客户端发送请求前进行的，因此客户端清楚的知道是哪个服务端提供的服务 |

## 2. 配置
#### 添加依赖
```xml
<dependency>  
    <groupId>org.springframework.cloud</groupId>  
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>  
</dependency>
```

1. 在客户端工程中添加配置类
```java
// 配置类注解
@Configuration
public class ConfigBean {
    
    @Bean //将 RestTemplate 注入到容器中
    @LoadBalanced //在客户端使用 RestTemplate 请求服务端时，开启负载均衡（Ribbon）
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```
2. 使用`restTemplate`进行负载均衡
```java
@RestController
public class DeptController_Consumer {
    //private static final String REST_URL_PROVIDER_PREFIX = "http://localhost:8001/"; 这种方式是直调用服务方的方法，根本没有用到 Spring Cloud

    //面向微服务编程，即通过微服务的名称来获取调用地址
    private static final String REST_URL_PROVIDER_PREFIX = "http://MICROSERVICECLOUDPROVIDERDEPT"; // 使用注册到 Spring Cloud Eureka 服务注册中心中的服务，即 application.name

    @Autowired
    private RestTemplate restTemplate; //RestTemplate 是一种简单便捷的访问 restful 服务模板类，是 Spring 提供的用于访问 Rest 服务的客户端模板工具集，提供了多种便捷访问远程 HTTP 服务的方法

    //获取指定部门信息
    @RequestMapping(value = "/consumer/dept/get/{id}")
    public Dept get(@PathVariable("id") Integer id) {
        return restTemplate.getForObject(REST_URL_PROVIDER_PREFIX + "/dept/get/" + id, Dept.class);
    }
    //获取部门列表
    @RequestMapping(value = "/consumer/dept/list")
    public List<Dept> list() {
        return restTemplate.getForObject(REST_URL_PROVIDER_PREFIX + "/dept/list", List.class);
    }
}
```

## 3. 切换负载均衡策略

| 实现类 | 负载均衡策略 |
| ---------- | ---------- |
| RoundRobinRule | 按照线性轮询策略，即按照一定的顺序依次选取服务实例 |
| RandomRule | 随机选取一个服务实例 |
| RetryRule | 按照 RoundRobinRule（轮询）的策略来获取服务，如果获取的服务实例为 null 或已经失效，则在指定的时间之内不断地进行重试（重试时获取服务的策略还是 RoundRobinRule 中定义的策略），如果超过指定时间依然没获取到服务实例则返回 null |
| WeightedResponseTimeRule | WeightedResponseTimeRule 是 RoundRobinRule 的一个子类，它对 RoundRobinRule 的功能进行了扩展。根据平均响应时间，来计算所有服务实例的权重，响应时间越短的服务实例权重越高，被选中的概率越大。刚启动时，如果统计信息不足，则使用线性轮询策略，等信息足够时，再切换到 WeightedResponseTimeRule |
| BestAvailableRule | 继承自 ClientConfigEnabledRoundRobinRule。先过滤点故障或失效的服务实例，然后再选择并发量最小的服务实例 |
| AvailabilityFilteringRule |   先过滤掉故障或失效的服务实例，然后再选择并发量较小的服务实例 |
| ZoneAvoidanceRule | 默认的负载均衡策略，综合判断服务所在区域（zone）的性能和服务（server）的可用性，来选择服务实例。在没有区域的环境下，该策略与轮询（RandomRule）策略类似 |

```java
@Bean
public IRule myRule() {
    // RandomRule 为随机策略
    return  new RandomRule();
}
```

## 4. 自定义负载均衡策略

**需要继承`AbstractLoadBalancerRule`**

```java
/**
* 定制 Ribbon 负载均衡策略
*/
public class MyRandomRule extends AbstractLoadBalancerRule {
    private int total = 0;            // 总共被调用的次数，目前要求每台被调用5次
    private int currentIndex = 0;    // 当前提供服务的机器号

    public Server choose(ILoadBalancer lb, Object key) {
        if (lb == null) {
            return null;
        }
        Server server = null;

        while (server == null) {
            if (Thread.interrupted()) {
                return null;
            }
            //获取所有有效的服务实例列表
            List<Server> upList = lb.getReachableServers();
            //获取所有的服务实例的列表
            List<Server> allList = lb.getAllServers();

            //如果没有任何的服务实例则返回 null
            int serverCount = allList.size();
            if (serverCount == 0) {
                return null;
            }
            //与随机策略相似，但每个服务实例只有在调用 3 次之后，才会调用其他的服务实例
            if (total < 3) {
                server = upList.get(currentIndex);
                total++;
            } else {
                total = 0;
                currentIndex++;
                if (currentIndex >= upList.size()) {
                    currentIndex = 0;
                }
            }
            if (server == null) {
                Thread.yield();
                continue;
            }
            if (server.isAlive()) {
                return (server);
            }
            server = null;
            Thread.yield();
        }
        return server;
    }

    @Override
    public Server choose(Object key) {
        return choose(getLoadBalancer(), key);
    }

    @Override
    public void initWithNiwsConfig(IClientConfig clientConfig) {
        // TODO Auto-generated method stub
    }
}

@Configuration
public class MySelfRibbonRuleConfig {
    @Bean
    public IRule myRule() {
        //自定义 Ribbon 负载均衡策略
        return new MyRandomRule(); //自定义，随机选择某一个微服务，执行五次
    }
}
```

在消费者启动类添加注解`@RibbonClient(name = "生产者应用名spring.application.name", configuration = MySelfRibbonRuleConfig.class)`，在消费者应用启动时就会自动加载自定义的Ribbon配置类，使配置类生效

**`@RibbonClient`可以让单个消费者应用针对某个生产者应用加载某种Ribbon的配置信息**