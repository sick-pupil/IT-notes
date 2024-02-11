## 1. 概念
传统Java设计中，都是在对象内部通过new进行对象的创建，是程序主动创建对象
而对于IOC，存在一个专门的IOC容器管理对象的生命周期，控制对象创建与对象之间的引用关系，由容器进行对象的依赖与创建，对象被动接收依赖对象

作用：降低对象之间的耦合度

<img src="D:\Project\IT-notes\框架or中间件\spring\img\springtest图例1.png" style="width:300px;height:400px;" />

<img src="D:\Project\IT-notes\框架or中间件\spring\img\springtest图例2.png" style="width:700px;height:300px;" />

<img src="D:\Project\IT-notes\框架or中间件\spring\img\springtest图例3.png" style="width:700px;height:300px;" />

<img src="D:\Project\IT-notes\框架or中间件\spring\img\springtest图例4.png" style="width:700px;height:300px;" />

#### XML配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xmlns:context="http://www.springframework.org/schema/context"  
       xmlns:aop="http://www.springframework.org/schema/aop"  
       xmlns:tx="http://www.springframework.org/schema/tx"  
       xmlns:mvc="http://www.springframework.org/schema/mvc" 
       xsi:schemaLocation="http://www.springframework.org/schema/beans  
       http://www.springframework.org/schema/beans/spring-beans.xsd       http://www.springframework.org/schema/context       http://www.springframework.org/schema/context/spring-context.xsd       http://www.springframework.org/schema/aop       http://www.springframework.org/schema/aop/spring-aop.xsd       http://www.springframework.org/schema/tx       http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/mvc       http://www.springframework.org/schema/mvc/spring-mvc.xsd">
  
    <!--配置User对象-->  
    <bean id="..." class="..."></bean>  
  
</beans>
```

## 2. 底层原理
**xml解析、工厂模式、反射**

<img src="D:\Project\IT-notes\框架or中间件\spring\img\工厂模式.png" style="width:700px;height:150px;" />

1. xml配置文件，配置创建的对象
```xml
<bean id="..." class="..."></bean>
```
2. 创建工厂类
```java
public class BeanFactory {
	public static Bean getBean() {
		String classValue = class值; //xml解析
		Class clazz = Class.forName(classValue); //反射创建类对象
		return (Bean)clazz.newInstance();
	}
}
```


## 3. IOC容器
IOC思想由IOC容器实现，IOC容器底层实则为对象工厂

Spring提供IOC容器的两种实现方式（接口）：
1. BeanFactory对象工厂，Spring内部接口，**加载配置文件时不会创建对象，在获取对象getBean()时才会加载对象**
2. ApplicationContext应用上下文，BeanFactory的子接口，面向外部供开发者使用，**在ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml")就已经开始加载配置文件中的对象**

<img src="D:\Project\IT-notes\框架or中间件\spring\img\ApplicationContext实现类与接口.png" style="width:700px;height:300px;" />

## 4. IOC具体操作
```xml
<!--创建对象-->
<bean id="user" class="com.local.bean.User"></bean>
<!--
id属性：唯一标识
class属性：类全路径
创建对象时，默认执行无参构造方法
-->

<!--set方法注入属性-->
<bean id="user" class="com.local.bean.User">
	<property name="..." value="..."></property> <!--类中要有对应属性的set方法-->
	<property name="..." value="..."></property>
	
	<!--设置属性空值-->
	<property name="...">
		<null/>
	</property>
	
	<!--属性值存在特殊字符,转义 or CDATA-->
	<property name="..." value="&lt;&gt;"></property>
	<property name="...">
		<value><![CDATA[...]]></value>
	</property>
	
	<!--内部注入对象-->
	<property name="dept" ref="com.local.bean.Dept" /> <!--ref值需为其他bean的id值-->
	
	<property name="dept.dname" value="..." /> <!--dept在User中需要存在get方法-->
</bean>

<!--有参构造器注入属性-->
<bean id="user" class="com.local.bean.User">
	<constructor-arg name="..." value="..."></constructor-arg> <!--类中要有对象的有参构造方法-->
	<constructor-arg name="..." value="..."></constructor-arg>
</bean>

<!--级联关系-->
<bean id="..." class="...">
	<property name="..." value="..."></property>
	<property name="..." value="..."></property>
	<property name="">
		<bean id="" class="">
			<property name="" value=""></property>
		</bean>
	</property>
</bean>

<!--注入集合-->
<bean id="..." class="...">
	<!--属性都需要set方法-->
	
	<!--array类型-->
	<property name="...">
		<array>
			<value></value>
		</array>
	</property>
	
	<!--list类型-->
	<property name="...">
		<list>
			<value></value>
			or
			<ref bean="" /> <!--bean填入id值-->
		</list>
	</property>
	
	<!--map类型-->
	<property name="">
		<map>
			<entry key="" value="" />
		</map>
	</property>
	
	<!--set类型-->
	<property name="">
		<set>
			<value></value>
		</set>
	</property>
</bean>
```

## 5. FactoryBean
Spring存在两种bean，**普通bean**与**FactoryBean**

普通bean：在配置bean时已经确定定义好bean类型
工厂bean：配置定义bean的类型与实际使用时可以不一致

```xml
<bean id="myBean" class="com.local.bean.MyBean"></bean> <!--配置文件中定义bean-->
<bean id="course" class="com.local.bean.Course"></bean>
```

```java
public class Mybean implements FactoryBean<Course> {
	@Override  
	public boolean isSingleton() {  
	    return false;  
	}  
	  
	@Override  
	public Course getObject() throws Exception {  
	    return null;  
	}  
	  
	@Override  
	public Class<Course> getObjectType() {  
	    return null;  
	}
}
```

```java
...
	ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
	//此处返回的是Course对象
	Course course = applicationContext.getBean("com.local.bean.MyBean");
...
```

<img src="D:\Project\IT-notes\框架or中间件\spring\img\FactoryBean调用getObject流程.png" style="width:700px;height:1200px;" />

## 6. Bean作用域
Bean作用域：在IOC容器中决定bean对象是**单实例singleton**还是**多实例prototype**，在`applicationContext.getBean`的返回值会存在返回同一个对象或者多个同类型的不同对象，Spring容器中默认bean为单实例

区别：singleton在加载配置时就会创建bean对象，而prototype只有在获取bean实例的时候才会加载创建bean对象

```xml
<bean id="..." class="..." scope="singleton or prototype">
	...
</bean>
```

scope还可以填request、session

## 7. Bean生命周期
1. 通过构造器创建bean实例
2. 为bean属性设置值（调用set方法）
3. 调用bean初始化方法
4. 使用bean
5. 容器关闭时，调用bean销毁方法

```xml
<bean id="myBean" class="..." init-method="initMethod" destory-method="destoryMethod">
	<property name="property" value="..." />
</bean>

<bean id="myBeanPost" class="..."></bean> <!--在配置文件中注册后置处理器bean后会对配置中的所有bean生效-->
```

```java
public class MyBean {
	
	public MyBean() {
		System.out.println("第一步");
	}
	
	private String property;
	public void setProperty(String property) {
		this.property = property;
		System.out.println("第二步");
	}
	
	public void initMethod() {
		System.out.println("第三步");
	}
	
	public void destoryMethod() {
		System.out.println("第五步");
	}
}

class MyBeanPost implements BeanPostProcessor { //bean后置处理器
	
	@Override  //在其他bean的init-method前执行
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {  
	    return null;  
	}
	  
	@Override  //在其他bean的init-method后执行
	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {  
	    return null;  
	}
}
```

## 8. 属性的自动装配
```xml
<bean id="..." class="..." autowire="byName or byType"></bean>

<bean id="emp" class="com.local.Emp" autowire="byName" > <!--autowire为自动装配-->
	<!--
	<property name="dept" ref="dept" /> 此为手动装配属性
	-->
</bean> <!--emp中存在dept对象属性-->
<bean id="dept" class="com.local.Dept" />
```

## 9. 引用外部属性文件
```xml
<context:property-placeholder location="classpath:jdbc.properties" />

<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
	<property name="driverClassName" value="${jdbc.driverclass}"></property>
	<property name="url" value="${jdbc.url}"></property>
	<property name="username" value="${jdbc.username}"></property>
	<property name="password" value="${jdbc.password}"></property>
</bean>
```

```property
jdbc.driverclass=xxx
jdbc.url=xxx
jdbc.username=xxx
jdbc.password=xxx
```

## 10. Spring注解管理bean
### 1. 创建对象
- `Component`
- `Service`
- `Controller`
- `Repository`

使用注解需要在配置文件中提示Spring容器扫描具体位置包中的类
```xml
<context:component-scan base-package="com.local.dao,com.local.service,com.local.controller" />
<context:component-scan base-package="com.local" />
<!--以上扫描包中的所有类，不管有没有注解注释类-->

<context:component-scan base-package="com.local" use-default-filters="false">
	<context:include-filter type="annotation" expression="org.springframework.stereotype.Controller" />
</context:component-scan>
<!--以上只扫描包中的包含Controller注解的类-->

<context:component-scan base-package="com.local">
	<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller" />
</context:component-scan>
<!--以上不扫描包中的包含Controller注解的类-->
```

### 2. 属性注入
- `AutoWired`：根据属性类型进行自动装配
- `Qualifier`：根据属性名称注入
- `Resource`：可以根据类型也可以根据名称注入
- `Value`：注入普通类型属性

### 3. 纯注解开发
需要创建配置类，可以使用配置类替代xml配置文件
- `Configuration`
- `ComponentScan`

```java
@Configuration
@ComponentScan(basePackages = {"com.local"})
public class SpringConfig {

}
```

```java
ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class)
```