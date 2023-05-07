## 1. 基础概念
<img src="D:\Project\IT notes\框架or中间件\SpringCloud\img\SpringCloud各组件流程关系与作用.png" style="width:700px;height:400px;" />

### 1. Eureka
- **服务发现**：作为服务管理中心，提供服务注册、服务信息统计、服务暴露功能
- **服务提供者**：提供自身服务，让其他应用远程调用执行
- **服务消费者**：一类需要调用远程服务的应用
- **服务中介**：服务提供者与服务消费者之间交流的媒介，服务提供者将自身服务注册到媒介中，而服务消费者通过访问媒介获取远程服务调用的接口
- **服务注册**：服务提供者将自身元数据，如IP地址、服务端口、服务名称、服务接口、服务状态等信息，提交给服务注册中心，达到暴露服务的目的
- **服务续约**：服务提供者会创建注册中心的客户端，通过客户端每隔若干时间向注册中心发送一次心跳续约，向注册中心证明自身服务存活；若注册中心在一定时间内没收到心跳续约，则会将服务实例从服务注册列表中删除
- **获取注册列表信息**：从注册中心获取注册列表后，会缓存在本地。该列表定期更新一次
- **服务下线**：注册中心客户端主动关闭时会向注册中心发送取消请求，相应的服务实例将从注册列表中删除
- **服务剔除**：当客户端连续若干周期内没向注册中心发送服务续约心跳，注册中心则将该服务实例从注册列表中删除

### 2. Ribbon
**Ribbon**是一个运行在消费者端的负载均衡器，可以使用组件中已经定义好的负载均衡算法、或者自定义算法
<img src="D:\Project\IT notes\框架or中间件\SpringCloud\img\Ribbon负载均衡流程图.png" style="width:700px;height:400px;" />

### 3. OpenFeign
在还不存在Feign时，服务之间互相调用需要写入完整的目标URL。Feign让这个过程更加简洁，把服务接口与服务应用名、服务接口请求路径互相映射

*OpenFeign也是运行在消费者端，使用Ribbon负载均衡，所以OpenFeign内置Ribbon*

### 4. Hystrix
提供微服务的**熔断**和**降级**
- 熔断：**当调用某一服务接口失败率达到一定值**，那么这条调用链会被切断，使用回调方法作为服务接口的返回值
- 降级：**当调用某一服务接口出现某种异常时**，通过调用回调方法返回数据

所以*熔断*和*降级*定义上都类似于：当服务接口出现预料之中的某些异常时，调用回调方法，防止雪崩，提高用户体验

### 5. Gateway
<img src="D:\Project\IT notes\框架or中间件\SpringCloud\img\网关大致流程.png" style="width:700px;height:300px;" />

## 2. 基础配置
### 1. 创建父模块

#### 父maven模块pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project
    xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.6.RELEASE</version>
        <relativePath/>
        <!-- lookup parent from repository -->
    </parent>
    <packaging>pom</packaging>
    <modules>
        <module>api</module>
        <module>eureka</module>
        <module>provider</module>
        <module>consumer</module>
    </modules>
    <groupId>com.demo</groupId>
    <artifactId>springcloud-demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springcloud-demo</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>8</java.version>
        <spring-cloud.version>Hoxton.SR12</spring-cloud.version>
    </properties>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
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

### 2. 创建Eureka模块

#### Eureka模块pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project
    xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.demo</groupId>
        <artifactId>springcloud-demo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <relativePath/>
        <!-- lookup parent from repository -->
    </parent>
    <groupId>com.demo</groupId>
    <artifactId>eureka</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>
    <name>eureka</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
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
    <repositories>
        <repository>
            <id>netflix-candidates</id>
            <name>Netflix Candidates</name>
            <url>https://artifactory-oss.prod.netflix.net/artifactory/maven-oss-candidates</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
</project>
```

#### Eureka模块application.yml
```yml
server:  
  port: 9900  
  
eureka:  
  instance:  
    hostname: localhost  
  client:  
    registerWithEureka: false  
    fetchRegistry: false  
    serviceUrl:  
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

#### Eureka模块启动类
```java
package com.demo.eureka;  
  
import org.springframework.boot.SpringApplication;  
import org.springframework.boot.autoconfigure.SpringBootApplication;  
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;  
  
@SpringBootApplication  
@EnableEurekaServer  
public class EurekaApplication {  
  
    public static void main(String[] args) {  
        SpringApplication.run(EurekaApplication.class, args);  
    }  
  
}
```

### 3. 创建SpringBoot类型的api模块

#### api模块pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project
    xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.demo</groupId>
        <artifactId>springcloud-demo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <relativePath/>
        <!-- lookup parent from repository -->
    </parent>
    <groupId>com.demo</groupId>
    <artifactId>api</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>api</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.2.2</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
            <scope>runtime</scope>
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

#### api模块application.yml
```yml
server:  
  port: 9903 #服务端口号  
spring:  
  application:  
    name: api  #微服务名称，对外暴漏的微服务名称，十分重要  
  
eureka:  
  client: #将客户端注册到 eureka 服务列表内  
    service-url:  
      defaultZone: http://localhost:9900/eureka/  #这个地址是注册中心在 application.yml 中暴露出来额注册地址 （单机版）  
  instance:  
    instance-id: spring-cloud-api-9903 #自定义服务名称信息  
    prefer-ip-address: true  #显示访问路径的 ip 地址
```

#### api模块启动类
```java
package com.demo.api;  
  
import org.springframework.boot.SpringApplication;  
//import org.springframework.boot.autoconfigure.EnableAutoConfiguration;  
import org.springframework.boot.autoconfigure.SpringBootApplication;  
//import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;  
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;  
  
@SpringBootApplication  
@EnableEurekaClient  
//@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})  
public class ApiApplication {  
  
    public static void main(String[] args) {  
        SpringApplication.run(ApiApplication.class, args);  
    }  
  
}
```

### 4. 创建SpringBoot类型的provider模块

#### provider模块pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project
    xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.demo</groupId>
        <artifactId>springcloud-demo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <relativePath/>
        <!-- lookup parent from repository -->
    </parent>
    <groupId>com.demo</groupId>
    <artifactId>provider</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>
    <name>provider</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>8</java.version>
        <spring-cloud.version>Hoxton.SR12</spring-cloud.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.2.2</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
            <scope>runtime</scope>
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
        <!--
        <dependency><groupId>com.demo</groupId><artifactId>api</artifactId><version>0.0.1-SNAPSHOT</version></dependency>
        -->
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

#### provider模块application.yml
```yml
server:  
	port: 9902 #服务端口号  
spring:  
	application:  
		name: provider  #微服务名称，对外暴漏的微服务名称，十分重要  
  
eureka:  
	client: #将客户端注册到 eureka 服务列表内  
		service-url:  
			defaultZone: http://localhost:9900/eureka/  #这个地址是注册中心在 application.yml 中暴露出来额注册地址 （单机版）  
	instance:  
		instance-id: spring-cloud-provider-9902 #自定义服务名称信息  
		prefer-ip-address: true  #显示访问路径的 ip 地址
```

#### provider模块启动类
```java
package com.demo.provider;  
  
import org.springframework.boot.SpringApplication;  
//import org.springframework.boot.autoconfigure.EnableAutoConfiguration;  
import org.springframework.boot.autoconfigure.SpringBootApplication;  
//import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;  
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;  
  
@SpringBootApplication  
@EnableEurekaClient  
//@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})  
public class ProviderApplication {  
  
    public static void main(String[] args) {  
        SpringApplication.run(ProviderApplication.class, args);  
    }  
  
}
```

### 5. 创建SpringBoot类型的consumer模块

#### consumer模块pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project
    xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>com.demo</groupId>
        <artifactId>springcloud-demo</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <relativePath/>
        <!-- lookup parent from repository -->
    </parent>
    <groupId>com.demo</groupId>
    <artifactId>consumer</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>
    <name>consumer</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.2.2</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
            <scope>runtime</scope>
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
        <!--
        <dependency>
	        <groupId>com.demo</groupId>
	        <artifactId>api</artifactId>
	        <version>0.0.1-SNAPSHOT</version>
	    </dependency>
        -->

		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
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
    <repositories>
        <repository>
            <id>netflix-candidates</id>
            <name>Netflix Candidates</name>
            <url>https://artifactory-oss.prod.netflix.net/artifactory/maven-oss-candidates</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
</project>
```

#### consumer模块application.yml
```yml
server:  
  port: 9901 #服务端口号  
spring:  
  application:  
    name: consumer  #微服务名称，对外暴漏的微服务名称，十分重要  
  
eureka:  
	client: #将客户端注册到 eureka 服务列表内  
		register-with-eureka: false #本微服务为服务消费者，不需要将自己注册到服务注册中心
		fetch-registry: true #本微服务为服务消费者，需要到服务注册中心搜索服务
		service-url:  
			defaultZone: http://localhost:9900/eureka/  #这个地址是注册中心在 application.yml 中暴露出来额注册地址 （单机版）  
	instance:  
		instance-id: spring-cloud-consumer-9901 #自定义服务名称信息  
		prefer-ip-address: true  #显示访问路径的 ip 地址
```

#### consumer模块启动类
```java
package com.demo.consumer;  
  
import org.springframework.boot.SpringApplication;  
//import org.springframework.boot.autoconfigure.EnableAutoConfiguration;  
import org.springframework.boot.autoconfigure.SpringBootApplication;  
//import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;  
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;  
  
@SpringBootApplication  
@EnableEurekaClient  
//@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})  
public class ConsumerApplication {  
  
    public static void main(String[] args) {  
        SpringApplication.run(ConsumerApplication.class, args);  
    }  
  
}
```