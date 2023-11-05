## 1. 背景
几乎所有服务的运行都离不开配置文件的支持，这些配置文件通常由各个服务自行管理，以`properties`或`yml`格式保存在各个微服务的类路径下

而在微服务中，通常会使用配置中心统一对配置文件进行管理，而`Spring Cloud Config`可以将各个微服务的配置文件集中存储在一个外部存储仓库中（`Git or SVN`），对配置统一管理

## 2. 概念
`Spring Cloud Config`包含以下两个部分：
- `Config Server`：分布式配置中心，此为一个独立运行的微服务应用，用来连接配置仓库并为客户端提供获取配置信息、加密信息和解密信息的访问接口
- `Config Client`：微服务架构中的各个微服务，通过`Config Server`获取和加载配置信息

*Spring Cloud Config*默认使用`Git`存储配置信息，但还支持其他存储方式，如：`SVN or 本地化文件系统`

<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\Config基本原理.png" style="width:700px;height:350px;" />

`Config`工作流程：
1. 开发或运维人员提交配置文件到远程的`Config`仓库
2. `Config`服务端（分布式配置中心）负责连接配置仓库`Git`，并对`Config`客户端暴露获取配置的接口
3. `Config`客户端通过`Config`服务端暴露的接口，拉取配置仓库中的配置
4. `Config`客户端获取配置信息以支持服务运行

## 3. 使用
### 1. Config服务端
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>spring-cloud-demo2</artifactId>
        <groupId>net.biancheng.c</groupId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <groupId>net.biancheng.c</groupId>
    <artifactId>micro-service-cloud-config-center-3344</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>micro-service-cloud-config-center-3344</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!--配置中心服务器依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

```yml
server:
  port: 3344 #端口号
spring:
  application:
    name: spring-cloud-config-center #服务名
  cloud:
    config:
      server:
        git:
          # Git 地址，https://gitee.com/java-mohan/springcloud-config.git
          # 码云（gitee）地址 uri: https://github.com/javmohan/springcloud-config.git  (github 站点访问较慢，因此这里我们使用 gitee)
          uri: https://gitee.com/java-mohan/springcloud-config.git
          #仓库名
          search-paths:
            - springcloud-config
          force-pull: true
          # 如果Git仓库为公开仓库，可以不填写用户名和密码，如果是私有仓库需要填写
          # username: ********
          # password: ********
      #分支名
      label: master

eureka:                                            
  client: #将客户端注册到 eureka 服务列表内
    service-url: 
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/  #将服务注册到 Eureka 集群
```

```java
package net.biancheng.c;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
@EnableConfigServer
public class MicroServiceCloudConfigCenter3344Application {

    public static void main(String[] args) {
        SpringApplication.run(MicroServiceCloudConfigCenter3344Application.class, args);
    }

}
```

**新建一个config-dev.yml文件并上传到配置中心Git仓库master分支下，可以访问配置中心获取该配置文件内容：`http://localhost:3344/master/config-dev.yml`**
```yml
config:
	info: c.biancheng.net
	version: 2.0
```

`Config`服务端中配置文件访问规则：

| 访问规则 | 示例 |
| ----- | ----- |
| /{application}/{profile}\[/{label}] | /config/dev/master |
| /{application}-{profile}.{suffix} | /config-dev.yml |
| /{label}/{application}-{profile}.{suffix} | /master/config-dev.yml |

- `{application}`：应用名称
- `{profile}`：环境，prod、master、main、dev、test
- `{label}`：分支名
- `{suffix}`：文件后缀，yml、properties

### 2. Config客户端
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>spring-cloud-demo2</artifactId>
        <groupId>net.biancheng.c</groupId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <groupId>net.biancheng.c</groupId>
    <artifactId>micro-service-cloud-config-client-3355</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>micro-service-cloud-config-client-3355</name>
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

        <!--Spring Cloud Config 客户端依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
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
#bootstrap.yml 是系统级别的，加载优先级高于 application.yml ，负责从外部加载配置并解析
server:
  port: 3355 #端口号
spring:
  application:
    name: spring-cloud-config-client #服务名
  cloud:
    config:
      label: master #分支名称
      name: config  #配置文件名称，config-dev.yml 中的 config
      profile: dev  #环境名  config-dev.yml 中的 dev
      #这里不要忘记添加 http:// 否则无法读取
      uri: http://localhost:3344 #Spring Cloud Config 服务端（配置中心）地址

eureka:
  client: #将客户端注册到 eureka 服务列表内
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/  #将服务注册到 Eureka 集群
```

```java
package net.biancheng.c.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

//读取配置中心指定配置文件的内容，并展示到页面
@RestController
public class ConfigClientController {
    @Value("${server.port}")
    private String serverPort;

    @Value("${config.info}")
    private String configInfo;

    @Value("${config.version}")
    private String configVersion;

    @GetMapping(value = "/getConfig")
    public String getConfig() {
        return "info：" + configInfo + "<br/>version：" + configVersion + "<br/>port：" + serverPort;
    }
}
```

```java
package net.biancheng.c;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class MicroServiceCloudConfigClient3355Application {

    public static void main(String[] args) {
        SpringApplication.run(MicroServiceCloudConfigClient3355Application.class, args);
    }
}
```

## 4. 刷新配置
基本的`Config`配置，在配置中心的配置文件发生变化之后，需要重启`Config`客户端才能获取改动后的配置文件

### 1. 可引入`actuator`监控模块进行改造
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yml
# Spring Boot 2.50对 actuator 监控屏蔽了大多数的节点，只暴露了 health 节点，本段配置（*）就是为了开启所有的节点
management:
  endpoints:
    web:
      exposure:
        include: "*"   # * 在yaml 文件属于关键字，所以需要加引号
```

```java
package net.biancheng.c.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

// 读取配置中心指定配置文件的内容，并展示到页面
@RefreshScope //为了让动态（手动）的获取最新的git 配置，在添加 actuator 监控加载 RefreshScope，
@RestController
public class ConfigClientController {
    @Value("${server.port}")
    private String serverPort;

    @Value("${config.info}")
    private String configInfo;

    @Value("${config.version}")
    private String configVersion;

    @GetMapping(value = "/getConfig")
    public String getConfig() {
        return "info：" + configInfo + "<br/> version：" + configVersion + "<br/>port：" + serverPort;
    }
}
```

**配置文件发生改变则调用`curl -X POST "http://localhost:3355/actuator/refresh"`重新拉取配置**

### 2. 结合Bus动态刷新配置
1. 当 Git 仓库中的配置发生改变后，运维人员向`Config`服务端发送一个`POST`请求，请求路径为`/actuator/refresh`
2. `Config`服务端接收到请求后，会将该请求转发给服务总线`Spring Cloud Bus`
3. `Spring Cloud Bus`接到消息后，会通知给所有`Config`客户端
4. `Config`客户端接收到通知，请求`Config`服务端拉取最新配置
5. 所有`Config`客户端都获取到最新的配置

#### 1. `Config`服务端
在原本`Config`服务配置的基本上，添加配置
```xml
<!--添加消息总线（Bus）对 RabbitMQ 的支持-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
<!--添加Spring Boot actuator 监控模块的依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```yml
##### RabbitMQ 相关配置，15672 是web 管理界面的端口，5672 是 MQ 的访问端口###########
spring:
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest

# Spring Boot 2.50对 actuator 监控屏蔽了大多数的节点，只暴露了 heath 节点，本段配置（*）就是为了开启所有的节点
management:
  endpoints:
    web:
      exposure:
        include: 'bus-refresh'
```

#### 2. `Config`客户端
在原本`Config`客户端的基础上添加配置
```xml
<!--添加消息总线（Bus）对 RabbitMQ 的支持-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

```yml
##### RabbitMQ 相关配置，15672 是web 管理界面的端口，5672 是 MQ 的访问端口###########
spring: 
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest
```

#### 3. 新增的Bus应用
在`Config`客户端配置基础上新增一个`Bus`微服务
```yml
#bootstrap.yml 是系统级别的，加载优先级高于 application.yml ，负责从外部加载配置并解析
server:
  port: 3366  #端口号为 3366
spring:
  application:
    name: spring-cloud-config-client-bus

  cloud:
    config:
      label: master #分支名称
      name: config  #配置文件名称，config-dev.yml 中的 config
      profile: dev  #配置文件的后缀名  config-dev.yml 中的 dev
      #这里不要忘记添加 http:// 否则无法读取
      uri: http://localhost:3344 #spring cloud 配置中心地址

##### RabbitMQ 相关配置，15672 是web 管理界面的端口，5672 是 MQ 的访问端口###########
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest
###################### eureka 配置 ####################
eureka:
  client: #将客户端注册到 eureka 服务列表内
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/,http://eureka7003.com:7003/eureka/  #将服务注册到 Eureka 集群

# Spring Boot 2.50对 actuator 监控屏蔽了大多数的节点，只暴露了 heath 节点，本段配置（*）就是为了开启所有的节点
management:
  endpoints:
    web:
      exposure:
        include: "*"   # * 在yaml 文件属于关键字，所以需要加引号
```

**调用`curl -X POST "http://localhost:3344/actuator/bus-refresh"`刷新所有节点的配置文件**

*如果希望只有若干节点而非全部微服务节点刷新配置信息，则调用`http://{hostname}:{port}/actuator/bus-refresh/{destination}`*
- `{hostname}`：`Config`服务端域名或者`IP`
- `{port}`：`Config`服务端端口
- `{destination}`：需要定点通知的`Config`客户端，由`{spring.application.name} + {server.port}`组成
例：`curl -X POST "http://localhost:3344/actuator/bus-refresh/spring-cloud-config-client:3355"`