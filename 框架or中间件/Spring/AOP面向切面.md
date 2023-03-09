## 1. 概念
随着Java应用内部越来越庞大、复杂，在编写代码时会存在一些重复的功能性代码，比如日志输出、异常处理、权限判断，这些功能性代码段作为通用功能重复出现会造成代码冗余、难以维护，因此可以使用aop把这些通用代码合并封装在同一个模块中，在需要使用时只需要把代码段作为切面融入业务中即可

## 2. 底层原理
aop底层使用动态代理：（有接口则）JDK与（无接口则）CGLIB

### 1. JDK动态代理
底层实则在获取代理类时，把代理类方法与实现InvocationHandler接口类中的invoke方法绑定，当获取代理类并调用类方法时实则调用invoke方法
```java
package kite.lab.spring.aop.jdkaop;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * JdkProxy
 *
 * @author fengzheng
 */
public class JdkProxy implements InvocationHandler {

    private Object target;

    /**
     * 绑定委托对象并返回一个代理类
     *
     * @param target
     * @return
     */
    public Object bind(Object target) {
        this.target = target;
        //取得代理对象
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),
            target.getClass().getInterfaces(), this);
    }


    /**
     * 调用方法
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args)
    throws Throwable {
        Object result = null;
        System.out.println("事物开始");
        //执行方法
        result = method.invoke(target, args);
        System.out.println("事物结束");
        return result;
    }
}
```

```java
//IWorker为接口，worker为实体类
public static void main(String[] args) {
    JdkProxy jdkProxy = new JdkProxy();
    IWorker worker = (IWorker) jdkProxy.bind(new Worker());
    worker.dowork(); //调用代理类方法会实际调用InvocationHandler中的invoke方法
}
```

### 2. CGLIB动态代理


## 3. AOP术语
- 连接点：程序运行的某个特定的位置。比方类初始化前，初始化后。方法调用前。方法调用后等等
- 切入点：用于定义通知应该切入到哪些连接点上
- 通知（增强）：切面的详细实现，有前置、后置、环绕、异常、最终五种分类
- 切面：由切点和增强组成

## 4. Spring中AOP的实现操作
Spring中AOP操作一般基于AspectJ实现，需要引入spring-aop依赖、spring-aspects依赖、cglib依赖、aopalliance依赖、aspectj-weaver依赖
<img src="D:\Project\IT notes\框架or中间件\spring\img\spring新增AOP依赖.png" style="width:250px;height:400px;" />

### 1. xml实现
```java
public class HelloWorldImpl1 implements HelloWorld
{
    public void printHelloWorld()
    {
        System.out.println("Enter HelloWorldImpl1.printHelloWorld()");
    }
    
    public void doPrint()
    {
        System.out.println("Enter HelloWorldImpl1.doPrint()");
        return ;
    }
}

public class HelloWorldImpl2 implements HelloWorld
{
    public void printHelloWorld()
    {
        System.out.println("Enter HelloWorldImpl2.printHelloWorld()");
    }
    
    public void doPrint()
    {
        System.out.println("Enter HelloWorldImpl2.doPrint()");
        return ;
    }
}

public class TimeHandler
{
    public void printTime()
    {
        System.out.println("CurrentTime = " + System.currentTimeMillis());
    }
}

public class LogHandler
{
    public void LogBefore()
    {
        System.out.println("Log before method");
    }
    
    public void LogAfter()
    {
        System.out.println("Log after method");
    }
}
```

```xml
<context:component-scan base-package="..."/>
<aop:aspectj-autoproxy />

<bean id="helloWorldImpl1" class="com.xrq.aop.HelloWorldImpl1" />
<bean id="helloWorldImpl2" class="com.xrq.aop.HelloWorldImpl2" />
<bean id="timeHandler" class="com.xrq.aop.TimeHandler" />
<bean id="logHandler" class="com.xrq.aop.LogHandler" />

<!--aop配置-->
<aop:config>
	<!--aop切面1-->
	<aop:aspect id="time" ref="timeHandler" order="1">
		<!--aop切入点-->
		<aop:pointcut id="addTime" expression="execution(* com.xrq.aop.HelloWorld.*(..))" />

		<!--aop增强-->
		<aop:before method="printTime" pointcut-ref="addTime" />
		<aop:after method="printTime" pointcut-ref="addTime" />
	</aop:aspect>
	
	<!--aop切面2-->
	<aop:aspect id="log" ref="logHandler" order="2">
		<!--aop切入点-->
		<aop:pointcut id="printLog" expression="execution(* com.xrq.aop.HelloWorld.*(..))" />

		<!--aop增强-->
		<aop:before method="LogBefore" pointcut-ref="printLog" />
		<aop:after method="LogAfter" pointcut-ref="printLog" />
	</aop:aspect>
</aop:config>
```

### 2. 注解实现
```java
@Aspect
@Order(1)
public class LogAspect {
    //定义切面的执行规规则 
    @Pointcut("execution (* zfcoding.service..*.*(..))")
    public void pointCut() {
    }
    @Before("pointCut()") //公共切入点抽取
    public void logStart() {
        System.out.println("登录开始通知");
    }
    @After("pointCut()")
    public void logEnd() {
        System.out.println("登录结束通知");
    }
    @AfterReturning("pointCut()")
    public void logReturn() {
        System.out.println("返回通知");
    }
    @AfterThrowing("pointCut()")
    public void logException() {
        System.out.println("异常通知");
    }
}
```

```java
@Configuration  //配置类
@EnableAspectJAutoProxy  //开启注解的Aop 功能
public class MyConfigAspect {
    @Bean //bean注入到Spring容器当中
    public UserServiceImpl userService() {
        return new UserServiceImpl();
    }

    @Bean
    public LogAspect logAspect() {
        return new LogAspect();
    }

}
```

```java
public class AspectTest {
    @Test
    public void test(){
        AnnotationConfigApplicationContext applicationContext=new AnnotationConfigApplicationContext(MyConfigAspect.class);
        UserServiceImpl bean = applicationContext.getBean(UserServiceImpl.class);
        bean.loginUser("admin","123");
    }
}
```