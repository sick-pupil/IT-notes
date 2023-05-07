## 1. Tomcat配置
```yml
server:
	#设置请求端口
	port: 8080 
	# 请求断开时间
	connection-timeout: 20000
		
	servlet:
		#指定 Tomcat的请求路径
		context-path: /cl 
		#设置 Tomcat 编码格式
		encoding:
			charset: UTF-8
			
	tomcat:
		# tomcat的URI编码
		uri-encoding: UTF-8
		# 最大连接数
		max-connections: 10000
		# tomcat最大线程数，默认为200
		max-threads: 200
		#最大等待队列长度
		accept-count: 100
		# 最小线程数
		min-spare-threads: 10
		# 请求体最大长度kb
		max-http-post-size: 0
		# 请求体最大长度kb
		max-http-header-size: 0
```

## 2. Mybatis配置
```yml
mybatis:
	#加载 mapper.xml 文件到容器中
	mapper-locations: classpath:mapper/*.xml 
	# 别名，简化 mapper.xml 中请求响应参数类型
	type-aliases-package: com.cl.springboot.pojo  
	configuration:
		#开启驼峰映射
		map-underscore-to-camel-case: true
		# sql日志的打印
		log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

## 3. 日志设置
```yml
logging:
	level:
		com.cl.springboot:
			#指定打印对应文件夹的日志，并设置打印的日志的级别 （com.cl.springboot.mapper 包）
			mapper: debug 
	file:
		#指定日志文件生成的位置
		name: D:/spring.log
		
	# 指定使用的日志配置文件
	config: logback-spring.xml
```

## 4. 数据源配置
```yml
spring:
	datasource:
	    #mysql的配置加载驱动类信息
	    driver-class-name: com.mysql.jdbc.Driver
	    #mysql的连接信息
	    url: jdbc:mysql://localhost:3306/mybatis?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&useSSL=false&serverTimezone = GMT
	    #用户名
	    username: root
	    #密码
	    password: 123456
	    # Type 设置使用何种类型的数据源
	    type: com.alibaba.druid.pool.DruidDataSource 

	#redis配置
	redis:
		database: 0
		# Redis服务器地址
		host: 127.0.0.1
		# Redis服务器连接端口
		port: 6379
		# Redis服务器连接密码（默认为空）
		password:
		jedis:
			pool:
			# 连接池最大连接数（使用负值表示没有限制）
			max-active: 8
			# 连接池最大阻塞等待时间（使用负值表示没有限制）
			max-wait: -1
			# 连接池中的最大空闲连接
			max-idle: 8
			# 连接池中的最小空闲连接
			min-idle: 0
			# 连接超时时间（毫秒）默认是2000ms
			timeout: 2000ms 

	#Druid 数据源属性配置 （需要创建数据源配置类，进行配置才会生效）
	initialSize: 5
	minIdle: 5
	maxActive: 20
	maxWait: 60000
	timeBetweenEvictionRunsMillis: 60000
	minEvictableIdleTimeMillis: 300000
	validationQuery: SELECT 1 FROM DUAL
	testWhileIdle: true
	testOnBorrow: false
	testOnReturn: false
	poolPreparedStatements: true
	#  配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
	maxPoolPreparedStatementPerConnectionSize: 20
	useGlobalDataSourceStat: true
	connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
```