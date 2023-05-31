## 1. springboot-starter
#### springboot项目的pom.xml文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!--SpringBoot父项目依赖管理-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.5</version>
        <relativePath/>
    </parent>
    ....
    <dependencies>
        <!--导入 spring-boot-starter-web-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        ...
    </dependencies>
    ...
</project>
```

SpringBoot将日常企业应用研发中的各种场景都抽取出来，做成一个个的starter（启动器），starter中整合了该场景下各种可能用到的依赖，用户只需要在Maven中引入starter依赖，SpringBoot就能自动扫描到要加载的信息并启动相应的默认配置，starter提供了大量的自动配置，并允许用户调整这些配置，即遵循“约定大于配置”的原则

#### spring-boot-starter-web中的依赖树
```
-   [INFO] Scanning for projects...
-   [INFO]
-   [INFO] --------------------< net.biancheng.www:helloworld >--------------------
-   [INFO] Building helloworld 0.0.1-SNAPSHOT
-   [INFO] --------------------------------[ jar ]---------------------------------
-   [INFO]
-   [INFO] --- maven-dependency-plugin:3.1.2:tree (default-cli) @ helloworld ---
-   [INFO] net.biancheng.www:helloworld:jar:0.0.1-SNAPSHOT
-   [INFO] \- org.springframework.boot:spring-boot-starter-web:jar:2.4.5:compile
-   [INFO] +- org.springframework.boot:spring-boot-starter:jar:2.4.5:compile
-   [INFO] | +- org.springframework.boot:spring-boot:jar:2.4.5:compile
-   [INFO] | +- org.springframework.boot:spring-boot-autoconfigure:jar:2.4.5:compile
-   [INFO] | +- org.springframework.boot:spring-boot-starter-logging:jar:2.4.5:compile
-   [INFO] | | +- ch.qos.logback:logback-classic:jar:1.2.3:compile
-   [INFO] | | | +- ch.qos.logback:logback-core:jar:1.2.3:compile
-   [INFO] | | | \- org.slf4j:slf4j-api:jar:1.7.30:compile
-   [INFO] | | +- org.apache.logging.log4j:log4j-to-slf4j:jar:2.13.3:compile
-   [INFO] | | | \- org.apache.logging.log4j:log4j-api:jar:2.13.3:compile
-   [INFO] | | \- org.slf4j:jul-to-slf4j:jar:1.7.30:compile
-   [INFO] | +- jakarta.annotation:jakarta.annotation-api:jar:1.3.5:compile
-   [INFO] | +- org.springframework:spring-core:jar:5.3.6:compile
-   [INFO] | | \- org.springframework:spring-jcl:jar:5.3.6:compile
-   [INFO] | \- org.yaml:snakeyaml:jar:1.27:compile
-   [INFO] +- org.springframework.boot:spring-boot-starter-json:jar:2.4.5:compile
-   [INFO] | +- com.fasterxml.jackson.core:jackson-databind:jar:2.11.4:compile
-   [INFO] | | +- com.fasterxml.jackson.core:jackson-annotations:jar:2.11.4:compile
-   [INFO] | | \- com.fasterxml.jackson.core:jackson-core:jar:2.11.4:compile
-   [INFO] | +- com.fasterxml.jackson.datatype:jackson-datatype-jdk8:jar:2.11.4:compile
-   [INFO] | +- com.fasterxml.jackson.datatype:jackson-datatype-jsr310:jar:2.11.4:compile
-   [INFO] | \- com.fasterxml.jackson.module:jackson-module-parameter-names:jar:2.11.4:compile
-   [INFO] +- org.springframework.boot:spring-boot-starter-tomcat:jar:2.4.5:compile
-   [INFO] | +- org.apache.tomcat.embed:tomcat-embed-core:jar:9.0.45:compile
-   [INFO] | +- org.glassfish:jakarta.el:jar:3.0.3:compile
-   [INFO] | \- org.apache.tomcat.embed:tomcat-embed-websocket:jar:9.0.45:compile
-   [INFO] +- org.springframework:spring-web:jar:5.3.6:compile
-   [INFO] | \- org.springframework:spring-beans:jar:5.3.6:compile
-   [INFO] \- org.springframework:spring-webmvc:jar:5.3.6:compile
-   [INFO] +- org.springframework:spring-aop:jar:5.3.6:compile
-   [INFO] +- org.springframework:spring-context:jar:5.3.6:compile
-   [INFO] \- org.springframework:spring-expression:jar:5.3.6:compile
-   [INFO] ------------------------------------------------------------------------
-   [INFO] BUILD SUCCESS
-   [INFO] ------------------------------------------------------------------------
-   [INFO] Total time: 2.505 s
-   [INFO] Finished at: 2021-04-30T14:52:30+08:00
-   [INFO] ------------------------------------------------------------------------
```

spring-boot-starter-parent父级依赖用于被SpringBoot项目继承一些默认配置，主要提供以下特性：
- 默认JDK版本（Java 8）
- 默认字符集（UTF-8）
- 依赖管理功能
- 资源过滤
- 默认插件配置
- 识别application.properties和application.yml类型的配置文件

spring-boot-starter-parent底层还有父级依赖spring-boot-dependencies，底层源码提供了许多第三方软件的依赖
#### spring-boot-dependencies
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

    <modelVersion>4.0.0</modelVersion>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.4.5</version>
    <packaging>pom</packaging>
    ....
    <properties>
        <activemq.version>5.16.1</activemq.version>
        <antlr2.version>2.7.7</antlr2.version>
        <appengine-sdk.version>1.9.88</appengine-sdk.version>
        <artemis.version>2.15.0</artemis.version>
        <aspectj.version>1.9.6</aspectj.version>
        <assertj.version>3.18.1</assertj.version>
        <atomikos.version>4.0.6</atomikos.version>
        ....
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.apache.activemq</groupId>
                <artifactId>activemq-amqp</artifactId>
                <version>${activemq.version}</version>
            </dependency>
            <dependency>
                <groupId>org.apache.activemq</groupId>
                <artifactId>activemq-blueprint</artifactId>
                <version>${activemq.version}</version>
            </dependency>
            ...
        </dependencies>
    </dependencyManagement>

    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.codehaus.mojo</groupId>
                    <artifactId>build-helper-maven-plugin</artifactId>
                    <version>${build-helper-maven-plugin.version}</version>
                </plugin>
                <plugin>
                    <groupId>org.flywaydb</groupId>
                    <artifactId>flyway-maven-plugin</artifactId>
                    <version>${flyway.version}</version>
                </plugin>
                ...
            </plugins>
        </pluginManagement>
    </build>
</project>
```

## 2. YAML
SpringBoot默认加载使用以下两种全局的配置文件，文件名固定：
- `application.properties`
- `application.yml`

YAML支持三种数据结构：对象、数组、字面量
```yml
name: abc

website: {name: abc, url: www.baidu.com}

array: 
	-a
	-b
	-c

array: [a,b,c]
```

## 3. 配置文件绑定bean
### 1. @ConfigurationProperties+@Component
```java
@ConfigurationProperties(prefix="db")
@Component //需要把配置类交给spring容器进行管理才能完成配置的自动注入
public class ExternalProperties {
	
	private String url;
	private String driver;
	private String username;
	private String password;
}
```

### 2. @ConfigurationProperties+@EnableConfigurationProperties
```java
@ConfigurationProperties(prefix="db")
public class ExternalProperties {
	
	private String url;
	private String driver;
	private String username;
	private String password;
}

@SpringBootApplication
//@EnableConfigurationProperties让使用了@ConfigurationProperties注解的类生效,并且将该类注入到IOC容器中,交由IOC容器进行管理
@EnableConfigurationProperties({ExternalProperties.class})
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class.args);
	}
}
```

### 3. @PropertySource+@Value
`@Value`使用方式：
- `@Value(“常量”)` 常量,包括字符串,网址,文件路径等
- `@Value(“${}”)` 读取配置文件
- `@Value(“#{}”)` 读取已注入IOC的bean的属性
```java
// 注入普通字符串 @Value("Jack")
private String username;

// 注入文件资源
@Value("classpath:com/test/config.xml")
private Resource resource;

// 注入URL资源
@Value("http://www.baidu.com")
private Resource url;

@Value("${myUserName}")
private String myUserName;

@Value("#{user.username}")
private String username;
```

```java
@Component
//引入外部配置文件组，如果是获取application.properties或application.yml中的属性，则不需要@PropertySource
@PropertySource({"classpath:com/hry/spring/configinject/config.properties"})
public class ConfigurationFileInject{
    @Value("${app.name}")
    private String appName; // 这里的值来自application.properties，spring boot启动时默认加载此文件

    @Value("${book.name}")
    private String bookName; // 注入第一个配置外部文件属性

    @Value("${book.name.placeholder}")
    private String bookNamePlaceholder; // 注入第二个配置外部文件属性

    @Autowired
    private Environment env;  // 注入环境变量对象，存储注入的属性值
}
```

### 4. @ConfigurationProperties+@Import

## 4. 导入Spring配置
在SpringBoot项目中导入Spring xml配置文件需要**在SpringBoot启动类上使用@ImportResource注解**
```java
@ImportResource(locations={"classpath:/beans.xml"})
@SpringBootApplication
public class HelloWorldApplication {
	public static void main(String[] args) {
		SpringApplication.run(HelloWorldApplication.class, args);
	}
}
```
**如果不使用`@ImportResource`注解，则需要使用全注解配置，即`@Configuration+@Bean`**

## 5. Profile
平常项目开发，经常需要根据不同的环境进行配置的修改，比如在本地开发会加载本机的配置和开发环境数据库，在测试服务器上部署时就需要加载测试环境配置和数据库，同样地，当项目发布生产环境时就需要设置为生产环境配置和数据库。这样一来，不同的环境部署都需要额外的处理来调整环境的配置，维护起来十分繁琐，还容易出错

Spring Profiles 提供了一种方式允许我们指定在特定环境下只加载对应的程序配置，每一种环境配置对应一个 Profile，只有当前 Profile 处于激活状态时，才会将该 Profile 所对应的配置和 Bean 加载到 Spring 程序中

### 定义与激活Profile
- `@Profile+@Configuration+application.getEnvrionment()+setActiveProfiles("")`

- 多文件定义，采用application-${profile}.properties命名：
	`application-dev.properties` 开发环境
	`application-test.properties` 测试环境
	`application-pro.properties` 生产环境
最终在`application.properties`中激活具体环境：`spring.profiles.active=xxx`

- yml配置定义（与多文件properties定义相似）
```yaml
---
server:
	port:8080
spring:
	profiles:dev
---
server:
	port:8081
spring:
	profiles:test
---
server:
	port:8082
spring:
	profiles:pro
---
spring:
	profiles:
		active:dev
```

- 虚拟机参数激活：`-Dspring.profiles.active=dev`
- 命令行参数激活：`java –jar xxx.jar --spring.profiles.active=dev`

## 6. 配置文件、配置加载、自动配置
Spring Boot项目中可以存在多个`application.properties`或`apllication.yml`，Spring Boot启动时会扫描以下 5 个位置的`application.properties`或`apllication.yml`文件，并将它们作为Spring boot的默认配置文件：
1. `file：.\/config\/\*\/`
2. `file：.\/config\/`
3. `file：.\/`
4. `classpath：\/config\/`
5. `classpath：\/`

以上所有位置的配置文件都会被加载，且它们优先级依次降低，序号越小优先级越高。其次，位于相同位置的 `application.properties` 的优先级高于 `application.yml`

除了默认配置文件，Spring Boot还可以加载一些位于项目外部的配置文件。我们可以通过如下两个参数，指定外部配置文件的路径：
- `spring.config.location`：`java -jar {JAR} --spring.config.location={外部配置文件全路径}`，会覆盖项目内配置文件
- `spring.config.additional-location`：添加的外部配置文件会与项目默认的配置文件共同生效，形成互补配置，且其优先级是最高的，比所有默认配置文件的优先级都高

SpringBoot项目中必然存在一个使用`@SpringBootApplication`注解注释的启动类，使用该启动类中的main方法SpringApplication.run就可以运行SpringBoot项目，其中@SpringBootApplication注解起到重要作用

`@SpringBootApplication`注解底层包含了三个注解：
- `@SpringBootConfiguration`：表明注解的目标类是SpringBoot的配置类，`@SpringBootConfiguration`底层中实际就是`@Configuration`
- `@EnableAutoConfiguration`：自动配置的主要作用注解
- `@ComponentScan`：开启注解扫描，将注解所注释的目标类化为bean加载入IOC容器中

@EnableAutoConfiguration底层
```java
@AutoConfigurationPackage  
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
```
可以看得出`@EnableAutoConfiguration`底层中包含`@AutoConfigurationPackage`+`@Import`，其中`@AutoConfigurationPackage`底层使用了`@Import({Registrar.class})`
而Registrar类中的`registerBeanDefinitions`方法中就是把主配置类（即使用了`@SpringBooApplication`注解注释的类）所在的包以及下面的所有子包里面的bean扫描到Spring IOC容器中
**因此得出结论：默认情况下SpringBoot项目中会自动扫描主配置类所在包以及其下所有子包中的bean注入IOC容器中**

而`@Import({AutoConfigurationImportSelector.class})`会给当前配置类导入另外的多个自动配置类，`AutoConfigurationImportSelector`中的`selectImport`方法调用了一个`getAutoConfigurationEntry`方法，用来返回需要导入的bean的全类名数组
实际调用链为`getAutoConfigurationEntry() -> getCandidateConfigurations() -> loadFactoryNames()`，而loadFactoryNames方法传入了`EnableAutoConfiguration.class`这个参数，loadFactoryNames方法关键的三步骤：
1. 从当前项目的类路径中获取所有`META-INF/spring.factories`这个文件下的信息
2. 将上面获取到的信息封装成一个Map返回
3. 从返回的 Map 中通过刚才传入的`EnableAutoConfiguration.class`参数，获取该key下的所有值（`META-INF/spring.factories`文件中包含`key-values`，一个key对应多个类全名values）
4. 将类路径下`META-INF/spring.factories`里面配置的所有`EnableAutoConfiguration`的值加入到Spring容器中

自动配置类举例：`HttpEncodingAutoConfiguration`
```java
/**
	@Configuration：标记为配置类。
	@ConditionalOnWebApplication：web应用下才生效。
	@ConditionalOnClass：指定的类（依赖）存在才生效。
	@ConditionalOnProperty：主配置文件中存在指定的属性才生效。
	@EnableConfigurationProperties({HttpProperties.class})：启动指定类的ConfigurationProperties功能；将配置文件中对应的值和 HttpProperties 绑定起来；并把 HttpProperties 加入到 IOC 容器中
**/

@Configuration 
@EnableConfigurationProperties({HttpProperties.class}) 
@ConditionalOnWebApplication( 
type = Type.SERVLET 
) 
@ConditionalOnClass({CharacterEncodingFilter.class}) 
@ConditionalOnProperty( 
prefix = "spring.http.encoding", 
value = {"enabled"}, 
matchIfMissing = true 
) 
public class HttpEncodingAutoConfiguration { 
...

@ConfigurationProperties( 
prefix = "spring.http" 
)// 从配置文件中获取指定的值和bean的属性进行绑定 
public class HttpProperties { 
...
```
## 7. 日志
日志框架分为两类：日志抽象与日志实现，日志抽象为Java日志提供一系列接口表达实现标准与规范。如SLF4J，而日志实现则为日志抽象的具体实现Log4J、Log4J2、Logback
```java
//项目开发中，记录日志通常不调用具体日志实现类的方法，而是调用抽象层的方法
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
    public static void main(String[] args) {
        Logger logger = LoggerFactory.getLogger(HelloWorld.class);
       //调用 sl4j 的 info() 方法，而非调用 logback 的方法
        logger.info("Hello World");
    }
}
```

需要注意：
- Logback作为Slf4j的原生实现框架，当应用使用SLF4J+Logback的组合记录日志时，只需要引入SLF4J和Logback的Jar包即可
- 当应用使用 SLF4J+Log4j 的组合记录日志时，不但需要引入 SLF4J 和 Log4j 的 Jar 包，还必须引入它们之间的适配层（Adaptation layer）slf4j-log4j12.jar，因为Log4j出现早于SLF4J而没有直接实现它

当应用中存在默认的日志实现依赖，而又想使用其他日志实现框架来替换它，则可以排除原本的日志框架后引入新的日志框架依赖

而SpringBoot框架中存在sping-boot-starter-logging，已经引入了多个日志实现依赖以及替换依赖包，引入第三方框架时只需要排除该框架中的默认日志框架即可
<img src="D:\Project\IT-notes\框架or中间件\SpringBoot\img\spring-boot-starter-logging依赖树.png" style="width:700px;height:350px;" />
```xml
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-console</artifactId>
    <version>${activemq.version}</version>
    <exclusions>
        <exclusion>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

SpringBoot中默认使用Logback日志框架，也可使用Log4j2、JUL，常用类路径中的配置文件`logback-spring.xml`、`logback.xml`、`log4j2-spring.xml`、`log4j2.xml`、`logging.properties`
**其中不带spring标识的普通日志配置文件会跳过SpringBoot直接被日志框架加载，而带有spring标识的配置文件会首先由SpringBoot进行解析，这样就可以使用SpringBoot中的Profile功能**

### logback.xml示例
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false">

    <!--定义日志文件的存储地址 勿在 LogBack 的配置中使用相对路径-->
    <property name="LOG_HOME" value="/home" />

    <!--控制台日志， 控制台输出 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度,%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
    </appender>

    <!--文件日志， 按照每天生成日志文件 -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--日志文件输出的文件名-->
            <FileNamePattern>${LOG_HOME}/TestWeb.log.%d{yyyy-MM-dd}.log</FileNamePattern>
            <!--日志文件保留天数-->
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
        <!--日志文件最大的大小-->
        <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
            <MaxFileSize>10MB</MaxFileSize>
        </triggeringPolicy>
    </appender>

    <!-- show parameters for hibernate sql 专为 Hibernate 定制 -->
    <logger name="org.hibernate.type.descriptor.sql.BasicBinder" level="TRACE" />
    <logger name="org.hibernate.type.descriptor.sql.BasicExtractor" level="DEBUG" />
    <logger name="org.hibernate.SQL" level="DEBUG" />
    <logger name="org.hibernate.engine.QueryParameters" level="DEBUG" />
    <logger name="org.hibernate.engine.query.HQLQueryPlan" level="DEBUG" />

    <!--myibatis log configure-->
    <logger name="com.apache.ibatis" level="TRACE"/>
    <logger name="java.sql.Connection" level="DEBUG"/>
    <logger name="java.sql.Statement" level="DEBUG"/>
    <logger name="java.sql.PreparedStatement" level="DEBUG"/>

    <!-- 日志输出级别 -->
    <root level="DEBUG">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

### logback-spring.xml示例
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 日志级别从低到高分为TRACE < DEBUG < INFO < WARN < ERROR < FATAL，比如: 如果设置为WARN，则低于WARN的信息都不会输出 -->
<!-- configuration标签下的三个属性 -->
<!-- scan:当此属性设置为true时，配置文档如果发生改变，将会被重新加载，默认值为true -->
<!-- scanPeriod:设置监测配置文档是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟 -->
<!-- debug:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false -->
<configuration  scan="true" scanPeriod="10 seconds">
    <contextName>logback-test</contextName>

    <!-- 1.property标签用来定义变量值-->
    <!-- name的值是变量的名称，value的值时变量定义的值。通过定义的值会被插入到logger上下文中。定义后，可以使“${}”来使用变量 -->
    <property name="log.path" value="/opt/test/log"/>

    <!-- 2.日志格式和颜色渲染 -->
    <!-- 彩色日志依赖的渲染类 -->
    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />
    <conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter" />
    <conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter" />
    <!-- 彩色日志格式 -->
    <property name="CONSOLE_LOG_PATTERN" value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>

    <!-- 3.appender标签用于写日志的组件 -->
    <!-- 把日志输出到控制台 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <!--此日志appender是为开发使用，只配置最底级别，控制台输出的日志级别是大于或等于此级别的日志信息-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>debug</level>
        </filter>
        <!-- 日志格式化 -->
        <encoder>
            <Pattern>${CONSOLE_LOG_PATTERN}</Pattern>
            <!-- 设置字符集 -->
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!-- 滚动记录文件-->
    <!-- level为 DEBUG 日志，时间滚动输出  -->
    <appender name="DEBUG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文档的路径及文档名 -->
        <file>${log.path}/debug.log</file>
        <!--日志文档输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset> <!-- 设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 日志归档 -->
            <fileNamePattern>${log.path}/debug-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!-- 日志文档保留天数 -->
            <maxHistory>15</maxHistory>
	    <!-- 限制日志文件总容量 -->
	    <totalSizeCap>10GB</totalSizeCap>
        </rollingPolicy>
        <!-- 此日志文档只记录debug级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>debug</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- level为 INFO 日志，时间滚动输出  -->
    <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文档的路径及文档名 -->
        <file>${log.path}/info.log</file>
        <!--日志文档输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 每天日志归档路径以及格式 -->
            <fileNamePattern>${log.path}/info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文档保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文档只记录info级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>info</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- level为 WARN 日志，时间滚动输出  -->
    <appender name="WARN_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文档的路径及文档名 -->
        <file>${log.path}/warn.log</file>
        <!--日志文档输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset> <!-- 此处设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/warn-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文档保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文档只记录warn级别的 -->
	<!-- 级别拦截器如果事件的级别等于配置的级别，则过滤器接受或拒绝该事件，具体取决于onMatch和onMismatch属性的配置。-->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>warn</level>
	    <!-- 上面的级别放行-->
            <onMatch>ACCEPT</onMatch>
	    <!-- 没抓到上面级别的就拦截-->
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- level为 ERROR 日志，时间滚动输出  -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文档的路径及文档名 -->
        <file>${log.path}/error.log</file>
        <!--日志文档输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset> <!-- 此处设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/error-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文档保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文档只记录ERROR级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 所有 除了DEBUG级别的其它高于DEBUG的 日志，记录到一个文件  -->
    <appender name="ALL_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文档的路径及文档名 -->
        <file>${log.path}/all.log</file>
        <!--日志文档输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset> <!-- 此处设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/all-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文档保留天数-->
            <maxHistory>15</maxHistory>
        </rollingPolicy>
        <!-- 此日志文档记录除了DEBUG级别的其它高于DEBUG的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>DEBUG</level>
	    <!--抓到该级别的就拦截-->
            <onMatch>DENY</onMatch>
	    <!-- 上面没抓到的就放行-->
            <onMismatch>ACCEPT</onMismatch>
        </filter>
    </appender>

    <!--
        <logger>用来设置某一个包或者具体的某一个类的日志打印级别、以及指定<appender>。<logger>仅有一个name属性，一个可选的level和一个可选的addtivity属性。
        name:用来指定受此logger约束的某一个包或者具体的某一个类。
        level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，
              还有一个特殊值INHERITED或者同义词NULL，代表强制执行上级的级别。
              如果未设置此属性，那么当前logger将会继承上级的级别。
        addtivity:是否向上级logger传递打印信息。默认是true。
        <logger name="org.springframework.web" level="info"/>
        <logger name="org.springframework.scheduling.annotation.ScheduledAnnotationBeanPostProcessor" level="INFO"/>
    -->

    <!--
	root配置必须在appender下边
        root节点是必选节点，用来指定最基础的日志输出级别，只有一个level属性
        level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，
        不能设置为INHERITED或者同义词NULL。默认是DEBUG
        可以包含零个或多个元素，标识这个appender将会添加到这个logger。
    -->

    <!-- 最终的策略：基本策略(root级) + 根据profile在启动时, logger标签中定制化package日志级别(优先级高于上面的root级)-->
    <springProfile name="dev">
	<!-- 大于等于info级别的才会输出 -->
        <root level="info">
            <appender-ref ref="CONSOLE" />
            <appender-ref ref="DEBUG_FILE" />
            <appender-ref ref="INFO_FILE" />
            <appender-ref ref="WARN_FILE" />
            <appender-ref ref="ERROR_FILE" />
            <appender-ref ref="ALL_FILE" />
        </root>
        <logger name="com.shen.test" level="debug"/> <!-- 开发环境, 指定某包日志为debug级 -->
    </springProfile>

    <springProfile name="test">
        <root level="info">
            <appender-ref ref="CONSOLE" />
            <appender-ref ref="DEBUG_FILE" />
            <appender-ref ref="INFO_FILE" />
            <appender-ref ref="WARN_FILE" />
            <appender-ref ref="ERROR_FILE" />
            <appender-ref ref="ALL_FILE" />
        </root>
        <logger name="com.shen.test" level="info"/> <!-- 测试环境, 指定某包日志为info级 -->
    </springProfile>

    <springProfile name="pro">
        <root level="info">
            <!-- 生产环境最好不配置console写文件 -->
            <appender-ref ref="DEBUG_FILE" />
            <appender-ref ref="INFO_FILE" />
            <appender-ref ref="WARN_FILE" />
            <appender-ref ref="ERROR_FILE" />
            <appender-ref ref="ALL_FILE" />
        </root>
        <logger name="com.shen.test" level="warn"/> <!-- 生产环境, 指定某包日志为warn级 -->
        <logger name="com.shen.test.MyApplication" level="info"/> <!-- 特定某个类打印info日志, 比如application启动成功后的提示语 -->
    </springProfile>

</configuration>

```
## 8. web支持
### 1. starter-web
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

SpringBoot提供SpringMVC自动配置，在SpringMVC默认功能上添加额外特性：
-   引入了 ContentNegotiatingViewResolver 和 BeanNameViewResolver（视图解析器）
-   对包括 WebJars 在内的静态资源的支持
-   自动注册 Converter、GenericConverter 和 Formatter （转换器和格式化器）
-   对 HttpMessageConverters 的支持（Spring MVC 中用于转换 HTTP 请求和响应的消息转换器）
-   自动注册 MessageCodesResolver（用于定义错误代码生成规则）
-   支持对静态首页（index.html）的访问
-   自动使用 ConfigurableWebBindingInitializer

### 2. 静态资源映射
SpringBoot默认为我们提供了三种静态资源映射规则：
- WebJars映射
- 默认资源映射
- 静态首页映射

#### 1. WebJars映射
WebJars可以将Web前端资源（JS，CSS 等）打成一个个的Jar包，然后将这些Jar包部署到Maven中央仓库中进行统一管理，当SpringBoot项目中需要引入Web前端资源时，只需要访问WebJars官网`https://www.webjars.org/`，找到所需资源的pom依赖，将其导入到项目中即可

所有通过WebJars引入的前端资源都存放在当前项目类路径（classpath）下的“/META-INF/resources/webjars/”目录中

SpringBoot通过MVC的自动配置类WebMvcAutoConfiguration为这些WebJars前端资源提供了默认映射规则，WebJars的映射路径为`/webjars/**`，即所有访问`/webjars/**`的请求，都会去`classpath:/META-INF/resources/webjars/`查找WebJars前端资源

#### 2. 默认映射
当访问项目中的任意资源（即“/\*\*”）时，SpringBoot会默认从以下路径中查找资源文件（优先级依次降低）：
1.  `classpath:/META-INF/resources/`
2.  `classpath:/resources/`
3.  `classpath:/static/`
4.  `classpath:/public/`

#### 3. 静态首页映射
静态资源文件夹下的所有`index.html`被称为静态首页或者欢迎页，它们会被被`/**`映射，换句话说就是，当我们访问`/`或者`/index.html`时，都会跳转到静态首页（欢迎页）

### 3. Thymeleaf
```xml
<!--Thymeleaf 启动器-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```
添加依赖后再使用@ThymeleafAutoConfiguration引入Thymeleaf相关自动配置

注：Thymeleaf模板的默认位置在`resources/templates`目录下，默认的后缀是`html`，即只要将HTML页面放在`classpath:/templates/`下，Thymeleaf就能自动进行渲染

### 4. 定制SpringMVC

### 5. 国际化
在`src/main/resources`目录下创建一个`i18n`目录，目录中创建国际化资源文件，其命名格式为基本名_语言代码_国家或地区代码，最后添加`key=value`键值对
<img src="D:\Project\IT-notes\框架or中间件\SpringBoot\img\国际化.png" style="width:400px;height:200px;" />

### 6. 拦截器
1. 定义拦截器，创建一个实现了`HandlerInterceptor`接口的拦截器实现类，并重写`preHandle`、`postHandle`、`afterCompletion`方法
2. 注册拦截器，创建一个继承`WebMvcConfigurerAdapter`类的配置类（需要使用@Configuration注解注释）,并重写`addInterceptors`方法
```java
@Configuration
public class MyInterceptorConfig extends WebMvcConfigurationSupport {
    @Override
    protected void addInterceptors(InterceptorRegistry registry) {
        // 将上面自定义好的拦截器添加进去。
        registry.addInterceptor(new MyInterceptor()).addPathPatterns("/**");
        super.addInterceptors(registry);
    }
}
```
3. 指定拦截规则，使用`addPathPatterns`方法添加拦截匹配路径

### 7. 默认异常处理机制
SpringBoot提供了一套默认的异常处理机制，一旦程序中出现了异常，SpringBoot会自动识别客户端的类型（浏览器客户端或机器客户端），并根据客户端的不同，以不同的形式展示异常信息
如果需要所有异常出现时跳转到某一自定义错误页面，则可以在静态资源路径中创建相应错误页面：error.html、503.html、404.html等

SpringBoot通过配置类`ErrorMvcAutoConfiguration`对异常处理提供了自动配置，该配置类向容器中注入了以下4个组件
- `ErrorPageCustomizer`：该组件会在在系统发生异常后，默认将请求转发到`/error`上
- `BasicErrorController`：处理默认的`/error`请求
- `DefaultErrorViewResolver`：默认的错误视图解析器，将异常信息解析到相应的错误视图上
- `DefaultErrorAttributes`：用于页面上共享异常信息

#### 1. ErrorPageCustomizer
```java
//ErrorMvcAutoConfiguration向容器中注入了一个名为ErrorPageCustomizer的组件，它主要用于定制错误页面的响应规则
@Bean
public ErrorPageCustomizer errorPageCustomizer(DispatcherServletPath dispatcherServletPath) {
    return new ErrorPageCustomizer(this.serverProperties, dispatcherServletPath);
}

//ErrorPageCustomizer通过registerErrorPages()方法来注册错误页面的响应规则。当系统中发生异常后，ErrorPageCustomizer组件会自动生效，并将请求转发到“/error”上，交给BasicErrorController进行处理
@Override
public void registerErrorPages(ErrorPageRegistry errorPageRegistry) {
    //将请求转发到 /errror（this.properties.getError().getPath()）上
    ErrorPage errorPage = new ErrorPage(this.dispatcherServletPath.getRelativePath(this.properties.getError().getPath()));
    // 注册错误页面
    errorPageRegistry.addErrorPages(errorPage);
}
```

#### 2. BasicErrorController
```java
//ErrorMvcAutoConfiguration注入BasicErrorController源码
@Bean
@ConditionalOnMissingBean(value = ErrorController.class, search = SearchStrategy.CURRENT)
public BasicErrorController basicErrorController(ErrorAttributes errorAttributes,
                                                 ObjectProvider<ErrorViewResolver> errorViewResolvers) {
    return new BasicErrorController(errorAttributes, this.serverProperties.getError(),
            errorViewResolvers.orderedStream().collect(Collectors.toList()));
}

//BasicErrorController源码
//BasicErrorController 用于处理 “/error” 请求
@Controller
@RequestMapping("${server.error.path:${error.path:/error}}")
public class BasicErrorController extends AbstractErrorController {
    ......
    /**
     * 该方法用于处理浏览器客户端的请求发生的异常
     * 生成 html 页面来展示异常信息
     * @param request
     * @param response
     * @return
     */
    @RequestMapping(produces = MediaType.TEXT_HTML_VALUE)
    public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
        //获取错误状态码
        HttpStatus status = getStatus(request);
        //getErrorAttributes 根据错误信息来封装一些 model 数据，用于页面显示
        Map<String, Object> model = Collections
                .unmodifiableMap(getErrorAttributes(request, getErrorAttributeOptions(request, MediaType.TEXT_HTML)));
        //为响应对象设置错误状态码
        response.setStatus(status.value());
        //调用 resolveErrorView() 方法，使用错误视图解析器生成 ModelAndView 对象（包含错误页面地址和页面内容）
        ModelAndView modelAndView = resolveErrorView(request, response, status, model);
        return (modelAndView != null) ? modelAndView : new ModelAndView("error", model);
    }

    /**
     * 该方法用于处理机器客户端的请求发生的错误
     * 产生 JSON 格式的数据展示错误信息
     * @param request
     * @return
     */
    @RequestMapping
    public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
        HttpStatus status = getStatus(request);
        if (status == HttpStatus.NO_CONTENT) {
            return new ResponseEntity<>(status);
        }
        Map<String, Object> body = getErrorAttributes(request, getErrorAttributeOptions(request, MediaType.ALL));
        return new ResponseEntity<>(body, status);
    }
    ......
}

protected ModelAndView resolveErrorView(HttpServletRequest request, HttpServletResponse response, HttpStatus status,
                                        Map<String, Object> model) {
    //获取容器中的所有的错误视图解析器来处理该异常信息
    for (ErrorViewResolver resolver : this.errorViewResolvers) {
        //调用错误视图解析器的 resolveErrorView 解析到错误视图页面
        ModelAndView modelAndView = resolver.resolveErrorView(request, status, model);
        if (modelAndView != null) {
            return modelAndView;
        }
    }
    return null;
}
```

#### 3. DefaultErrorViewResolver
```java
//ErrorMvcAutoConfiguration注入DefaultErrorViewResolver源码
@Bean
@ConditionalOnBean(DispatcherServlet.class)
@ConditionalOnMissingBean(ErrorViewResolver.class)
DefaultErrorViewResolver conventionErrorViewResolver() {
    return new DefaultErrorViewResolver(this.applicationContext, this.resources);
}

//DefaultErrorViewResolver源码
public class DefaultErrorViewResolver implements ErrorViewResolver, Ordered {

    private static final Map<HttpStatus.Series, String> SERIES_VIEWS;

    static {
        Map<HttpStatus.Series, String> views = new EnumMap<>(HttpStatus.Series.class);
        views.put(Series.CLIENT_ERROR, "4xx");
        views.put(Series.SERVER_ERROR, "5xx");
        SERIES_VIEWS = Collections.unmodifiableMap(views);
    }

    ......

    @Override
    public ModelAndView resolveErrorView(HttpServletRequest request, HttpStatus status, Map<String, Object> model) {
        //尝试以错误状态码作为错误页面名进行解析
        ModelAndView modelAndView = resolve(String.valueOf(status.value()), model);
        if (modelAndView == null && SERIES_VIEWS.containsKey(status.series())) {
            //尝试以 4xx 或 5xx 作为错误页面页面进行解析
            modelAndView = resolve(SERIES_VIEWS.get(status.series()), model);
        }
        return modelAndView;
    }

    private ModelAndView resolve(String viewName, Map<String, Object> model) {
        //错误模板页面，例如 error/404、error/4xx、error/500、error/5xx
        String errorViewName = "error/" + viewName;
        //当模板引擎可以解析这些模板页面时，就用模板引擎解析
        TemplateAvailabilityProvider provider = this.templateAvailabilityProviders.getProvider(errorViewName,
                this.applicationContext);
        if (provider != null) {
            //在模板能够解析到模板页面的情况下，返回 errorViewName 指定的视图
            return new ModelAndView(errorViewName, model);
        }
        //若模板引擎不能解析，则去静态资源文件夹下查找 errorViewName 对应的页面
        return resolveResource(errorViewName, model);
    }

    private ModelAndView resolveResource(String viewName, Map<String, Object> model) {
        //遍历所有静态资源文件夹
        for (String location : this.resources.getStaticLocations()) {
            try {
                Resource resource = this.applicationContext.getResource(location);
                //静态资源文件夹下的错误页面，例如error/404.html、error/4xx.html、error/500.html、error/5xx.html
                resource = resource.createRelative(viewName + ".html");
                //若静态资源文件夹下存在以上错误页面，则直接返回
                if (resource.exists()) {
                    return new ModelAndView(new DefaultErrorViewResolver.HtmlResourceView(resource), model);
                }
            } catch (Exception ex) {
            }
        }
        return null;
    }
    ......
}

/**

DefaultErrorViewResolver 解析异常信息的步骤如下：

1.  根据错误状态码（例如 404、500、400 等），生成一个错误视图 error/status，例如 error/404、error/500、error/400
2.  尝试使用模板引擎解析 error/status 视图，即尝试从 classpath 类路径下的 templates 目录下，查找 error/status.html，例如 error/404.html、error/500.html、error/400.html
3.  若模板引擎能够解析到 error/status 视图，则将视图和数据封装成 ModelAndView 返回并结束整个解析流程，否则跳转到第 4 步
4.  依次从各个静态资源文件夹中查找 error/status.html，若在静态文件夹中找到了该错误页面，则返回并结束整个解析流程，否则跳转到第 5 步
5.  将错误状态码（例如 404、500、400 等）转换为 4xx 或 5xx，然后重复前 4 个步骤，若解析成功则返回并结束整个解析流程，否则跳转第 6 步
6.  处理默认的 “/error ”请求，使用 Spring Boot 默认的错误页面（Whitelabel Error Page）

**/
```

#### 4. DefaultErrorAttributes
```java
//ErrorMvcAutoConfiguration注入DefaultErrorAttributes源码
@Bean
@ConditionalOnMissingBean(value = ErrorAttributes.class, search = SearchStrategy.CURRENT)
public DefaultErrorAttributes errorAttributes() {
    return new DefaultErrorAttributes();
}

//DefaultErrorAttributes源码
public class DefaultErrorAttributes implements ErrorAttributes, HandlerExceptionResolver, Ordered {
    ......
    @Override
    public Map<String, Object> getErrorAttributes(WebRequest webRequest, ErrorAttributeOptions options) {
        Map<String, Object> errorAttributes = getErrorAttributes(webRequest, options.isIncluded(Include.STACK_TRACE));
        if (!options.isIncluded(Include.EXCEPTION)) {
            errorAttributes.remove("exception");
        }
        if (!options.isIncluded(Include.STACK_TRACE)) {
            errorAttributes.remove("trace");
        }
        if (!options.isIncluded(Include.MESSAGE) && errorAttributes.get("message") != null) {
            errorAttributes.remove("message");
        }
        if (!options.isIncluded(Include.BINDING_ERRORS)) {
            errorAttributes.remove("errors");
        }
        return errorAttributes;
    }

    private Map<String, Object> getErrorAttributes(WebRequest webRequest, boolean includeStackTrace) {
        Map<String, Object> errorAttributes = new LinkedHashMap<>();
        errorAttributes.put("timestamp", new Date());
        addStatus(errorAttributes, webRequest);
        addErrorDetails(errorAttributes, webRequest, includeStackTrace);
        addPath(errorAttributes, webRequest);
        return errorAttributes;
    }
    ......
}
```


### 8. 全局异常处理
1. @ExceptionHandler
2. @ControllerAdvice+@ExceptionHandler
3. 注册SimpleMappingExceptionResolver
4. 自定义HandlerExceptinResolver接口实现类

### 9. 注册原生web-bean
1. @WebServlet
2. @WebFilter
3. @WebListener
注：需要使用@ServletComponentScan扫描

若不使用注解，可以使用相应组件的RegistrationBean注册注入
```java
@Configuration
public class MyConfig {
    /**
     * 注册 servlet
     * @return
     */
    @Bean
    public ServletRegistrationBean servletRegistrationBean() {
        MyServlet myServlet = new MyServlet();
        return new ServletRegistrationBean(myServlet, "/myServlet");
    }

    /**
     * 注册过滤器
     * @return
     */
    @Bean
    public FilterRegistrationBean filterRegistrationBean() {
        MyFiler myFiler = new MyFiler();
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean(myFiler);
        //注册该过滤器需要过滤的 url
        filterRegistrationBean.setUrlPatterns(Arrays.asList("/myServlet"));
        return filterRegistrationBean;
    }

    /**
     * 注册监听器
     * @return
     */
    @Bean
    public ServletListenerRegistrationBean servletListenerRegistrationBean() {
        MyListener myListener = new MyListener();
        return new ServletListenerRegistrationBean(myListener);
    }
}
```

### 10. JDBC
```xml
<!--导入JDBC的场景启动器-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>

<!--导入数据库驱动-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

```yml
#数据源连接信息
spring:
  datasource:
    username: root
    password: root
    url: jdbc:mysql://127.0.0.1:3306/bianchengbang_jdbc
    driver-class-name: com.mysql.cj.jdbc.Driver
```

最后使用`@Autowired JdbcTemplate jdbcTemplate`

### 11. 数据源注入原理
```yml
spring:
	datasource:
		type:com.alibaba.druid.pool.DruidDataSource
		driver-class-name:com.mysql.cj.jdbc.Driver
		url:jdbc:mysql://localhost:3306/test?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
		username:root 
		password:root
```

`@SpringBootApplication`中存在的`@EnableAutoConfiguration`实现了大量的默认自动类配置，其中的自动类源于`spring-boot-autoconfigure`包中`/META-INF/spring.factories`文件，文件中`key=auto configure`的一列类全名

而这些自动配置类中包含`org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration`实现数据源的自动配置。在该类中，默认支持两种类型的数据源`EmbeddedDatabase内嵌数据源、PooledDataSource池化数据源`：
```java
@Configuration(proxyBeanMethods = false)
@Conditional(EmbeddedDatabaseCondition.class)
@ConditionalOnMissingBean({DataSource.class, XADataSource.class})
@Import(EmbeddedDataSourceConfiguration.class)
protected static class EmbeddedDatabaseConfiguration {
}

@Configuration(proxyBeanMethods = false)
@Conditional(PooledDataSourceCondition.class)
@ConditionalOnMissingBean({DataSource.class, XADataSource.class})
@Import({DataSourceConfiguration.Hikari.class, DataSourceConfiguration.Tomcat.class, DataSourceConfiguration.Dbcp2.class, DataSourceConfiguration.OracleUcp.class, DataSourceConfiguration.Generic.class, DataSourceJmxConfiguration.class})
protected static class PooledDataSourceConfiguration {
}
```

根据两种数据源源码的@Conditional注入条件得出
1. 池化数据源条件：配置了`spring.datasource.type`；加载`hikari.HikariDataSource`、`tomcat.jdbc.pool.DataSource`、`commons.dbcp2.BasicDataSource`、`oracle.ucp.jdbc.PoolDataSourceImpl`
2. 内嵌数据源条件：未配置`spring.datasource.url`；不满足池化数据源条件
```java
//DruidDataSourceAutoConfigure源码
@Configuration
@ConditionalOnClass(DruidDataSource.class)
@AutoConfigureBefore(DataSourceAutoConfiguration.class)
@EnableConfigurationProperties({DruidStatProperties.class, DataSourceProperties.class})
@Import({DruidSpringAopConfiguration.class,
    DruidStatViewServletConfiguration.class,
    DruidWebStatFilterConfiguration.class,
    DruidFilterConfiguration.class})
public class DruidDataSourceAutoConfigure {

    private static final Logger LOGGER = LoggerFactory.getLogger(DruidDataSourceAutoConfigure.class);

    @Bean(initMethod = "init")
    @ConditionalOnMissingBean
    public DataSource dataSource() {
        LOGGER.info("Init DruidDataSource");
        return new DruidDataSourceWrapper();
    }
}
```

### 12. 整合Druid数据源
#### 1. 手动整合
```xml
<!--导入 JDBC 场景启动器-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jdbc</artifactId>
</dependency>
<!--导入数据库驱动-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
<!--采用自定义方式整合 druid 数据源-->
<!--自定义整合需要编写一个与之相关的配置类-->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.6</version>
</dependency>
```

```java
@Configuration
public class MyDataSourceConfig {
    /**
     * 当向容器中添加了 Druid 数据源
   * 使用 @ConfigurationProperties 将配置文件中 spring.datasource 开头的配置与数据源中的属性进行绑定
     * @return
     */
    @ConfigurationProperties("spring.datasource")
    @Bean
    public DataSource dataSource() throws SQLException {
        DruidDataSource druidDataSource = new DruidDataSource();
        //我们一般不建议将数据源属性硬编码到代码中，而应该在配置文件中进行配置（@ConfigurationProperties 绑定）
//        druidDataSource.setUrl("jdbc:mysql://127.0.0.1:3306/bianchengbang_jdbc");
//        druidDataSource.setUsername("root");
//        druidDataSource.setPassword("root");
//        druidDataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        return druidDataSource;
    }
}
```

```yaml
#数据源连接信息，此为jdbc通用配置
spring:
  datasource:
    username: root
    password: root
    url: jdbc:mysql://127.0.0.1:3306/bianchengbang_jdbc
    driver-class-name: com.mysql.cj.jdbc.Driver
```

#### 2. starter整合
```xml
<!--添加 druid 的 starter-->
<dependency>
   <groupId>com.alibaba</groupId>
   <artifactId>druid-spring-boot-starter</artifactId>
   <version>1.1.17</version>
</dependency>
```

```yml
spring:
	datasource:
		#通用jdbc配置
		username: root
	    password: root
	    url: jdbc:mysql://127.0.0.1:3306/bianchengbang_jdbc
	    driver-class-name: com.mysql.cj.jdbc.Driver
		
		#连接池配置
	    druid:
			initial-size: 5 #初始化连接大小
			min-idle: 5 #最小连接池数量
			max-active: 20 #最大连接池数量
			max-wait: 60000 #获取连接时最大等待时间，单位毫秒
			time-between-eviction-runs-millis: 60000 #配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
			min-evictable-idle-time-millis: 300000 #配置一个连接在池中最小生存的时间，单位是毫秒
			validation-query: SELECT 1 FROM DUAL #测试连接
			test-while-idle: true #申请连接的时候检测，建议配置为true，不影响性能，并且保证安全性
			test-on-borrow: false #获取连接时执行检测，建议关闭，影响性能
			test-on-return: false #归还连接时执行检测，建议关闭，影响性能
			pool-prepared-statements: false #是否开启PSCache，PSCache对支持游标的数据库性能提升巨大，oracle建议开启，mysql下建议关闭
			max-pool-prepared-statement-per-connection-size: 20 #开启poolPreparedStatements后生效
			filters: stat,wall #配置扩展插件，常用的插件有=>stat:监控统计  wall:防御sql注入
			connection-properties:'druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000' #通过connectProperties属性来打开mergeSql功能;慢SQL记录
			
			
		#监控配置
			#StatViewServlet配置，说明请参考Druid Wiki，配置_StatViewServlet配置
			stat-view-servlet:
				enabled: true #是否开启内置监控页面，默认值为 false
				url-pattern: '/druid/*' #StatViewServlet 的映射路径，即内置监控页面的访问地址
				reset-enable: true #是否启用重置按钮
				login-username: admin #内置监控页面的登录页用户名 username
				login-password: admin #内置监控页面的登录页密码 password
			
			# WebStatFilter配置，说明请参考Druid Wiki，配置_配置WebStatFilter
			web-stat-filter:
				enabled: true #是否开启内置监控中的 Web-jdbc 关联监控的数据
				url-pattern: '/*' #匹配路径
				exclusions: '*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*' #排除路径
				session-stat-enable: true #是否监控session
				
				# Spring监控配置，说明请参考Druid Github Wiki，配置_Druid和Spring关联监控配置
				aop-patterns: net.biancheng.www.*
		
		#Filter配置
			# 对配置已开启的 filters 即 stat(sql 监控)  wall（防火墙）
			filter:
				#配置StatFilter (SQL监控配置)
				stat:
					enabled: true #开启 SQL 监控
					slow-sql-millis: 1000 #慢查询
					log-slow-sql: true #记录慢查询 SQL
					
				#配置WallFilter (防火墙配置)
				wall:
					enabled: true #开启防火墙
					config:
						update-allow: true #允许更新操作
						drop-table-allow: false #禁止删表操作
						insert-allow: true #允许插入操作
						delete-allow: true #删除数据操作
```

### 13. 整合Mybatis
```xml
<!--引入 mybatis-spring-boot-starter 的依赖-->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.2.0</version>
</dependency>
```

```yaml
###################################### MyBatis 配置######################################
mybatis:
	# 指定 mapper.xml 的位置
	mapper-locations: classpath:mybatis/mapper/*.xml
	#扫描实体类的位置,在此处指明扫描实体类的包，在 mapper.xml 中就可以不写实体类的全路径名
	type-aliases-package: net.biancheng.www.bean
	configuration:
		#默认开启驼峰命名法，可以不用设置该属性
		map-underscore-to-camel-case: true
```

## 9. Swagger接口文档生成
### 1. Swagger基本配置
1. 添加依赖
```xml
<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger2 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>

<!-- https://mvnrepository.com/artifact/io.springfox/springfox-swagger-ui -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>

```

2. 启动类添加注解`@EnableSwagger2`
3. 添加Swagger的`JavaConfig`
```java
@Configuration
public class SwaggerConfig {

    // 创建swagger bean
    @Bean
    public Docket docket() {
		
		// 配置swagger的docket的bean实例 
		Profiles profiles = Profiles.of("dev","test"); 
		// 通过environment.acceptsProfiles()判断是否指定的环境中，是，则为true 
		boolean flag = environment.acceptsProfiles(profiles);
		
        // Docket是swagger全局配置对象
        // DocumentationType：指定文档类型为swagger2
        return new Docket(DocumentationType.SWAGGER_2)
                // swagger信息
                .apiInfo(apiInfo())
                // swagger 扫描包配置
				// select()获取Docket中的选择器，返回ApiSelectorBuilder构造选择器，如扫描扫描包的注解
				// 配置是否开启swagger，若为false，则浏览器不能访问
				.enable(flag)
				.select()
				/**
				 * requestHandlerSelectors：请求处理选择器
				 * basePackage()：扫描指定包下的所有接口
				 * any()：扫描所有的包
				 * none()：不扫描
				 * withClassAnnotation()：扫描指定类上的注解，参数是一个注解的放射对象
				 * withMethodAnnotation()：扫描方法上的注解
				 */
				// 指定扫描器扫描的规则（断言）
				.apis(RequestHandlerSelectors.basePackage("com.iqiuq.swaggerdemo.controller"))
				/**
				 * pathSelectors：路径选择器，过滤路径
				 * ang()：选择所有路径
				 * none()：都不选择
				 * ant()：选择指定路径
				 * regex()：正则表达式
				 */
				.paths(PathSelectors.regex("/hello"))
				.build();
    }

    // swagger文档信息
    public ApiInfo apiInfo() {
        // 作者信息
        Contact contact = new Contact(
                // 文档发布者的名称
                "iqiuq",
                // 文档发布者的网站地址
                "https://iqiuq.gitee.io/qiuqblogs/",
                // 文档发布者的电子邮箱
                "qiuyonghui258@163.com"
        );
        return new ApiInfo(
                // 标题
                "iqiuq swagger api",
                // 文档描述
                "演好自己人生的剧本",
                // 版本号
                "1.0",
                // 服务组url地址
                "urn:tos", 
                // 作者信息
                contact,
                // 开源组织
                "Apache 2.0",
                // 开源地址
                "http://www.apache.org/licenses/LICENSE-2.0",
                // 集合
                new ArrayList()
        );
    }
}

```

### 2. Swagger常用注解
- ### @Api 
	类上注解，控制整个类生成接口信息的内容
- ### @ApiOperation
	方法的说明
- ### @ApiParam
	可以作用于方法参数和成员变量
- ### @ApiLgnore
	忽略，当前注解描述的方法或类型，不生成api文档
- ### @ApiImplicitParam和@ApiImplicitParams
	使用在方法上，描述方法的单个或一组参数
- ### @ApiModel和@ApiModelProperty
	描述一个实体类型，这个实体类型如果成为任何一个生成api帮助文档方法的一个返回值类型的时候，此注解被解析
	实体类属性描述
- ### @ApiResponse和@ApiResponses
	方法返回每个参数或对象的说明
- ### @Authorization、@AuthorizationScope、@ResponseHeader、@ApiProperty、@ApiError