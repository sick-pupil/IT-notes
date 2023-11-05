`Consul`可作为注册中心与配置中心
### 1. 下载
`https://www.consul.io/downloads.html`

### 2. 运行exe文件

### 3. 启动
```shell
# 查看版本号
consul --version

# 使用开发者模式启动
consul agent -dev

# 访问http://localhost:8500
```

### 4. 注册服务
```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yml
server:
  port: 8763
spring:
  application:
    name: consul-provider
  cloud:
    consul:
      host: localhost
      port: 8500
      discovery:
        serviceName: consul-provider
```

```java
@SpringBootApplication
@EnableDiscoveryClient
public class ConsulProviderApplication {

	public static void main(String[] args) {
		SpringApplication.run(ConsulProviderApplication.class, args);
	}
}
```

```java
@RestController
public class HiController {

    @Value("${server.port}")
    String port;
    @GetMapping("/hi")
    public String home(@RequestParam String name) {
        return "hi "+name+",i am from port:" +port;
    }
}
```

### 5. 配置中心
```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-consul-config</artifactId>
</dependency>
```

```yml
spring:
	profiles:
		active: dev 

spring:
	application:
		name: consul-provider
	cloud:
		consul:
			host: localhost
			port: 8500
			discovery:
				register: true # 是否需要注册
				instance-id: ${spring.application.name}-01 # 注册实例id
				serviceName: ${spring.application.name} # 服务名称
				port: ${server.port} # 服务端口
				prefer-ip-address: true # 是否使用ip地址注册
				ip-address: ${spring.cloud.client.ip-address}
			config:
				enabled: true # config是否启用
				format: yaml # 配置的格式
				prefix: config # 配置的基本目录
				# default-context 默认配置，被所有服务读取
				profile-separator: ':' # 配置分隔符
				data-key: data # 应用配置的key 例：config/consul-provider:dev/data
				watch:
					enabled: true # 是否开启自动刷新
					delay: 1000 # 刷新频率
```

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class ConfigDemoMain8007 {
    public static void main(String[] args) {
        SpringApplication.run(ConfigDemoMain8007.class, args);
    }
}
```

```java
@RestController
@RefreshScope
// 添加RefreshScope实现动态刷新配置信息
public class FooBarController {

    @Value("${foo.bar}")
    String fooBar;

    @GetMapping("/foo")
    public String getFooBar() {
        return fooBar;
    }
}
```

**配置中心的要点**
- `bootstrap.yml`和`application.yml`都可以用来配置参数
- `bootstrap.yml`用来程序引导时执行，应用于更加早期配置信息读取。可以理解成系统级别的一些参数配置，这些参数一般是不会变动的
- `application.yml`可以用来定义应用级别的，应用程序特有配置信息，可以用来配置后续各个模块中需使用的公共参数等。如果搭配`config`使用，`application.yml`里面定义的文件可以实现动态替换
- `bootstrap`先于`application`加载，且`application`中的配置项会覆盖`bootstarp`中的相同配置项
- 配置项优先级：`consul配置` > `application.yml` > `bootstarp.yml`
