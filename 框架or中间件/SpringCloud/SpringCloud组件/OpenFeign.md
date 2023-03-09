## 1. 概念
Feign 是一种声明式服务调用组件，它在 RestTemplate 的基础上做了进一步的封装。通过 Feign，我们只需要声明一个接口并通过注解进行简单的配置即可实现对 HTTP 接口的绑定
通过 Feign，我们可以像调用本地方法一样来调用远程服务，而完全感觉不到这是在进行远程调用

| 注解 | 说明 |
| ---------- | ---------- |
| @FeignClient | 该注解用于通知 OpenFeign 组件对 @RequestMapping 注解下的接口进行解析，并通过动态代理的方式产生实现类，实现负载均衡和服务调用 |
| @EnableFeignClients | 该注解用于开启 OpenFeign 功能，当 Spring Cloud 应用启动时，OpenFeign 会扫描标有 @FeignClient 注解的接口，生成代理并注册到 Spring 容器中 |
| @RequestMapping | Spring MVC 注解，在 Spring MVC 中使用该注解映射请求，通过它来指定控制器（Controller）可以处理哪些 URL 请求，相当于 Servlet 中 web.xml 的配置 |
| @GetMapping | Spring MVC 注解，用来映射 GET 请求，它是一个组合注解，相当于 @RequestMapping(method = RequestMethod.GET) |
| @PostMapping | Spring MVC 注解，用来映射 POST 请求，它是一个组合注解，相当于 @RequestMapping(method = RequestMethod.POST) |

## 2. 配置
#### 添加依赖
```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

1. 绑定接口
```java
@Component
// 服务提供者提供的服务名称，即 spring.application.name
@FeignClient(value = "MICROSERVICECLOUDPROVIDERDEPT")
public interface DeptFeignService {
    //对应服务提供者Controller 中定义的方法
    @RequestMapping(value = "/dept/get/{id}", method = RequestMethod.GET)
    public Dept get(@PathVariable("id") int id);

    @RequestMapping(value = "/dept/list", method = RequestMethod.GET)
    public List<Dept> list();
}
```

注意：
- 在`@FeignClient`注解中，`value`属性的取值为：服务提供者的服务名，即服务提供者配置文件`application.yml`中`spring.application.name`的取值
- 接口中定义的每个方法都与服务提供者中`Controller`定义的服务方法对应

2. 使用远程接口
```java
@RestController
public class DeptController_Consumer {
	
	//注入远程接口
    @Autowired
    private DeptFeignService deptFeignService;

    @RequestMapping(value = "/consumer/dept/get/{id}")
    public Dept get(@PathVariable("id") Integer id) {
        return deptFeignService.get(id);
    }

    @RequestMapping(value = "/consumer/dept/list")
    public List<Dept> list() {
        return deptFeignService.list();
    }
}
```

3. 在消费者端启动类添加注解`@EnableFeignClients`开启OpenFeign功能

## 3. 设置超时
**因为OpenFeign底层由Ribbon实现，因此配置OpenFeign实则需要配置Ribbon**
```yml
ribbon:
	ReadTimeout: 6000 #建立连接所用的时间，适用于网络状况正常的情况下，两端两端连接所用的时间
	ConnectionTimeout: 6000 #建立连接后，服务器读取到可用资源的时间
```