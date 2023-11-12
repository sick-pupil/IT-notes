## 1. 基本概念

|基本概念|描述|
|---|---|
|资源|资源是`Sentinel`的关键概念。它可以是 Java 应用程序中的任何内容，例如由应用程序提供的服务或者是服务里的方法，甚至可以是一段代码。我们可以通过`Sentinel`提供的`API`来定义一个资源，使其能够被 `Sentinel`保护起来。通常情况下，我们可以使用方法名、`URL`甚至是服务名来作为资源名来描述某个资源。|
|规则|围绕资源而设定的规则。`Sentinel`支持流量控制、熔断降级、系统保护、来源访问控制和热点参数等多种规则，所有这些规则都可以动态实时调整。|

`@SentinelResource`注解是`Sentinel`提供的最重要的注解之一，它还包含了多个属性

|属性|说明|必填与否|使用要求|
|---|---|---|---|
|value|用于指定资源的名称|必填|-|
|entryType|entry 类型|可选项（默认为 EntryType.OUT）|-|
|blockHandler|服务限流后会抛出 BlockException 异常，而 blockHandler 则是用来指定一个函数来处理 BlockException  异常的。  <br>  <br>简单点说，该属性用于指定服务限流后的后续处理逻辑。|可选项|- blockHandler 函数访问范围需要是 public；<br>- 返回类型需要与原方法相匹配；<br>- 参数类型需要和原方法相匹配并且最后加一个额外的参数，类型为 BlockException；<br>- blockHandler 函数默认需要和原方法在同一个类中，若希望使用其他类的函数，则可以指定 blockHandler 为对应的类的 Class 对象，注意对应的函数必需为 static 函数，否则无法解析。|
|blockHandlerClass|若 blockHandler 函数与原方法不在同一个类中，则需要使用该属性指定 blockHandler 函数所在的类。|可选项|- 不能单独使用，必须与 blockHandler 属性配合使用；<br>- 该属性指定的类中的 blockHandler 函数必须为 static 函数，否则无法解析。|
|fallback|用于在抛出异常（包括 BlockException）时，提供 fallback 处理逻辑。  <br>  <br>fallback 函数可以针对所有类型的异常（除了 exceptionsToIgnore 里面排除掉的异常类型）进行处理。|可选项|- 返回值类型必须与原函数返回值类型一致；<br>- 方法参数列表需要和原函数一致，或者可以额外多一个 Throwable 类型的参数用于接收对应的异常；<br>- fallback 函数默认需要和原方法在同一个类中，若希望使用其他类的函数，则可以指定 fallbackClass 为对应的类的 Class 对象，注意对应的函数必需为 static 函数，否则无法解析。|
|fallbackClass|若 fallback 函数与原方法不在同一个类中，则需要使用该属性指定 blockHandler 函数所在的类。|可选项|- 不能单独使用，必须与 fallback 或 defaultFallback  属性配合使用；<br>- 该属性指定的类中的 fallback 函数必须为 static 函数，否则无法解析。|
|defaultFallback|默认的 fallback 函数名称，通常用于通用的 fallback 逻辑（即可以用于很多服务或方法）。  <br>  <br>默认 fallback 函数可以针对所以类型的异常（除了 exceptionsToIgnore 里面排除掉的异常类型）进行处理。|可选项|- 返回值类型必须与原函数返回值类型一致；<br>- 方法参数列表需要为空，或者可以额外多一个 Throwable 类型的参数用于接收对应的异常；<br>- defaultFallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 fallbackClass 为对应的类的 Class 对象，注意对应的函数必需为 static 函数，否则无法解析。|
|exceptionsToIgnore|用于指定哪些异常被排除掉，不会计入异常统计中，也不会进入 fallback 逻辑中，而是会原样抛出。|可选项|-|

## 2. 控制台
Sentinel 控制台提供的功能如下：
- **查看机器列表以及健康情况**：`Sentinel`控制台能够收集`Sentinel`客户端发送的心跳包，判断机器是否在线
- **监控（单机和集群聚合）**：`Sentinel`控制台通过`Sentinel`客户端暴露的监控`API`，可以实现秒级的实时监控
- **规则管理和推送**：通过`Sentinel`控制台，我们还能够针对资源定义和推送规则。
- **鉴权**：从`Sentinel 1.6.0`起，`Sentinel`控制台引入基本的登录功能，默认用户名和密码都是`sentinel`

下载`sentinel-dashboard-1.8.2.jar`并执行命令启动`java -jar sentinel-dashboard-1.8.2.jar`，访问`http://localhost:8080/`登录，输入`sentinel`作为用户名以及密码

## 3. 开发流程
`Sentinel`的开发流程如下：
1. **引入 Sentinel 依赖**：在项目中引入`Sentinel`的依赖，将`Sentinel`整合到项目中
2. **定义资源**：通过对主流框架提供适配或`Sentinel`提供的显式`API`和注解，可以定义需要保护的资源，此外`Sentinel`还提供了资源的实时统计和调用链路分析
3. **定义规则**：根据实时统计信息，对资源定义规则，例如流控规则、熔断规则、热点规则、系统规则以及授权规则等
4. **检验规则是否在生效**：运行程序，检验规则是否生效，查看效果

*spring-cloud-alibaba-sentinel-service-8401*
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
    <artifactId>spring-cloud-alibaba-sentinel-service-8401</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-cloud-alibaba-sentinel-service-8401</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <!--Nacos 服务发现依赖-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--Snetinel 依赖-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
        <!--SpringCloud ailibaba sentinel-datasource-nacos 后续做持久化用到-->
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>
       
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
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
  port: 8401 #端口

spring:
  application:
    name: sentinel-service #服务名
  cloud:
    nacos:
      discovery:
        #Nacos服务注册中心(集群)地址
        server-addr: localhost:1111

    sentinel:
      transport:
        #配置 Sentinel dashboard 地址
        dashboard: localhost:8080
        #默认8719端口，假如被占用会自动从8719开始依次+1扫描,直至找到未被占用的端口
        port: 8719

management:
  endpoints:
    web:
      exposure:
        include: '*'
```

## 4. 资源
`Sentinel`为我们提供了多种定义资源的方式：
- 适配主流框架自动定义资源
- 通过`SphU`手动定义资源
- 通过`SphO`手动定义资源
- 注解方式定义资源

```java
@GetMapping("/testC")
@SentinelResource(value = "testCbyAnnotation") //通过注解定义资源
public String testC() {
    log.info("c语言中文网提醒您，服务访问成功------testC：" + serverPort);
    return "c语言中文网提醒您，服务访问成功------testC：" + serverPort;
}
```

## 5. 流量控制
可以针对资源定义流控规则，`Sentinel`会根据这些规则对流量相关的各项指标进行监控。当这些指标当达到或超过流控规则规定的阈值时，`Sentinel`会对请求的流量进行限制（即“限流”），以避免系统被瞬时的流量高峰冲垮，保障系统的高可用性

|属性|说明|默认值|
|---|---|---|
|资源名|流控规则的作用对象。|-|
|阈值|流控的阈值。|-|
|阈值类型|流控阈值的类型，包括 QPS 或并发线程数。|QPS（每秒钟最多通过的请求数）|
|针对来源|流控针对的调用来源。|default，表示不区分调用来源|
|流控模式|调用关系限流策略，包括直接、链路和关联。|直接|
|流控效果|流控效果（直接拒绝、Warm Up、匀速排队），不支持按调用关系限流。|直接拒绝|

同一个资源可以创建多条流控规则，`Sentinel`会遍历这些规则，直到有规则触发限流或者所有规则遍历完毕为止
`Sentinel`触发限流时，资源会抛出`BlockException`异常，此时我们可以捕捉`BlockException`来自定义被限流之后的处理逻辑

1. 在`sentinel`控制台中定义某资源的限流规则
2. 通过代码定义限流后的处理
*通过`@SentinelResource`注解的`blockHandler`属性指定了一个`blockHandler`函数，进行限流之后的后续处理*
```java
/**
 * 通过 Sentinel 控制台定义流控规则
 *
 */
@GetMapping("/testD")
@SentinelResource(value = "testD-resource", blockHandler = "blockHandlerTestD") //通过注解定义资源
public String testD() {
    log.info("c语言中文网提醒您，服务访问成功------testD：" + serverPort);
    return "c语言中文网提醒您，服务访问成功------testD：" + serverPort;
}

/**
 * 限流之后的逻辑
 * @param exception
 * @return
 */
public String blockHandlerTestD(BlockException exception) {
    log.info(Thread.currentThread().getName() + "c语言中文网提醒您，TestD服务访问失败! 您已被限流，请稍后重试");
    return "c语言中文网提醒您，TestD服务访问失败! 您已被限流，请稍后重试";
}
```

使用`@SentinelResource`注解的`blockHandler`属性时，需要注意以下事项：
- `blockHandler`函数访问范围需要是`public`
- 返回类型需要与原方法相匹配
- 参数类型需要和原方法相匹配并且最后加一个额外的参数，类型为`BlockException`
- `blockHandler`函数默认需要和原方法在同一个类中，若希望使用其他类的函数，则可以指定`blockHandler`为对应的类的 Class 对象，注意对应的函数必需为`static`函数，否则无法解析
- 请务必添加`blockHandler`属性来指定自定义的限流处理方法，若不指定，则会跳转到错误页（用户体验不好）

3. 通过代码定义规则
还可以在服务代码中，调用`FlowRuleManager`类的`loadRules()`方法来定义流控规则，该方法需要一个`FlowRule`类型的`List`集合作为其参数
```java
public static void loadRules(List<FlowRule> rules) {
    currentProperty.updateValue(rules); 
}
```

|属性|说明|默认值|
|---|---|---|
|resource|资源名，即流控规则的作用对象|-|
|count|限流的阈值。|-|
|grade|流控阈值的类型，包括 QPS 或并发线程数|QPS|
|limitApp|流控针对的调用来源|default，表示不区分调用来源|
|strategy|调用关系限流策略，包括直接、链路和关联|直接|
|controlBehavior|流控效果（直接拒绝、Warm Up、匀速排队），不支持按调用关系限流|直接拒绝|

```java
@GetMapping("/testD")
@SentinelResource(value = "testD-resource", blockHandler = "blockHandlerTestD") //通过注解定义资源
public String testD() {
    initFlowRules(); //调用初始化流控规则的方法
    log.info("c语言中文网提醒您，服务访问成功------testD：" + serverPort);
    return "c语言中文网提醒您，服务访问成功------testD：" + serverPort;
}

/**
 * 通过代码定义流量控制规则
 */
private static void initFlowRules() {
    List<FlowRule> rules = new ArrayList<>();
    //定义一个限流规则对象
    FlowRule rule = new FlowRule();
    //资源名称
    rule.setResource("testD-resource");
     //限流阈值的类型
    rule.setGrade(RuleConstant.FLOW_GRADE_QPS);
    // 设置 QPS 的阈值为 2
    rule.setCount(2);
    rules.add(rule);
    //定义限流规则
    FlowRuleManager.loadRules(rules);
}
```

## 6. 熔断降级

|熔断策略|说明|
|---|---|
|慢调用比例  <br>(SLOW_REQUEST_RATIO）|选择以慢调用比例作为阈值，需要设置允许的慢调用 RT（即最大响应时间），若请求的响应时间大于该值则统计为慢调用。  <br>  <br>当单位统计时长（statIntervalMs）内请求数目大于设置的最小请求数目，且慢调用的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。  <br>  <br>经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求响应时间小于设置的慢调用 RT 则结束熔断，若大于设置的慢调用 RT 则再次被熔断。|
|异常比例 (ERROR_RATIO)|当单位统计时长（statIntervalMs）内请求数目大于设置的最小请求数目且异常的比例大于阈值，则在接下来的熔断时长内请求会自动被熔断。  <br>  <br>经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。异常比率的阈值范围是 [0.0, 1.0]，代表 0% - 100%。|
|异常数 (ERROR_COUNT)|当单位统计时长内的异常数目超过阈值之后会自动进行熔断。  <br>  <br>经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，否则会再次被熔断。|

<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\Sentinel熔断状态.png" style="width:700px;height:500px;" />

|熔断状态|说明|触发条件|
|---|---|---|
|熔断关闭状态  <br>（CLOSED）|处于关闭状态时，请求可以正常调用资源。|满足以下任意条件，Sentinel 熔断器进入熔断关闭状态：<br><br>- 全部请求访问成功。<br>- 单位统计时长（statIntervalMs）内请求数目小于设置的最小请求数目。<br>- 未达到熔断标准，例如服务超时比例、异常数、异常比例未达到阈值。<br>- 处于探测恢复状态时，下一个请求访问成功。|
|熔断开启状态  <br>（OPEN）|处于熔断开启状态时，熔断器会一定的时间（规定的熔断时长）内，暂时切断所有请求对该资源的调用，并调用相应的降级逻辑使请求快速失败避免系统崩溃。|满足以下任意条件，Sentinel 熔断器进入熔断开启状态：<br><br>- 单位统计时长内请求数目大于设置的最小请求数目，且已达到熔断标准，例如请求超时比例、异常数、异常比例达到阈值。<br>- 处于探测恢复状态时，下一个请求访问失败。|
|探测恢复状态  <br>（HALF-OPEN）|处于探测恢复状态时，Sentinel 熔断器会允许一个请求调用资源。则若接下来的一个请求成功完成（没有错误）则结束熔断，熔断器进入熔断关闭（CLOSED）状态；否则会再次被熔断，熔断器进入熔断开启（OPEN）状态。|在熔断开启一段时间（降级窗口时间或熔断时长，单位为 s）后，Sentinel 熔断器自动会进入探测恢复状态。|

|属性|说明|默认值|使用范围|
|---|---|---|---|
|资源名|规则的作用对象。|-|所有熔断策略|
|熔断策略|Sentinel 支持3 中熔断策略：慢调用比例、异常比例、异常数策略。|慢调用比例|所有熔断策略|
|最大 RT|请求的最大相应时间，请求的响应时间大于该值则统计为慢调用。|-|慢调用比例|
|熔断时长|熔断开启状态持续的时间，超过该时间熔断器会切换为探测恢复状态（HALF-OPEN），单位为 s。|-|所有熔断策略|
|最小请求数|熔断触发的最小请求数，请求数小于该值时即使异常比率超出阈值也不会熔断（1.7.0 引入）。|5|所有熔断策略|
|统计时长|熔断触发需要统计的时长（单位为 ms），如 60\*1000 代表分钟级（1.8.0 引入）。|1000 ms|所有熔断策略|
|比例阈值|分为慢调用比例阈值和异常比例阈值，即慢调用或异常调用占所有请求的百分比，取值范围 \[0.0,1.0\]。|-|慢调用比例 、异常比例|
|异常数|请求或调用发生的异常的数量。|-|异常数|

`Sentinel`实现熔断降级的步骤：
1. 在项目中，使用`@SentinelResource`注解的`fallback`属性可以为资源指定熔断降级逻辑（方法）
2. 通过`Sentinel`控制台或代码定义熔断规则，包括熔断策略、最小请求数、阈值、熔断时长以及统计时长等
3. 若单位统计时长（`statIntervalMs`）内，请求数目大于设置的最小请求数目且达到熔断标准（例如请求超时比例、异常数、异常比例达到阈值），`Sentinel`熔断器进入熔断开启状态（`OPEN`）
4. 处于熔断开启状态时，`@SentinelResource`注解的`fallback`属性指定的降级逻辑会临时充当主业务逻辑，而原来的主逻辑则暂时不可用。当有请求访问该资源时，会直接调用降级逻辑使请求快速失败，而不会调用原来的主业务逻辑
5. 在经过一段时间（在熔断规则中设置的熔断时长）后，熔断器会进入探测恢复状态（`HALF-OPEN`），此时`Sentinel`会允许一个请求对原来的主业务逻辑进行调用，并监控其调用结果
6. 若请求调用成功，则熔断器进入熔断关闭状态（`CLOSED`），服务原来的主业务逻辑恢复，否则重新进入熔断开启状态（`OPEN`）

*熔断的服务消费者配置*
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
    <artifactId>spring-cloud-alibaba-consumer-mysql-8803</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-cloud-alibaba-consumer-mysql-8803</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.cloud</groupId>
                    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--引入 OpenFeign 的依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-loadbalancer</artifactId>
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
        <!--SpringCloud ailibaba sentinel -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
        <dependency>
            <groupId>net.biancheng.c</groupId>
            <artifactId>spring-cloud-alibaba-api</artifactId>
            <version>${project.version}</version>
        </dependency>
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

```yml
server:
  port: 8803
spring:
  application:
    name: spring-cloud-alibaba-consumer-mysql-feign
  cloud:
    nacos:
      discovery:
        server-addr: localhost:1111
    sentinel:
      transport:
        dashboard: localhost:8080
        port: 8719

# 以下配置信息并不是默认配置，而是我们自定义的配置，目的是不在 Controller 内硬编码 服务提供者的服务名
service-url:
  nacos-user-service: http://spring-cloud-alibaba-provider-mysql #消费者要方位的微服务名称

# 激活Sentinel对Feign的支持
feign:
  sentinel:
    enabled: true
```

```java
package net.biancheng.c.controller;

import com.alibaba.csp.sentinel.annotation.SentinelResource;
import com.alibaba.csp.sentinel.slots.block.degrade.circuitbreaker.CircuitBreaker;
import com.alibaba.csp.sentinel.slots.block.degrade.circuitbreaker.EventObserverRegistry;
import com.alibaba.csp.sentinel.util.TimeUtil;
import lombok.extern.slf4j.Slf4j;
import net.biancheng.c.entity.CommonResult;
import net.biancheng.c.entity.Dept;
import net.biancheng.c.service.DeptFeignService;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

@RestController
@Slf4j
public class DeptFeignController {
    @Resource
    DeptFeignService deptFeignService;

    @RequestMapping(value = "consumer/feign/dept/get/{id}", method = RequestMethod.GET)
    @SentinelResource(value = "fallback", fallback = "handlerFallback")
    public CommonResult<Dept> get(@PathVariable("id") int id) {
        monitor();
        System.out.println("--------->>>>主业务逻辑");
        CommonResult<Dept> result = deptFeignService.get(id);
        if (id == 6) {
            System.err.println("--------->>>>主业务逻辑，抛出非法参数异常");
            throw new IllegalArgumentException("IllegalArgumentException，非法参数异常....");
            //如果查到的记录也是 null 也控制正异常
        } else if (result.getData() == null) {
            System.err.println("--------->>>>主业务逻辑，抛出空指针异常");
            throw new NullPointerException("NullPointerException，该ID没有对应记录,空指针异常");
        }
        return result;
    }

    @RequestMapping(value = "consumer/feign/dept/list", method = RequestMethod.GET)
    public CommonResult<List<Dept>> list() {
        return deptFeignService.list();
    }

    //处理异常的回退方法（服务降级）
    public CommonResult handlerFallback(@PathVariable int id, Throwable e) {
        System.err.println("--------->>>>服务降级逻辑");
        Dept dept = new Dept(id, "null", "null");
        return new CommonResult(444, "C语言中文网提醒您，服务被降级！异常信息为：" + e.getMessage(), dept);
    }

    /**
     * 自定义事件监听器，监听熔断器状态转换
     */
    public void monitor() {
        EventObserverRegistry.getInstance().addStateChangeObserver("logging",
                (prevState, newState, rule, snapshotValue) -> {
                    SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
                    if (newState == CircuitBreaker.State.OPEN) {
                        // 变换至 OPEN state 时会携带触发时的值
                        System.err.println(String.format("%s -> OPEN at %s, 发送请求次数=%.2f", prevState.name(),
                                format.format(new Date(TimeUtil.currentTimeMillis())), snapshotValue));
                    } else {
                        System.err.println(String.format("%s -> %s at %s", prevState.name(), newState.name(),
                                format.format(new Date(TimeUtil.currentTimeMillis()))));
                    }
                });
    }

}
```

使用`@SentinelResource`注解的`fallback`属性时，需要注意以下事项：  
- 返回值类型必须与原函数返回值类型一致
- 方法参数列表需要和原函数一致，或者可以额外多一个`Throwable`类型的参数用于接收对应的异常
- `fallback`函数默认需要和原方法在同一个类中，若希望使用其他类的函数，则可以指定`fallbackClass`为对应的类的`Class`对象，注意对应的函数必需为`static`函数，否则无法解析

可以通过`Sentinel`控制台定义降级规则，也可以通过代码定义降级规则：提供了的一个名为 `DegradeRuleManager`类，我们可以通过调用它的`loadRules()`方法来定义熔断降级规则，该方法需要一个`DegradeRule`类型的`List`参数
```java
public static void loadRules(List<DegradeRule> rules) {
    try {
        currentProperty.updateValue(rules);
    } catch (Throwable var2) {
        RecordLog.error("[DegradeRuleManager] Unexpected error when loading degrade rules", var2);
    }
}
```

|降级属性|说明|默认值|
|---|---|---|
|resource|资源名，即规则的作用对象||
|grade|熔断策略，支持慢调用比例/异常比例/异常数策略|慢调用比例|
|count|慢调用比例模式下为慢调用临界 RT（超出该值计为慢调用）；异常比例/异常数模式下为对应的阈值||
|timeWindow|熔断时长，单位为 s||
|minRequestAmount|熔断触发的最小请求数，请求数小于该值时即使异常比率超出阈值也不会熔断（1.7.0 引入）|5|
|statIntervalMs|统计时长（单位为 ms），如 60\*1000 代表分钟级（1.8.0 引入）|1000 ms|
|slowRatioThreshold|慢调用比例阈值，仅慢调用比例模式有效（1.8.0 引入）| |

```java
@RequestMapping(value = "consumer/feign/dept/get/{id}", method = RequestMethod.GET)
@SentinelResource(value = "fallback", fallback = "handlerFallback")
public CommonResult<Dept> get(@PathVariable("id") int id) {
    initDegradeRule();
    monitor();
    System.out.println("--------->>>>主业务逻辑");
    CommonResult<Dept> result = deptFeignService.get(id);
    if (id == 6) {
        System.err.println("--------->>>>主业务逻辑，抛出非法参数异常");
        throw new IllegalArgumentException("IllegalArgumentException，非法参数异常....");
        //如果查到的记录也是 null 也控制正异常
    } else if (result.getData() == null) {
        System.err.println("--------->>>>主业务逻辑，抛出空指针异常");
        throw new NullPointerException("NullPointerException，该ID没有对应记录,空指针异常");
    }
    return result;
}

/**
 * 初始化熔断策略
 */
private static void initDegradeRule() {
    List<DegradeRule> rules = new ArrayList<>();
    DegradeRule rule = new DegradeRule("fallback");
    //熔断策略为异常比例
    rule.setGrade(CircuitBreakerStrategy.ERROR_RATIO.getType());
    //异常比例阈值
    rule.setCount(0.7);
    //最小请求数
    rule.setMinRequestAmount(100);
    //统计市场，单位毫秒
    rule.setStatIntervalMs(30000);
    //熔断市场，单位秒
    rule.setTimeWindow(10);
    rules.add(rule);
    DegradeRuleManager.loadRules(rules);
}
```