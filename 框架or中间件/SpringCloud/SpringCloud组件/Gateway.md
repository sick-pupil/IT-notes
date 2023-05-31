## 1. Predicate断言
Spring Cloud Gateway 通过 Predicate 断言来实现 Route 路由的匹配规则。简单点说，Predicate 是路由转发的判断条件，请求只有满足了 Predicate 的条件，才会被转发到指定的服务上进行处理

使用 Predicate 断言需要注意以下 3 点：
-   Route 路由与 Predicate 断言的对应关系为“一对多”，一个路由可以包含多个不同断言
-   一个请求想要转发到指定的路由上，就必须同时匹配路由上的所有断言
-   当一个请求同时满足多个路由的断言条件时，请求只会被首个成功匹配的路由转发

<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\断言匹配.png" style="width:700px;height:400px;" />

| 断言 | 示例 | 说明 |
| ---------- | ---------- | ---------- |
| Path | \- Path=\/dept\/list\/\*\* | 当请求路径与 \/dept\/list\/\*\* 匹配时，该请求才能被转发到 http:\/\/localhost:8001 上 |
| Before | \- Before=2021\-10\-20T11\:47\:34\.255+08:00\[Asia\/Shanghai\] | 在 2021 年 10 月 20 日 11 时 47 分 34\.255 秒之前的请求，才会被转发到 http\:\/\/localhost\:8001 上 |
| After | \- After=2021\-10\-20T11\:47\:34\.255+08\:00\[Asia\/Shanghai\] | 在 2021 年 10 月 20 日 11 时 47 分 34\.255 秒之后的请求，才会被转发到 http\:\/\/localhost\:8001 上 |
| Between | \- Between=2021\-10\-20T15\:18\:33\.226+08\:00\[Asia\/Shanghai\],2021\-10\-20T15\:23\:33\.226+08:00\[Asia/Shanghai\] | 在 2021 年 10 月 20 日 15 时 18 分 33\.226 秒 到 2021 年 10 月 20 日 15 时 23 分 33\.226 秒之间的请求，才会被转发到 http\:\/\/localhost\:8001 服务器上 |
| Cookie | \- Cookie=name,c.biancheng\.net | 携带 Cookie 且 Cookie 的内容为 name=c\.biancheng\.net 的请求，才会被转发到http\:\/\/localhost\:8001 上 |
| Header | \- Header=X\-Request\-Id,\\d+ | 请求头上携带属性 X\-Request\-Id 且属性值为整数的请求，才会被转发到http\:\/\/localhost\:8001上 |
| Method | \- Method=GET | 只有 GET 请求才会被转发到http\:\/\/localhost\:8001上 |

1. 创建SpringBoot类型应用作为Gateway，添加依赖
```xml
<!--特别注意：在 gateway 网关服务中不能引入 spring-boot-starter-web 的依赖，否则会报错-->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>

<!--Eureka 客户端-->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

2. 配置文件添加配置
```yml
server:
  port: 9527  #端口号
spring:
  application:
    name: microServiceCloudGateway
  cloud:
    gateway: #网关路由配置
      routes:
        #将 micro-service-cloud-provider-dept-8001 提供的服务隐藏起来，不暴露给客户端，只给客户端暴露 API 网关的地址 9527
        - id: provider_dept_list_routh   #路由 id,没有固定规则，但唯一，建议与服务名对应
          uri: http://localhost:8001          #匹配后提供服务的路由地址
          predicates:
            #以下是断言条件，必选全部符合条件
            - Path=/dept/list/**               #断言，路径匹配 注意：Path 中 P 为大写
            - Method=GET #只能时 GET 请求时，才能访问

eureka:
  instance:
    instance-id: micro-service-cloud-gateway-9527
    hostname: micro-service-cloud-gateway
  client:
    fetch-registry: true
    register-with-eureka: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
```

3. 启动类添加注解`@EnableEurekaClient`

## 2. 动态路由
```yml
server:
  port: 9527 #端口号
 
spring:
  application:
    name: microServiceCloudGateway  #服务注册中心注册的服务名
   
  cloud:
    gateway: #网关路由配置
      discovery:
        locator:
          enabled: true #默认值为 true，即默认开启从注册中心动态创建路由的功能，利用微服务名进行路由

      routes:
        #将 micro-service-cloud-provider-dept-8001 提供的服务隐藏起来，不暴露给客户端，只给客户端暴露 API 网关的地址 9527
        - id: provider_dept_list_routh   #路由 id,没有固定规则，但唯一，建议与服务名对应
          uri: lb://MICROSERVICECLOUDPROVIDERDEPT #动态路由，使用服务名代替上面的具体带端口   http://eureka7001.com:9527/dept/list

          predicates:
            #以下是断言条件，必选全部符合条件
            - Path=/dept/list/**    #断言，路径匹配 注意：Path 中 P 为大写
            - Method=GET #只能时 GET 请求时，才能访问

eureka:
  instance:
    instance-id: micro-service-cloud-gateway-9527
    hostname: micro-service-cloud-gateway
  client:
    fetch-registry: true
    register-with-eureka: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/
```

以上配置说明如下：
-   lb：uri 的协议，表示开启 Spring Cloud Gateway 的负载均衡功能
-   service-name：服务名，Spring Cloud Gateway 会根据它获取到具体的微服务地址

## 3. Filter
```yml
spring:
  cloud:
    gateway: 
      routes:
        - id: xxxx
          uri: xxxx
          predicates:
            - Path=xxxx
          filters:
            - AddRequestParameter=X-Request-Id,1024 #过滤器工厂会在匹配的请求头加上一对请求头，名称为 X-Request-Id 值为 1024
            - PrefixPath=/dept #在请求路径前面加上 /dept
            ……
```

#### SpringCloud Gateway内置的GatewayFilter
| 路由过滤器 | 描述 | 参数 | 使用示例 |
| ----- | ----- | ----- | ----- |
| AddRequestHeader | 拦截传入的请求，并在请求上添加一个指定的请求头参数 | name：需要添加的请求头参数的 key；value：需要添加的请求头参数的 value | \- AddRequestHeader=my\-request\-header,1024 |
| AddRequestParameter | 拦截传入的请求，并在请求上添加一个指定的请求参数 | name：需要添加的请求参数的 key；value：需要添加的请求参数的 value | \- AddRequestParameter=my\-request\-param,c.biancheng.net |
| AddResponseHeader | 拦截响应，并在响应上添加一个指定的响应头参数 | name：需要添加的响应头的 key；value：需要添加的响应头的 value | \- AddResponseHeader=my\-response\-header,c\.biancheng\.net |
| PrefixPath | 拦截传入的请求，并在请求路径增加一个指定的前缀 | prefix：需要增加的路径前缀 | \- PrefixPath=\/consumer |
| PreserveHostHeader | 转发请求时，保持客户端的 Host 信息不变，然后将它传递到提供具体服务的微服务中 |  | \- PreserveHostHeader |
| RemoveRequestHeader | 移除请求头中指定的参数 | name：需要移除的请求头的 key | \- RemoveRequestHeader=my\-request\-header |
| RemoveResponseHeader | 移除响应头中指定的参数 | name：需要移除的响应头 | \- RemoveResponseHeader=my\-response\-header |
| RemoveRequestParameter | 移除指定的请求参数 | name：需要移除的请求参数 | \- RemoveRequestParameter=my\-request\-param |
| RequestSize | 配置请求体的大小，当请求体过大时，将会返回 413 Payload Too Large | maxSize：请求体的大小 | \- name: RequestSize args:maxSize: 5000000 |

## 4. 全局Filter
```java
@Component
@Slf4j
public class MyGlobalFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("进入自定义的全局过滤器 MyGlobalFilter" + new Date());
        String uname = exchange.getRequest().getQueryParams().getFirst("uname");
        if (uname == null) {
            log.info("参数 uname 不能为 null！");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        //过滤器的顺序，0 表示第一个
        return 0;
    }
}
```

## 5. 网关限流
