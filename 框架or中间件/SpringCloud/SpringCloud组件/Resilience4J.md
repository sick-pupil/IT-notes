Hystrix的平替方案
存在两种starter：
1. `org.springframework.cloud:spring-cloud-starter-circuitbreaker-resilience4j` 普通模式
2. `org.springframework.cloud:spring-cloud-starter-circuitbreaker-reactor-resilience4j` reactor 模式

## 1. 默认配置
要为所有断路器提供默认配置，我们需要创建一个自定义`bean`，该`bean`被传递给`Resilience4JCuitBreakerFactory`或`ReactiveResilience4JCuitBreakerFactory`。`configureDefault`方法可用于提供默认配置

```java
	@Bean
	public Customizer<Resilience4JCircuitBreakerFactory> defaultCustomizer() {
		return factory -> factory.configureDefault(id -> new Resilience4JConfigBuilder(id)
				// 默认超时时间 4s
				.timeLimiterConfig(TimeLimiterConfig.custom().timeoutDuration(Duration.ofSeconds(4)).build())
				// circuitBreaker 使用默认配置
				.circuitBreakerConfig(CircuitBreakerConfig.ofDefaults())
				.build());
	}

	/**
     * 初始化断路器，读取Resilience4J的yaml配置
     * @param circuitBreakerRegistry
     * @return
     */
    @Bean
    public ReactiveResilience4JCircuitBreakerFactory reactiveResilience4JCircuitBreakerFactory(
            CircuitBreakerRegistry circuitBreakerRegistry) {
         ReactiveResilience4JCircuitBreakerFactory factory = new ReactiveResilience4JCircuitBreakerFactory();
 
        //自定义断路器配置
//        CircuitBreakerConfig circuitBreakerConfig = CircuitBreakerConfig.custom().slidingWindowSize(100).build();
 
        //设置断路器默认配置
        //不修改默认值可以忽略
        factory.configureDefault(id -> new Resilience4JConfigBuilder(id)
                        //默认超时规则,默认1s,不使用断路器超时规则可以设置大一点
                        .timeLimiterConfig(TimeLimiterConfig.custom().timeoutDuration(Duration.ofMillis(60000)).build())
                        //默认断路器规则
//                        .circuitBreakerConfig(circuitBreakerConfig).build())
        .build());
                      
        //添加自定义拦截器
        factory.configureCircuitBreakerRegistry(circuitBreakerRegistry);
        return factory;
    }
```

## 2. 使用
### 1. 断路
```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot2</artifactId>
    <version>1.7.0</version>
</dependency>

<!--
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-cloud2</artifactId>
    <version>1.7.0</version>
</dependency>

<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
</dependency>
-->

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

```yml
resilience4j.circuitbreaker:
	configs: # 通用配置
	    default: # 断路器系统默认配置
			#失败率，错误率达到或高于该值则进入open状态
			failureRateThreshold: 50
			#慢调用阀值，请求执行的时间大于该值时会标记为慢调用
			slowCallDurationThreshold: 60s
			#慢调用熔断阀值，当慢调用率达到或高于该值时，进入open状态
			slowCallRateThreshold: 100
			#状态滚动收集器大小，close状态时收集多少请求状态，用于计算失败率。
			slidingWindowSize: 100
			#状态收集器类型
			#COUNT_BASED：根据数量计算，slidingWindowSize为次数
			#TIME_BASED：根据时间计算，slidingWindowSize为秒数
			slidingWindowType: COUNT_BASED
			#计算错误率的最小请求数，不足最小调用次数不会触发任何变化。
			minimumNumberOfCalls: 10
			#是否自动进入halfOpen状态，默认false-一定时间后进入halfopen，ture-需要通过接口执行。
			automaticTransitionFromOpenToHalfOpenEnabled: false
			#进入halfOpen状态时，可以被调用次数，就算这些请求的失败率，低于设置的失败率变为close状态，否则变为open。
			permittedNumberOfCallsInHalfOpenState: 10
			#open状态变为half状态需要等待的时间，即熔断多久后开始尝试访问被熔断的服务。
			waitDurationInOpenState: 60s
			#事件缓冲区大小？？
			eventConsumerBufferSize: 10
			#被计为失败的异常集合，默认情况下所有异常都为失败。
			recordExceptions:
				- java.lang.Exception
			#不会被计为失败的异常集合，优先级高于recordExceptions。
			ignoreExceptions:
				- java.lang.IllegalStateException
	instances:
		aCustomizer:
			baseConfig: default
		circuitBreakerDemo:
			failure-rate-threshold: 50 # 失败率超过50%就熔断
			wait-duration-in-open-state: 60000 # 熔断后等待60S才允许再次调用
			sliding-window-size: 10 # 5个请求统计一次
			sliding-window-type: COUNT_BASED # 断路器的滑动窗口期类型可以基于“次数”，TIME_BASED为时间
			permitted-number-of-calls-in-half-open-state: 3 # 进入半开的状态的时候，还允许请求多少个
			minimum-number-of-calls: 5 # 最少有多少个请求才开始统计
```

```java
@RestController
public class Resilience4jController
{
    private final RestTemplate restTemplate;
    
    public Resilience4jController(RestTemplateBuilder builder)
    {
        this.restTemplate = builder.build();
    }
    
    @GetMapping("/circuit-breaker")
    @CircuitBreaker(name = "circuitBreakerDemo") 
    public String circuitBreaker()
    {
        return restTemplate.getForObject("http://no-such-site/api/external", String.class);
    }
}
```

### 2. 重试
```yml
resilience4j.retry:
	instances:
		retryDemo:
			max-attempts: 3 # 最大尝试重试次数
			wait-duration: 1s # 每次重试之间的间隔时间
	metrics:
		legacy:
			enabled: true
		enabled: true
```

```java
@RestController
public class Resilience4jController
{
    private final RestTemplate restTemplate;
    
    public Resilience4jController(RestTemplateBuilder builder)
    {
        this.restTemplate = builder.build();
    }
    
    @GetMapping("/retry")
    @Retry(name = "retryDemo", fallbackMethod = "fallback") 
    public String retry()
    {
        return restTemplate.getForObject("http://no-such-site/api/external", String.class);
    }
    
    public String fallback(Exception exception)
    {
        return "系统故障，无法访问";
    }
}
```

### 3. 隔板
```yml
resilience4j.bulkhead:
	instances:
		bulkheadDemo:
			max-concurrent-calls: 3 # 隔板允许的最大并发执行数量
			max-wait-duration: 1 # 当试图进入一个饱和的隔板时，线程应被阻断的最大时间
	metrics:
		enabled: true
```

```java
@GetMapping("/bulkhead")
@Bulkhead(name = "bulkheadDemo")
public String bulkhead()
{
    return restTemplate.getForObject("https://reqres.in/api/users", String.class);
}

@ExceptionHandler({ BulkheadFullException.class })
@ResponseStatus(HttpStatus.BANDWIDTH_LIMIT_EXCEEDED)
public String handleBulkheadFullException() {
	return "隔板已满";
}
```

### 4. 限流
```yml
resilience4j.ratelimiter:
	instances:
		rateLimitDemo:
			limit-for-period: 5
			limit-refresh-period: 60s
			timeout-duration: 0s
			allow-health-indicator-to-fail: true
			subscribe-for-events: true
			event-consumer-buffer-size: 50
			register-health-indicator: true
	metrics:
		enabled: true
```

```java
@GetMapping("/rate-limiter")
@RateLimiter(name = "rateLimitDemo")
public String rateLimit()
{
    return restTemplate.getForObject("https://reqres.in/api/users", String.class);
}

@ExceptionHandler({RequestNotPermitted.class})
@ResponseStatus(HttpStatus.TOO_MANY_REQUESTS)
public String handleRequestNotPermitted()
{
    return "请求不被允许";
}
```