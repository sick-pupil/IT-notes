## 1. Mybatis配置
### 1. Mybatis全局配置文件
```xml
<!--
	mybatis配置文件标签顺序要求
	properties settings typeAliases typeHandlers 
	objectFactory objectWrapperFactory reflectorFactory
	plugins environments databaseIdProvider mappers
-->

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration 
	PUBLIC "-//mybatis.org//DTD Config 3.0//EN" 
	"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	
	<!--获取资源文件-->
	<properties resource="jdbc.properties" />
	
	<settings>
		<!--将sql中字段名中的_下划线命名自动映射为bean中的驼峰命名-->
		<setting name="mapUnderscoreToCamelCase" value="true" />
		<!--延迟加载与按需加载-->
		<setting name="lazyLoadingEnabled" value="true" />
		<setting name="aggressiveLazyLoading" value="false" />
	</settings>
	
	<!--设置类型别名-->
	<typeAliases>
		<!-- <typeAlias type="com.example.bean.student" />同理 -->
		<typeAlias type="com.example.bean.student" alias="student" />

		<!--以包为单位，将包下的所有类设置别名-->
		<package name="com.example.bean.student" />
	</typeAliases>
	
	<!--连接数据库环境-->
	<environments default="development">
		<environment id="development">
			<!--事务管理器类型-->
			<!--
				type="JDBC/MANAGED"
				JDBC：表示执行sql使用的是JDBC原生的事务管理方式，事务提交需手动
				MANAGED：被管理
			-->
			<transactionManager type="JDBC" /> <!--JDBC类型为最原始的JDBC类型，事务提交需要手动-->
			<!--数据源 数据库连接池-->
			<!--
				type="POOLED/UNPOOLED/JNDI"
				POOLED:使用数据库连接池缓存数据库连接
				UNPOOLED：不使用数据库连接池
				JNDI：使用上下文中的数据源
			-->
			<dataSource type="POOLED">
				<property name="driver" value="${driver}" />
				<property name="url" value="${url}" />
				<property name="username" value="${username}" />
				<property name="password" value="${password}" />
			</dataSource>
		</environment>
		
		<environment id="test">
			...
		</environment>
	</environments>
	<!--映射文件-->
	<mappers>
		<mapper resource="com/mybatis/example/mappers/UserMapper.xml" />
		
		<!--以包为单位绑定xml与mapper接口-->
		<!--
			要求：
			1. mapper接口所在的包要和映射文件所在的包一致
			2. mapper接口要和映射文件的名字一致
		-->
		<package name="com.mybatis.example.mappers" />
	</mappers>
</configuration>
```

### 2. mapper接口
Mybatis的mapper接口相当于dao，只不过mapper仅仅是接口，不需要提供实现类
```java
package com.mybatis.example.mappers.UserMapper;

import com.mybatis.example.bean.User;

public interface UserMapper {
	
	//1.默认情况
	public int userNum1(Integer id);
	
	//2.使用@Param传参
	public int userNum2(@Param("id") Integer id);
	
	//3.使用bean传参
	public int insertUser1(User user);
	
	//4.使用map传参
	public int insertUser2(Map params);
	
	//5.返回userList
	public List<User> getUsers1();
	
	//6.返回userMap，可以使用@MapKey("")嵌套形成map
	public List<Map<String, Object>> getUsers2();
}

```

### 3. 映射文件
在resources里建mappers文件夹，在其中创建mapper映射
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper 
	PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
	"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mybatis.example.mappers.UserMapper">
	
	<!--
	当不使用@Param时，mybatis中mapper接口中传递的参数缺省名称为param1、param2
	-->
	<select id="userNum1" resultType="int">
		select count(*) from user where id = ${param1}
	</select>
	
	<!--2.使用@Param传参-->
	<select id="userNum2" resultType="int">
		select count(*) from user where id = ${id} <!--or #{id}-->
		<!-- #{}为占位符 ${}为拼接符，有可能因为缺少单引号双引号产生问题 -->
	</select>
	
	<!--3.使用bean传参-->
	<insert id="insertUser1" parameterType="com.mybatis.example.bean.User">
		insert into user(username, password) values(#{username}, #{password})
	</insert>
	
	<!--4.使用map传参-->
	<insert id="insertUser2" parameterType="map">
		insert into user(username, password) values(#{username}, #{password})
	</insert>
	
	<!--5.返回userList-->
	<select id="getUsers1" resultType="user">
		select * from user
	</select>
	
	<!--6.返回userMap-->
	<select id="getUsers2" resultType="map">
		select * from user
	</select>
</mapper>
```

### 4. 执行
```java
public void testMybatis() {
	//加载全局配置文件
	InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
	//获取SqlSessionFactoryBuilder，构建sqlSesion的工厂Factory类的构建Builder类
	SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();
	//获取SqlSessionFactory
	SqlSessionFactory sqlSessionFactory = sqlSessionFactoryBuilder.build(is);
	//获取SqlSession，代表Java程序与数据库的会话
	SqlSession sqlSession = sqlSessionFactory.openSession(true); //sqlSession.commit();
	//获取mapper接口实现类对象，使用代理模式生成的接口实现类
	UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
	
	int result = userMapper.userNum();
}
```

## 2. 配置idea中mybatis的xml配置文件模板

## 3. Mybatis中特殊SQL
```xml
<!-- 模糊查询 -->
<select id="">
	select * from user where username like '%${username}%'
	<!--
		select * from user where username like concat('%',#{username},'%')
		
		select * from user where username like "%"#{username}"%"
	-->
</select>
<!-- keyProperty所设置的值id为mapper接口方法形参user bean中的成员变量id -->
<insert id="insertUser" useGeneratedKeys="true" keyProperty="id">
	insert into user values(null,#{username},#{password},#{age},#{sex},#{email})
</insert>

<!--使用resultMap映射表字段与bean属性-->
<resultMap id="userResultMap" type="com.mybatis.example.bean.user">
	<id property="id" column="id" />
	<result property="username" column="user_name" />
	<result property="password" column="password" />
	<result property="sex" column="sex" />
	<result property="email" column="email" />
</resultMap>
<select id="getUsers" resultMap="userResultMap">
	select * from user
</select>

<!--级联关系，多个emp员工对应一个dept部门-->
<!--1. Emp员工类中存在Dept部门类-->
<resultMap id="empAndDeptResultMap" type="emp">
	<id property="id" column="id" />
	<result property="empName" column="emp_name" />
	<result property="age" column="age" />
	<result property="sex" column="sex" />
	<result property="email" column="email" />
	<result property="dept.did" column="did" />
	<result property="dept.deptName" column="dept_name" />
</resultMap>
<select id="getEmpAndDept" resultMap="empAndDeptResultMap">
	select * from t_emp left join t_dept on t_emp.did = t_dept.did where t_emp.eid = #{eid}
</select>
<!--2. 使用association-->
<resultMap id="empAndDeptResultMap" type="emp">
	<id property="id" column="id" />
	<result property="empName" column="emp_name" />
	<result property="age" column="age" />
	<result property="sex" column="sex" />
	<result property="email" column="email" />
	<association property="dept" javaType="com.mybatis.example.bean.Dept">
		<id property="did" column="did" />
		<result property="deptName" column="dept_name" />
	</association>
</resultMap>
<!--3. association分步查询与延迟加载-->
<resultMap id="empAndDeptResultMap" type="emp">
	<id property="id" column="id" />
	<result property="empName" column="emp_name" />
	<result property="age" column="age" />
	<result property="sex" column="sex" />
	<result property="email" column="email" />
	<!--这里的column作用与t_emp.did=t_dept.did差不多，即将外部SQL中得到的did值赋给内部SQL中的参数-->
	<association property="dept" column="did" javaType="com.mybatis.example.bean.Dept" 
		select="com.mybatis.example.mybatis.mapper.DeptMapper.selectById" 
		fetchType="eager" /> <!--当开启全局延迟加载后，可通过fetchType属性手动控制延迟加载效果->
	<!--selectById(@Param("did") Integer did)-->
</resultMap>
<!--4. 使用collection-->
<resultMap id="empAndDeptResultMap" type="dept">
	<id property="id" column="id" />
	<result property="deptName" column="dept_name" />
	<collection property="emps" ofType="com.mybatis.example.bean.Emp">
		<id property="eid" column="eid" />
		<result property="empName" column="emp_name" />
		<result property="age" column="age" />
		<result property="sex" column="sex" />
		<result property="email" column="email" />
	</collection>
</resultMap>
<!--5. collection分步查询与延迟加载-->
<resultMap id="empAndDeptResultMap" type="dept">
	<id property="id" column="id" />
	<result property="deptName" column="dept_name" />
	<collection property="emps" ofType="com.mybatis.example.bean.Emp" 
		column="did" select="com.example.mybatis.mapper.EmpMapper.selectByDid" />
</resultMap>
```

## 4. 动态SQL标签
- ### \<if\>
- ### \<where\>
- ### \<trim\>
- ### \<choose\>
- ### \<when\>
- ### \<otherwise\>
- ### \<foreach\>
- ### \<sql\>

## 5. Mybatis缓存
### 1. 一级缓存
一级缓存为SqlSession级别的，通过同一个SqlSession查询的数据会被缓存，下次查询相同的数据，就会从缓存中直接获取，不会从数据库重新访问，*一级缓存是默认开启的*，`sqlSession.clearCache()`只会清空一级缓存

使一级缓存失效的四种情况：
1. 不同的SqlSession对应不同的一级缓存
2. 同一个SqlSession但是查询条件不同
3. 同一个SqlSession两次查询期间执行任何一次增删改操作（因为有可能改变缓存，所以应该让缓存失效，清空缓存）
4. 同一个SqlSession两次查询期间手动清空缓存

### 2. 二级缓存
二级缓存是SqlSessionFactory级别，通过同一个SqlSessionFactory创建的SqlSession查询的结果会被缓存；从此若再次执行相同的查询语句，结果就会从缓存中获取

二次缓存开启的条件：
1. 在核心配置文件中，设置全局配置属性`cacheEnabled="true"`，该属性默认true
2. 在mapper映射文件中设置标签`<cache />`
3. 二级缓存必须在SqlSession关闭或提交之后有效
4. 查询的数据所转换的实体类类型必须实现序列化接口`implements Serializable`

使二级缓存失效的情况：两次查询之间执行任意的增删改，会使一级和二级缓存同时失效

### 3. 缓存查询顺序
1. 先查询二级缓存，因为二级缓存中可能会有其他程序已经查出来的数据，可以拿来直接使用
2. 如果二级缓存没有命中，再查询一级缓存
3. 如果一级缓存也没有命中，则查询数据库
4. SqlSession关闭之后，一级缓存中的数据会写入二级缓存，关闭前基本都是缓存在SqlSession缓存中

### 4. 整合EHCache
```xml
<dependency>
	<groupId>org.mybatis.caches</groupId>
	<artifactId>mybatis-ehcache</artifactId>
	<version>1.2.1</version>
</dependency>
```

ehcache.xml
```xml
<?xml version="1.0" encoding="utf-8" ?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xsi:noNamespaceSchemaLocation="../config/ehcache.xsd">
	<diskStore path="D:\ehcache-data" />
	<defaultCache maxElementsInMemory="1000" 
		maxElementsOnDisk="10000000" 
		eternal="false" 
		overflowToDisk="true" 
		timeToIdleSeconds="120" 
		timeToLiveSeconds="120" 
		diskExpiryThreadIntervalSeconds="120" 
		memoryStoreEvictionPolicy="LRU" />
</ehcache>
```

mybatis-config.xml
```xml
<settings>
	<setting name="cacheEnabled" value="true" />
</settings>
```

mapper.xml
```xml
<mapper ...>
	<!-- cache配置 -->
	<cache type="com.mybatis.caches.ehcache.EhcacheCache" />
	<!--
		eviction="FIFO"
		flushInterval="60000"
		size="512"
		readOnly="true" />
	-->
</mapper>
```

## 6. 分页插件
```xml
<dependency>
	<groupId>com.github.pagehelper</groupId>
	<artifactId>pagehelper</artifactId>
	<version>5.0.0</version>
</dependency>
```

```xml
<configuration>
	...
	<plugins interceptor="com.github.pagehelper.PageInterceptor" />
	...
</configuration>
```

```java
	/*
		limit index,pageSize
		index 数据索引，索引从0开始，当前页的起始索引
		pageSize 每页显示条数
		pageNum 当前页的页码
		index=(pageNum-1)*pageSize
	*/
	//使用分页功能，必须在查询之前定义分页，startPage(pageNum, pageSize)
	Page<Object> page = PageHelper.startPage(1, 4);
	List<Emp> list = mapper.selectByExample();
	PageInfo<Emp> page = new Page<>(list);
```

## 7. 逆向工程
即根据配置文件逆向生成controller、service、dao、mapper等工程包与文件
1. 添加maven依赖包
```xml
<dependencies>
	<!-- 逆向工程 -->
	<dependency>
		<groupId>org.mybatis.generator</groupId>
		<artifactId>mybatis-generator-core</artifactId>
		<version>${mybatis.generator.core.version}</version>
	</dependency>
	
	<!-- Mysql -->
	<dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
		<version>${mysql.connector.java.version}</version>
	</dependency>
</dependencies>
```

2. pom.xml中添加插件
```xml
......
<plugin>
	<groupId>org.mybatis.generator</groupId>
	<artifactId>mybatis-generator-maven-plugin</artifactId>
	<version>1.4.0</version>   
	<configuration>
		<verbose>true</verbose>
		<overwrite>true</overwrite>
	</configuration>
</plugin>
......
```

3. 添加逆向工程generator配置文件`generatorConfig.xml`
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE generatorConfiguration PUBLIC
        "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd" >
<generatorConfiguration>
    <!-- classPathEntry:数据库的 JDBC驱动的jar 包地址 -->
    <classPathEntry location="lib/mysql-connector-java-5.1.47.jar"/>

    <!--生成java代码-->
    <context id="context" targetRuntime="MyBatis3">

        <!-- 配置生成pojo的序列化的插件-->
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin"/>

        <!-- 配置生成toString的序列化的插件  -->
        <plugin type="org.mybatis.generator.plugins.ToStringPlugin"/>

        <commentGenerator>
            <!-- 是否去除自动生成的（english）注释 true：是 ： false:否 -->
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>

        <!--数据库连接的信息：驱动类、连接地址、用户名、密码 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/spring_141"
                        userId="root"
                        password="root"/>

        <!-- 指定生成的实体类的存放位置 -->
        <javaModelGenerator targetPackage="com.itheima.domain" targetProject="./src/main/java">
            <property name="enableSubPackages" value="true"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>

        <!-- 指定生成的Dao映射文件的存放位置 -->
        <sqlMapGenerator targetPackage="com.itheima.dao" targetProject="./src/main/resources">
            <property name="enableSubPackages" value="false"/>
        </sqlMapGenerator>

        <!--指定生成的Dao接口的存放位置-->
        <javaClientGenerator targetPackage="com.itheima.dao" targetProject="./src/main/java" type="XMLMAPPER">
            <property name="enableSubPackages" value="false"/>
        </javaClientGenerator>

        <!-- 指定数据库表 -->
        <table tableName="account" domainObjectName="Account" mapperName="AccountDao"
               enableCountByExample="false" enableDeleteByExample="false"
               enableSelectByExample="true" enableUpdateByExample="false"/>

    </context>
</generatorConfiguration>
```

```xml
<!DOCTYPE generatorConfiguration PUBLIC
        "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!--导入properties配置文件-->
    <properties resource="jdbc.properties"></properties>

    <!--指定特定数据库的jdbc驱动jar包的位置-->
    <classPathEntry location="驱动包的绝对路径"/>

    <context id="default" targetRuntime="MyBatis3">

        <!-- 配置生成的类不带注释（建议，因为生成的注释没啥用） -->
        <commentGenerator>
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>

        <!--jdbc的数据库连接 -->
        <jdbcConnection
                driverClass="${driver}"
                connectionURL="${url}"
                userId="${user}"
                password="${password}" />


        <!-- 指定生成的JavaBean的包位置,
                用来生成含有主键key的类，记录类 以及查询Example类
                不用事先创建好包结构，mybatis-generator会根据配置自动创建，sqlMapGenerator同
            targetPackage     指定生成的JavaBean生成所在的包名
            targetProject     指定在该项目下所在的路径
        -->
        <javaModelGenerator targetPackage="com.generator.bean"
                            targetProject="src/main/java">

            <!-- 是否允许子包，即targetPackage.schemaName.tableName -->
            <property name="enableSubPackages" value="false"/>
            <!-- 是否对model添加 构造函数 -->
            <property name="constructorBased" value="true"/>
            <!-- 建立的Model对象是否 不可改变  即生成的Model对象不会有 setter方法，只有构造方法 -->
            <property name="immutable" value="false"/>
        </javaModelGenerator>

        <!--Mapper映射文件生成所在的目录 为每一个数据库的表生成对应的SqlMap文件 -->
        <sqlMapGenerator targetPackage="mapper" targetProject="src/main/resources" />

        <!-- 指定dao接口（mapper）的位置
                type="ANNOTATEDMAPPER",生成Java Model 和基于注解的Mapper对象
                type="MIXEDMAPPER",生成基于注解的Java Model 和相应的Mapper对象
                type="XMLMAPPER",生成SQLMap XML文件和独立的Mapper接口
        -->
        <javaClientGenerator targetPackage="com.generator.dao"
                             targetProject="src/main/java" type="XMLMAPPER">
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>

        <!-- 指定每个表的生成策略
                配置数据表与JavaBean的名称映射，以及是否创建根据条件查询的sql语句等 -->
        <!-- tableName指定表名，domainObjectName指定表对应的JavaBean名 -->
        <table tableName="dept" domainObjectName="Department"
               enableCountByExample="false" enableUpdateByExample="true"
               enableDeleteByExample="false" enableSelectByExample="true"
               selectByExampleQueryId="false">
        </table>
        <table tableName="employees" domainObjectName="Employee"
               enableCountByExample="false" enableUpdateByExample="true"
               enableDeleteByExample="false" enableSelectByExample="true"
               selectByExampleQueryId="false">
        </table>
    </context>
</generatorConfiguration>
```

4. 运行`generator`插件，不运行`generator-maven`插件也可以运行`Java`代码
```java
package deecyn.shop_02.mbg;

import org.mybatis.generator.api.MyBatisGenerator;
import org.mybatis.generator.config.Configuration;
import org.mybatis.generator.config.xml.ConfigurationParser;
import org.mybatis.generator.internal.DefaultShellCallback;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;

public class Generator {
    public static void main(String[] args) throws Exception {
        // MBG 执行过程中的警告信息
        List<String> warnings = new ArrayList<String>();
        // 当生成的代码重复时，覆盖原代码
        boolean overwrite = true;
        // 读取我们的 MBG 配置文件
        InputStream is = Generator.class.getResourceAsStream("/mybatis-generator-config.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(is);
        is.close();

        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        //创建 MBG
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        //执行生成代码
        myBatisGenerator.generate(null);
        //输出警告信息
        for (String warning : warnings) {
            System.out.println(warning);
        }
    }
}

```