## 1. 降级
### 1. 服务端服务降级
#### 添加依赖
```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

```java
@Service("deptService")
public class DeptServiceImpl implements DeptService {

    @Override
    public String deptInfo_Ok(Integer id) {
        return "线程池：" + Thread.currentThread().getName() + "  deptInfo_Ok,id:   " + id;
    }

    //一旦该方法失败并抛出了异常信息后，会自动调用  @HystrixCommand 注解标注的 fallbackMethod 指定的方法
    @HystrixCommand(fallbackMethod = "dept_TimeoutHandler",
            commandProperties =
                    //规定 5 秒钟以内就不报错，正常运行，超过 5 秒就报错，调用指定的方法
                    {@HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "5000")})
    @Override
    public String deptInfo_Timeout(Integer id) {
        int outTime = 6;
        try {
            TimeUnit.SECONDS.sleep(outTime);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "线程池：" + Thread.currentThread().getName() + "  deptInfo_Timeout,id:   " + id + "  耗时: " + outTime;
    }

    // 当服务出现故障后，调用该方法给出友好提示
    public String dept_TimeoutHandler(Integer id) {
       return  "C语言中文网提醒您，系统繁忙请稍后再试！"+"线程池：" + Thread.currentThread().getName() + "  deptInfo_Timeout,id:   " + id;
    }
}
```

**在服务端启动类上添加注解`@EnableCircuitBreaker`**

### 2. 客户端服务降级
#### 添加依赖
```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

```xml
feign:
	hystrix:
		enabled: true #开启客户端 hystrix
```

```java
@RestController
public class HystrixController_Consumer {

    @Resource
    private DeptHystrixService deptHystrixService;

    @RequestMapping(value = "/consumer/dept/hystrix/ok/{id}")
    public String deptInfo_Ok(@PathVariable("id") Integer id) {
        return deptHystrixService.deptInfo_Ok(id);
    }

    //在客户端进行降级
    @RequestMapping(value = "/consumer/dept/hystrix/timeout/{id}")
    @HystrixCommand(fallbackMethod = "dept_TimeoutHandler") //为该请求指定专属的回退方法
    public String deptInfo_Timeout(@PathVariable("id") Integer id) {
        String s = deptHystrixService.deptInfo_Timeout(id);
        log.info(s);
        return s;
    }

    // deptInfo_Timeout方法的 专用 fallback 方法
    public String dept_TimeoutHandler(@PathVariable("id") Integer id) {
        log.info("deptInfo_Timeout 出错，服务已被降级！");
        return "C语言中文网提醒您：服务端系统繁忙，请稍后再试！（客户端 deptInfo_Timeout 专属的回退方法触发）";
    }
}
```

```yml
######################### Ribbon 客户端超时控制 ###################################
ribbon:
  ReadTimeout: 6000 #建立连接所用的时间，适用于网络状况正常的情况下，两端两端连接所用的时间
  ConnectionTimeout: 6000 #建立连接后，服务器读取到可用资源的时间
######################配置请求超时时间##########################
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 7000
####################配置具体方法超时时间 为 3 秒########################
    DeptHystrixService#deptInfo_Timeout(Integer):
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 3000

# Hystrix 可以来为所有请求（方法）设置超时时间（单位为毫秒），若请求超时则触发全局的回退方法进行处理
# hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=mmm

# Hystrix 还可以为某个特定的服务请求（方法）设置超时时间
# hystrix.command.xxx#yyy(zzz).execution.isolation.thread.timeoutInMilliseconds=mmm
```

**在消费者启动类上添加注解`@EnableHystrix`**

#### 全局降级
若是设置了全局的回退，则需要在Controller类上设置注解`@DefaultProperties(defaultFallback = "xxx")`，并在需要回退的方法上添加注解`@HystrixCommand`，但这种方式会让`fallback`方法与被降级方法耦合度极高

**相应解耦方式**
```java
@Component
public class DeptHystrixFallBackService implements DeptHystrixService {
    @Override
    public String deptInfo_Ok(Integer id) {
        return "--------------------C语言中文网提醒您，系统繁忙，请稍后重试！（解耦回退方法触发）-----------------------";
    }

    @Override
    public String deptInfo_Timeout(Integer id) {
        return "--------------------C语言中文网提醒您，系统繁忙，请稍后重试！（解耦回退方法触发）-----------------------";
    }
}

@Component
@FeignClient(value = "MICROSERVICECLOUDPROVIDERDEPTHYSTRIX", fallback = DeptHystrixFallBackService.class)
public interface DeptHystrixService {
    @RequestMapping(value = "/dept/hystrix/ok/{id}")
    public String deptInfo_Ok(@PathVariable("id") Integer id);
	
    @RequestMapping(value = "/dept/hystrix/timeout/{id}")
    public String deptInfo_Timeout(@PathVariable("id") Integer id);
}
```

## 2. 熔断
当微服务系统中的某个微服务不可用或响应时间太长时，为了保护系统的整体可用性，熔断器会暂时切断请求对该服务的调用，并快速返回一个友好的错误响应。这种熔断状态不是永久的，在经历了一定的时间后，熔断器会再次检测该微服务是否恢复正常，若服务恢复正常则恢复其调用链路

在熔断机制中涉及了三种熔断状态：  
- 熔断关闭状态（Closed）：当服务访问正常时，熔断器处于关闭状态，服务调用方可以正常地对服务进行调用
- 熔断开启状态（Open）：默认情况下，在固定时间内接口调用出错比率达到一个阈值（例如 50%），熔断器会进入熔断开启状态。进入熔断状态后，后续对该服务的调用都会被切断，熔断器会执行本地的降级（FallBack）方法
- 半熔断状态（Half-Open）： 在熔断开启一段时间之后，熔断器会进入半熔断状态。在半熔断状态下，熔断器会尝试恢复服务调用方对服务的调用，允许部分请求调用该服务，并监控其调用成功率。如果成功率达到预期，则说明服务已恢复正常，熔断器进入关闭状态；如果成功率仍旧很低，则重新进入熔断开启状态

<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\Hystrix机制.png" style="width:500px;height:500px;" />

Hystrix 实现服务熔断的步骤如下：  
1. 当服务的调用出错率达到或超过 Hystix 规定的比率（默认为 50%）后，熔断器进入熔断开启状态
2. 熔断器进入熔断开启状态后，Hystrix 会启动一个休眠时间窗，在这个时间窗内，该服务的降级逻辑会临时充当业务主逻辑，而原来的业务主逻辑不可用
3. 当有请求再次调用该服务时，会直接调用降级逻辑快速地返回失败响应，以避免系统雪崩
4. 当休眠时间窗到期后，Hystrix 会进入半熔断转态，允许部分请求对服务原来的主业务逻辑进行调用，并监控其调用成功率
5. 如果调用成功率达到预期，则说明服务已恢复正常，Hystrix 进入熔断关闭状态，服务原来的主业务逻辑恢复；否则 Hystrix 重新进入熔断开启状态，休眠时间窗口重新计时，继续重复第 2 到第 5 步

```java
//Hystrix 熔断案例
@Override
@HystrixCommand(fallbackMethod = "deptCircuitBreaker_fallback", commandProperties = {
        //以下参数在 HystrixCommandProperties 类中有默认配置
        @HystrixProperty(name = "circuitBreaker.enabled", value = "true"), 
        //是否开启熔断器
	    @HystrixProperty(name = "metrics.rollingStats.timeInMilliseconds",value = "1000"), 
	    //统计时间窗
        @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"), 
        //统计时间窗内请求次数
        @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "10000"), 
        //休眠时间窗口期
        @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "60"), 
        //在统计时间窗口期以内，请求失败率达到 60% 时进入熔断状态
})
public String deptCircuitBreaker(Integer id) {
    if (id < 0) {
        //当传入的 id 为负数时，抛出异常，调用降级方法
        throw new RuntimeException("c语言中文网提醒您，id 不能是负数！");
    }
    String serialNum = IdUtil.simpleUUID();
    return Thread.currentThread().getName() + "\t" + "调用成功，流水号为：" + serialNum;
}

//deptCircuitBreaker 的降级方法
public String deptCircuitBreaker_fallback(Integer id) {
    return "c语言中文网提醒您，id 不能是负数,请稍后重试!\t id:" + id;
}
```
## 3. 监控
### 1. 监控端配置

**新建一个SpringBoot类型应用作为dashboard应用**

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yml
server:
	port: 9002  #端口号

spring:
	application:
		name: hystrix-dashboard

#http://xxx:9002/hystrix 熔断器监控页面
# localhost:8004//actuator/hystrix.stream 监控地址
hystrix:
	dashboard:
		proxy-stream-allow-list: "localhost"
```

**在启动类上添加注解`@EnableHystrixDashboard`**

### 2. 被监控端配置
**添加配置类**
```java
@Configuration
public class HystrixDashboardConfig {
    /**
     *  Hystrix dashboard 监控界面必须配置
     * @return
     */
    @Bean
    public ServletRegistrationBean getServlet() {
        HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.addUrlMappings("/actuator/hystrix.stream");//访问路径
        registrationBean.setName("hystrix.stream");
        return registrationBean;
    }
}
```