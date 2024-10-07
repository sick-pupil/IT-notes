## 1. 概念
为了保证多数据库之间，数据的正确性和一致性，我们必须保证所有这些操作要么全部成功，要么全部失败

分布式事务主要涉及以下概念：
- **事务**：由一组操作构成的可靠、独立的工作单元，事务具备`ACID`的特性，即原子性、一致性、隔离性和持久性
- **本地事务**：本地事务由本地资源管理器（通常指数据库管理系统`DBMS`，例如`MySQL`、`Oracle`等）管理，严格地支持`ACID`特性，高效可靠。本地事务不具备分布式事务的处理能力，隔离的最小单位受限于资源管理器，即本地事务只能对自己数据库的操作进行控制，对于其他数据库的操作则无能为力
- **全局事务**：全局事务指的是一次性操作多个资源管理器完成的事务，由一组分支事务组成
- **分支事务**：在分布式事务中，就是一个个受全局事务管辖和协调的本地事务

`Seata`对分布式事务的协调和控制，主要是通过`XID`和3个核心组件实现的：
- `XID`：全局事务的唯一标识，它可以在服务的调用链路中传递，绑定到服务的事务上下文中
- 核心组件：
	- `TC`：`Transaction Coordinator`，事务协调器，指的`seata`服务器，负责维护全局事务和分支事务的状态，驱动全局事务提交或回滚
	- `TM`：`Transaction Manager`，事务管理器，是事务的发起者，负责定义全局事务的范围，并根据`TC`维护的全局事务和分支事务状态，做出开始事务、提交事务、回滚事务的决议
	- `RM`：`Resource Manager`，资源管理器，是资源的管理者（这里可以将其理解为各服务使用的数据库）。它负责管理分支事务上的资源，向`TC`注册分支事务，汇报分支事务状态，驱动分支事务的提交或回滚

<img src="D:\Project\IT-notes\框架or中间件\SpringCloud\img\Seata工作流程.png" style="width:700px;height:500px;" />

`Seata`的整体工作流程如下：
1. `TM`向`TC`申请开启一个全局事务，全局事务创建成功后，`TC`会针对这个全局事务生成一个全局唯一的`XID`
2. `XID`通过服务的调用链传递到其他服务
3. `RM`向`TC`注册一个分支事务，并将其纳入`XID`对应全局事务的管辖
4. `TM`根据`TC`收集的各个分支事务的执行结果，向`TC`发起全局事务提交或回滚决议
5. `TC`调度`XID`下管辖的所有分支事务完成提交或回滚操作

`Seata`提供了`AT`、`TCC`、`SAGA`和`XA`四种事务模式，可以快速有效地对分布式事务进行控制

## 2. AT模式
任何应用想要使用`Seata`的`AT`模式对分布式事务进行控制，必须满足以下两个前提：
- 必须使用支持本地`ACID`事务特性的关系型数据库，例如`MySQL`、`Oracle`等
- 应用程序必须是使用`JDBC`对数据库进行访问的`JAVA`应用

此外，我们还需要针对业务中涉及的各个数据库表，分别创建一个`UNDO_LOG`（回滚日志）表。不同数据库在创建`UNDO_LOG`表时会略有不同，以`MySQL`为例：
```sql
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

`AT`工作流程：
1. 阶段一
	- 获取`SQL`基本信息，`seata`拦截并解析业务`SQL`，得到`SQL`操作类型、表名、判断条件等
	- 查询前镜像：根据得到的业务`SQL`，生成前镜像查询语句，并执行前镜像查询语句后得到的结果保存为“前镜像数据”
	- 执行业务`SQL`
	- 根据前镜像语句，生成后镜像查询语句，并执行，将得到的结果保存为“后镜像数据”
	- 插入回滚日志，插入创建的`UNDO_LOG`表
	- 注册分支事务，生成锁；在此次业务`SQL`对应的本地事务提交前，`RM`向`TC`注册分支事务，并针对记录生成锁
	- 本地事务提交，`SQL`数据更新，并提交之前生成的`UNDO_LOG`
	- 将本地事务提交结果上报`TC`
2. 阶段二
	- 当所有的`RM`都将自己分支事务的提交结果上报给`TC`后，`TM`根据`TC`收集的各个分支事务的执行结果，来决定向`TC`发起全局事务的提交或回滚；若所有分支事务都执行成功，`TM`向`TC`发起全局事务的提交，并批量删除各个`RM`保存的`UNDO_LOG`记录和行锁；否则全局事务回滚
	- 若全局事务中的任何一个分支事务失败，则`TM`向`TC`发起全局事务的回滚，并开启一个本地事务，执行如下操作：
		1. 查找`UNDO_LOG`记录：通过`XID`和分支事务`ID`查找所有的`UNDO_LOG`记录
		2. 数据校验：将`UNDO_LOG`中的后镜像数据（`afterImage`）与当前数据进行比较，如果有不同，则说明数据被当前全局事务之外的动作所修改
		3. 生成回滚语句：根据`UNDO_LOG`中的前镜像（`beforeImage`）和业务`SQL`的相关信息生成回滚语句
		4. 还原数据：执行回滚语句，并将前镜像数据、后镜像数据以及行锁删除
		5. 提交事务：提交本地事务，并把本地事务的执行结果（即分支事务回滚的结果）上报给`TC`

## 3. 使用
下载`seata-server-1.4.2.zip`，解压后得到`bin、conf、lib、logs`四个主要目录：
- `bin`：用于存放`Seata Server`可执行命令
- `conf`：用于存放`Seata Server`的配置文件
- `lib`：用于存放`Seata Server`依赖的各种`Jar`包
- `logs`：用于存放`Seata Server`的日志

### 1. 整合Nacos配置中心
```xml
<!--引入 seata 依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
</dependency>
```

在`Seata Server`安装目录下的`config/registry.conf`中，将配置方式（`config.type`）修改为`Nacos`，并对`Nacos`配置中心的相关信息进行配置
```
config {
  # Seata 支持 file、nacos 、apollo、zk、consul、etcd3 等多种配置中心
  #配置方式修改为 nacos
  type = "nacos"

  nacos {
    #修改为使用的 nacos 服务器地址
    serverAddr = "127.0.0.1:1111"
    #配置中心的命名空间
    namespace = ""
    #配置中心所在的分组
    group = "SEATA_GROUP"
    #Nacos 配置中心的用户名
    username = "nacos"
    #Nacos 配置中心的密码
    password = "nacos"
  }
}
```

`Seata Client application.yml`对`nacos`配置中心进行配置
```yml
seata:
  config:
    type: nacos
    nacos:
      server-addr: 127.0.0.1:1111 # Nacos 配置中心的地址
      group : "SEATA_GROUP"  #分组
      namespace: ""
      username: "nacos"   #Nacos 配置中心的用于名
      password: "nacos"  #Nacos 配置中心的密码
```

将`Seata`配置上传至`Nacos`配置中心
1. `Seata`目录中的`/script/config-center`获取`config.txt`，并根据需要修改其中配置
2. 在`/script/config-center/nacos`目录中，存在`Seata`脚本：`nacos-config.py`与`nacos-config.sh`
3. 执行上传命令：`sh nacos-config.sh -h 127.0.0.1 -p 1111  -g SEATA_GROUP -u nacos -w nacos`
	- `-h`：`Nacos`的`host`
	- `-p`：端口号
	- `-g`：`Nacos`配置的分组
	- `-u`：`Nacos`用户名
	- `-w`：`Nacos`密码

### 2. 整合Nacos注册中心
```xml
<!--引入 seata 依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
</dependency>
```

在`Seata Server`安装目录下的`config/registry.conf`中，将注册方式（`registry.type`）修改为`Nacos`，并对`Nacos`注册中心的相关信息进行配置
```
registry {
  # Seata 支持 file 、nacos 、eureka、redis、zk、consul、etcd3、sofa 作为其注册中心
  # 将注册方式修改为 nacos
  type = "nacos"

  nacos {
    application = "seata-server"
    # 修改 nacos 注册中心的地址
    serverAddr = "127.0.0.1:1111"
    group = "SEATA_GROUP"
    namespace = ""
    cluster = "default"
    username = ""
    password = ""
  }
}
```

`Seata Client application.yml`对`nacos`注册中心进行配置
```yml
seata:
  registry:
    type: nacos
    nacos:
      application: seata-server  
      server-addr: 127.0.0.1:1111 # Nacos 注册中心的地址
      group : "SEATA_GROUP" #分组
      namespace: ""   
      username: "nacos"   #Nacos 注册中心的用户名
      password: "nacos"   # Nacos 注册中心的密码
```

### 3. 存储模式

|模式|说明|准备工作|
|---|---|---|
|file|文件存储模式，默认存储模式；  <br>  <br>该模式为单机模式，全局事务的会话信息在内存中读写，并持久化本地文件 root.data，性能较高|-|
|db|数据库存储模式；  <br>  <br>该模式为高可用模式，全局事务会话信息通过数据库共享，性能较低。|建数据库表|
|redis|缓存处处模式；  <br>  <br>Seata Server 1.3 及以上版本支持该模式，性能较高，但存在事务信息丢失风险,|配置 redis 持久化配置|

在`db`模式下，我们需要针对全局事务的会话信息创建以下三张数据库表。
- 全局事务表，对应的表为：`global_table`
- 分支事务表，对应的表为：`branch_table`
- 全局锁表，对应的表为：`lock_table`

```sql
CREATE TABLE IF NOT EXISTS `global_table`
(
    `xid`                       VARCHAR(128) NOT NULL,
    `transaction_id`            BIGINT,
    `status`                    TINYINT      NOT NULL,
    `application_id`            VARCHAR(32),
    `transaction_service_group` VARCHAR(32),
    `transaction_name`          VARCHAR(128),
    `timeout`                   INT,
    `begin_time`                BIGINT,
    `application_data`          VARCHAR(2000),
    `gmt_create`                DATETIME,
    `gmt_modified`              DATETIME,
    PRIMARY KEY (`xid`),
    KEY `idx_gmt_modified_status` (`gmt_modified`, `status`),
    KEY `idx_transaction_id` (`transaction_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

CREATE TABLE IF NOT EXISTS `branch_table`
(
    `branch_id`         BIGINT       NOT NULL,
    `xid`               VARCHAR(128) NOT NULL,
    `transaction_id`    BIGINT,
    `resource_group_id` VARCHAR(32),
    `resource_id`       VARCHAR(256),
    `branch_type`       VARCHAR(8),
    `status`            TINYINT,
    `client_id`         VARCHAR(64),
    `application_data`  VARCHAR(2000),
    `gmt_create`        DATETIME(6),
    `gmt_modified`      DATETIME(6),
    PRIMARY KEY (`branch_id`),
    KEY `idx_xid` (`xid`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

CREATE TABLE IF NOT EXISTS `lock_table`
(
    `row_key`        VARCHAR(128) NOT NULL,
    `xid`            VARCHAR(96),
    `transaction_id` BIGINT,
    `branch_id`      BIGINT       NOT NULL,
    `resource_id`    VARCHAR(256),
    `table_name`     VARCHAR(32),
    `pk`             VARCHAR(36),
    `gmt_create`     DATETIME,
    `gmt_modified`   DATETIME,
    PRIMARY KEY (`row_key`),
    KEY `idx_branch_id` (`branch_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;
```

修改`registry.conf`配置
```
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  # 将注册方式修改为 nacos
  type = "nacos"

  nacos {
    application = "seata-server"
    # 修改 nacos 的地址
    serverAddr = "127.0.0.1:1111"
    group = "SEATA_GROUP"
    namespace = ""
    cluster = "default"
    username = ""
    password = ""
  }
}

config {
  # file、nacos 、apollo、zk、consul、etcd3
  #配置方式修改为 nacos
  type = "nacos"

  nacos {
    #修改为使用的 nacos 服务器地址
    serverAddr = "127.0.0.1:1111"
    namespace = ""
    group = "SEATA_GROUP"
    username = "nacos"
    password = "nacos"
    #不使用 seataServer.properties 方式配置
    #dataId = "seataServer.properties"
  }
}
```

修改`/script/config-center/config.txt`
```
#将 Seata Server 的存储模式修改为 db
store.mode=db
# 数据库驱动
store.db.driverClassName=com.mysql.cj.jdbc.Driver
# 数据库 url
store.db.url=jdbc:mysql://127.0.0.1:3306/seata?useSSL=false&characterEncoding=UTF-8&useUnicode=true&serverTimezone=UTC 
# 数据库的用户名
store.db.user=root 
# 数据库的密码
store.db.password=root
# 自定义事务分组
service.vgroupMapping.service-order-group=default 
service.vgroupMapping.service-storage-group=default
service.vgroupMapping.service-account-group=default
```
并上传配置：`sh nacos-config.sh -h 127.0.0.1 -p 1111  -g SEATA_GROUP -u nacos -w nacos`

### 4. 例子
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
    <artifactId>spring-cloud-alibaba-seata-order-8005</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring-cloud-alibaba-seata-order-8005</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
        <seata.version>1.4.2</seata.version>
    </properties>
    <dependencies>
        <!--nacos 服务注册中心-->
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
        <!--引入 seata 依赖-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
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
        <dependency>
            <groupId>net.biancheng.c</groupId>
            <artifactId>spring-cloud-alibaba-api</artifactId>
            <version>${project.version}</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.19</version>
        </dependency>

        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>

        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.2.0</version>
        </dependency>
        <!--添加 Spring Boot 的监控模块-->
        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-actuator -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--SpringCloud ailibaba sentinel -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
        <!--配置中心 nacos-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <!--mybatis自动生成代码插件-->
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.4.0</version>
                <configuration>
                    <configurationFile>src/main/resources/mybatis-generator/generatorConfig.xml</configurationFile>
                    <verbose>true</verbose>
                    <!-- 是否覆盖，true表示会替换生成的JAVA文件，false则不覆盖 -->
                    <overwrite>true</overwrite>
                </configuration>
                <dependencies>
                    <!--mysql驱动包-->
                    <dependency>
                        <groupId>mysql</groupId>
                        <artifactId>mysql-connector-java</artifactId>
                        <version>8.0.19</version>
                    </dependency>
                    <dependency>
                        <groupId>org.mybatis.generator</groupId>
                        <artifactId>mybatis-generator-core</artifactId>
                        <version>1.4.0</version>
                    </dependency>
                </dependencies>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

`bootstrap.yml`
```yml
spring:
  cloud:
    ## Nacos认证信息
    nacos:
      config:
        username: nacos
        password: nacos
        context-path: /nacos
        server-addr: 127.0.0.1:1111 # 设置配置中心服务端地址
        namespace:   # Nacos 配置中心的namespace。需要注意，如果使用 public 的 namcespace ，请不要填写这个值，直接留空即可
```

`application.yml`
```yml
spring:
  application:
    name: spring-cloud-alibaba-seata-order-8005 #服务名
  #数据源配置
  datasource:
    driver-class-name: com.mysql.jdbc.Driver #数据库驱动
    name: defaultDataSource
    url: jdbc:mysql://localhost:3306/seata_order?serverTimezone=UTC #数据库连接地址
    username: root  #数据库的用户名
    password: root  #数据库密码

  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:1111 #nacos 服务器地址
        namespace: public #nacos 命名空间
        username:
        password:
    sentinel:
      transport:
        dashboard: 127.0.0.1:8080  #Sentinel 控制台地址
        port: 8719

    alibaba:
      seata:
        #自定义服务群组，该值必须与 Nacos 配置中的 service.vgroupMapping.{my-service-group}=default 中的 {my-service-group}相同
        tx-service-group: service-order-group
server:
  port: 8005  #端口
seata:
  application-id: ${spring.application.name}
  #自定义服务群组，该值必须与 Nacos 配置中的 service.vgroupMapping.{my-service-group}=default 中的 {my-service-group}相同
  tx-service-group: service-order-group
  service:
    grouplist:
      #Seata 服务器地址
      seata-server: 127.0.0.1:8091
  # Seata 的注册方式为 nacos
  registry:
    type: nacos
    nacos:
      server-addr: 127.0.0.1:1111
  # Seata 的配置中心为 nacos
  config:
    type: nacos
    nacos:
      server-addr: 127.0.0.1:1111

feign:
  sentinel:
    enabled: true #开启 OpenFeign 功能

management:
  endpoints:
    web:
      exposure:
        include: "*"

###################################### MyBatis 配置 ######################################
mybatis:
  # 指定 mapper.xml 的位置
  mapper-locations: classpath:mybatis/mapper/*.xml
  #扫描实体类的位置,在此处指明扫描实体类的包，在 mapper.xml 中就可以不写实体类的全路径名
  type-aliases-package: net.biancheng.c.entity
  configuration:
    #默认开启驼峰命名法，可以不用设置该属性
    map-underscore-to-camel-case: true
```

```java
package net.biancheng.c.service;

import net.biancheng.c.entity.CommonResult;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;

@FeignClient(value = "spring-cloud-alibaba-seata-storage-8006")
public interface StorageService {
    @PostMapping(value = "/storage/decrease")
    CommonResult decrease(@RequestParam("productId") Long productId, @RequestParam("count") Integer count);
}
```

```java
package net.biancheng.c.service.impl;

import io.seata.spring.annotation.GlobalTransactional;
import lombok.extern.slf4j.Slf4j;
import net.biancheng.c.entity.Order;
import net.biancheng.c.mapper.OrderMapper;
import net.biancheng.c.service.AccountService;
import net.biancheng.c.service.OrderService;
import net.biancheng.c.service.StorageService;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

@Service
@Slf4j
public class OrderServiceImpl implements OrderService {
    @Resource
    private OrderMapper orderMapper;
    @Resource
    private StorageService storageService;
    @Resource
    private AccountService accountService;

    /**
     * 创建订单->调用库存服务扣减库存->调用账户服务扣减账户余额->修改订单状态
     * 简单说：下订单->扣库存->减余额->改订单状态
     */
    @Override
    @GlobalTransactional(name = "fsp-create-order", rollbackFor = Exception.class)
    public CommonResult create(Order order) {
        log.info("----->开始新建订单");
        //1 新建订单
        order.setUserId(new Long(1));
        order.setStatus(0);
        orderMapper.create(order);
        //2 扣减库存
        log.info("----->订单服务开始调用库存服务，开始扣减库存");
        storageService.decrease(order.getProductId(), order.getCount());
        log.info("----->订单微服务开始调用库存，扣减库存结束");
        //3 扣减账户
        log.info("----->订单服务开始调用账户服务，开始从账户扣减商品金额");
        accountService.decrease(order.getUserId(), order.getMoney());
        log.info("----->订单微服务开始调用账户，账户扣减商品金额结束");
        //4 修改订单状态，从零到1,1代表已经完成
        log.info("----->修改订单状态开始");
        orderMapper.update(order.getUserId(), 0);
        log.info("----->修改订单状态结束");
        log.info("----->下订单结束了------->");
        return new CommonResult(200, "订单创建成功");
    }
}
```

**@GlobalTransactional**注解：
当调用`@GlobalTransaction`注解的方法时，`TM`会先向`TC`注册全局事务，`TC`生成一个全局唯一的`XID`，返回给`TM`
`@GlobalTransactional`注解既可以在类上使用，也可以在类方法上使用，该注解的使用位置决定了全局事务的范围
- 在类中某个方法使用时，全局事务的范围就是该方法以及它所涉及的所有服务
- 在类上使用时，全局事务的范围就是这个类中的所有方法以及这些方法涉及的服务