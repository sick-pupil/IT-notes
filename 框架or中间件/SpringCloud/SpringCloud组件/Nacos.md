## 1. 基础概念
`Nacos`为服务注册中心与配置中心的组合体，可以替换`Eureka`作为服务注册中心与`SpringCloud Config`作为配置中心（类似的还有`Consul`）

- 服务发现
- 服务健康监测：`Nacos`提供对服务的实时健康检查，能够阻止请求发送到不健康主机或服务实例上。`Nacos`还提供了一个健康检查仪表盘，能够帮助我们根据健康状态管理服务的可用性及流量
- 动态配置：动态配置服务可以让我们以中心化、外部化和动态化的方式，管理所有环境的应用配置和服务配置；消除了配置变更时重新部署应用和服务的需要，让配置管理变得更加高效、敏捷；提供了一个简洁易用的`UI`帮助我们管理所有服务和应用的配置
- 动态`DNS`服务：提供了动态`DNS`服务，能够让我们更容易地实现负载均衡、流量控制以及数据中心内网的简单`DNS`解析服务
- 元数据管理：能让我们从微服务平台建设的视角管理数据中心的所有服务及元数据，包括管理服务的描述、生命周期、服务的静态依赖分析、服务的健康状态、服务的流量管理、路由及安全策略、服务的`SLA`以及`metrics`统计数据

|组件|描述|功能|
|---|---|---|
|Nacos Server|Nacos 服务端，与 Eureka Server 不同，Nacos Server 由阿里巴巴团队使用 Java 语言编写并将 Nacos Server 的下载地址给用户，用户只需要直接下载并运行即可。|Nacos Server 可以作为服务注册中心，帮助 Nacos Client 实现服务的注册与发现。Nacos Server 可以作为配置中心，帮助 Nacos Client 在不重启的情况下，实现配置的动态刷新。|
|Nacos Client|Nacos 客户端，通常指的是微服务架构中的各个服务，由用户自己搭建，可以使用多种语言编写。|Nacos Client 通过添加依赖 spring-cloud-starter-alibaba-nacos-discovery，在服务注册中心（Nacos Server）中实现服务的注册与发现。Nacos Client 通过添加依赖 spring-cloud-starter-alibaba-nacos-config，在配置中心（Nacos Server）中实现配置的动态刷新。|

`Nacos`实现服务注册与发现的流程：
1. 运行`Nacos Server`
2. 服务提供者`Nacos Client`启动时，会以服务名称注册到服务注册中心
3. 服务消费者`Nacos Client`启动时，也会将自己的服务注册到注册中心
4. 服务被注册到注册中心时，会从服务注册中心获取一份服务注册列表信息，该列表中包含所有注册到服务注册中心上的所有服务信息
5. 服务消费者从注册列表中获取服务生产者的信息后，通过`HTTP`或者消息调用远程服务

## 2. 注册中心
### 1. 配置并启动`Nacos Server`
1. 下载`nacos-server-2.0.3.zip`
2. 解压压缩包，得到`bin`文件夹：存放可执行命令；`conf`文件夹：存放配置文件；`target`文件夹：存放应用`jar`
3. 执行`bin`目录下的`startup.cmd`：`startup.cmd -m standalone`以单机模式启动`nacos server`
4. 使用浏览器访问`http://localhost:8848/nacos`，登录进入`nacos server`，默认登录名与密码均为`nacos`

### 2. 搭建服务提供者
*spring-cloud-alibaba-demo*
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <packaging>pom</packaging>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.6</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>net.biancheng.c</groupId>
    <artifactId>spring-cloud-alibaba-demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <junit.version>4.12</junit.version>
        <log4j.version>1.2.17</log4j.version>
        <lombok.version>1.16.18</lombok.version>
        <spring-cloud.version>2020.0.4</spring-cloud.version>
    </properties>
    <dependencyManagement>
        <dependencies>
            <!--Spring Cloud Alibaba 的版本信息-->
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2021.1</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!--Spring Cloud 的版本信息-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

*spring-cloud-alibaba-provider-8001*
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <!--父工程-->
    <parent>
        <groupId>net.biancheng.c</groupId>
        <version>1.0-SNAPSHOT</version>
        <artifactId>spring-cloud-alibaba-demo</artifactId>
    </parent>

    <groupId>net.biancheng.c</groupId>
    <artifactId>spring-cloud-alibaba-provider-8001</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-cloud-alibaba-provider-8001</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!--Spring Cloud Alibaba Nacos discovery -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

```yml
server:
	port: 8001
spring:
	application:
		name: spring-cloud-alibaba-provider
	cloud:
		nacos:
			discovery:
				server-addr: 127.0.0.1:8848
management:
	endpoints:
		web:
			exposure:
				include: *
```

```java
package net.biancheng.c;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient //开启服务发现功能
public class SpringCloudAlibabaProvider8001Application {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudAlibabaProvider8001Application.class, args);
    }
}
```

```java
package net.biancheng.c.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
@Slf4j
public class DeptController {
    @Value("${server.port}")
    private String serverPort;

    @GetMapping(value = "/dept/nacos/{id}")
    public String getPayment(@PathVariable("id") Integer id) {
        return "<h2>c语言中文网提醒您，服务访问成功！</h2>服务名：spring-cloud-alibaba-provider<br /> 端口号： " + serverPort + "<br /> 传入的参数：" + id;
    }
}
```

### 3. 搭建服务消费者
*spring-cloud-alibaba-consumer-nacos-8801*
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>net.biancheng.c</groupId>
        <version>1.0-SNAPSHOT</version>
        <artifactId>spring-cloud-alibaba-demo</artifactId>
    </parent>
    <groupId>net.biancheng.c</groupId>
    <artifactId>spring-cloud-alibaba-consumer-nacos-8081</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-cloud-alibaba-consumer-nacos-8081</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <!--SpringCloud ailibaba nacos discovery-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--由于 Netflix Ribbon 进入停更维护阶段，因此新版本的 Nacos discovery 都已经移除了 Ribbon ，此时我们需要引入 loadbalancer 代替 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-loadbalancer</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

```yml
server:
	port: 8801
spring:
	application:
		name: spring-cloud-alibaba-consumer
	cloud:
		nacos:
			discovery:
				server-addr: localhost:8848
service-url:
	nacos-user-service: http://spring-cloud-alibaba-provider
```

```java
package net.biancheng.c;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient // 开启服务注册与发现功能
public class SpringCloudAlibabaConsumerNacos8801Application {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudAlibabaConsumerNacos8081Application.class, args);
    }
}
```

```java
package net.biancheng.c.config;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class ApplicationContextBean {

    @Bean
    @LoadBalanced //与 Ribbon 集成，并开启负载均衡功能
    public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

```java
package net.biancheng.c.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

import javax.annotation.Resource;

@RestController
@Slf4j
public class DeptController_Consumer {

    @Resource
    private RestTemplate restTemplate;

    @Value("${service-url.nacos-user-service}")
    private String serverURL; //服务提供者的服务名

    @GetMapping("/consumer/dept/nacos/{id}")
    public String paymentInfo(@PathVariable("id") Long id) {
        return restTemplate.getForObject(serverURL + "/dept/nacos/" + id, String.class);
    }
}
```

## 3. 配置中心
*spring-cloud-alibaba-config-client-3377*
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>net.biancheng.c</groupId>
        <version>1.0-SNAPSHOT</version>
        <artifactId>spring-cloud-alibaba-demo</artifactId>
    </parent>
    <groupId>net.biancheng.c</groupId>
    <artifactId>spring-cloud-alibaba-config-client-3377</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-cloud-alibaba-nacos-config-client-3377</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--SpringCloud2020及以后的版本默认不启用 bootstrap 配置，我们需要在pom里面显式地引入：-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bootstrap</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!--Spring Cloud Alibaba Config 依赖-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <!--SpringCloud ailibaba nacos 服务注册与发现模块 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--Spring Boot 监控模块-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

*bootstrap.yml*
```yml
server:
  port: 3377 #端口号
spring:
  application:
    name: config-client #服务名
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848 #Nacos服务注册中心地址

      config:
        server-addr: 127.0.0.1:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置
```

*application.yml*
```yml
spring:
  profiles:
    active: dev #激活 dev 的配置
```

```java
package net.biancheng.c;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class SpringCloudAlibabaNacosConfigClient3377Application {

    public static void main(String[] args) {
        SpringApplication.run(SpringCloudAlibabaNacosConfigClient3377Application.class, args);
    }
}
```

```java
package net.biancheng.c.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RefreshScope
public class ConfigClientController {

    @Value("${config.info}")
    private String ConfigInfo;

    @GetMapping("/config/info")
    public String getConfigInfo(){
        return ConfigInfo;
    }
}
```

在`nacos server`新建配置列表
```
Data ID:        config-client-dev.yaml

Group  :        DEFAULT_GROUP

配置格式:        YAML

配置内容:      config:
                  info: c.biancheng.net
```

在`nacos server`中，配置的`dataId`完整格式为：`${prefix}-${spring.profiles.active}.${file-extension}`
- `${prefix}`：默认取值为微服务的服务名，即配置文件中`spring.application.name`的值，我们可以在配置文件中通过配置`spring.cloud.nacos.config.prefix`来指定。
- `${spring.profiles.active}`：表示当前环境对应的`Profile`，例如`dev、test、prod`等。当没有指定环境的`Profile`时，其对应的连接符也将不存在，`dataId`的格式变成`${prefix}.${file-extension}`。
- `${file-extension}`：表示配置内容的数据格式，我们可以在配置文件中通过配置项 `spring.cloud.nacos.config.file-extension`来配置，例如`properties`和`yaml`