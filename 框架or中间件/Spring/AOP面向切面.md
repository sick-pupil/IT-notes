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
```xml
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.2.6</version>
</dependency>
```

```java
public class UserService {
    public String getUserName(Long userId) {
        System.out.println("获取用户名..");
        return "user" + userId;
    }
}

public class TransactionInterceptor implements MethodInterceptor {
    Object target;

    public TransactionInterceptor(Object target) {
        this.target = target;
    }

    /**
     * proxy：代理对象，CGLib动态生成的代理类实例
     * method：目标对象的方法，上文中实体类所调用的被代理的方法引用
     * args：目标对象方法的参数列表，参数值列表
     * methodProxy：代理对象的方法，生成的代理类对方法的代理引用
     */
    @Override
    public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("开启事务..." + proxy.getClass().getSimpleName());
        Object objValue = null;
        try {
            // 反射调用目标类方法
            objValue = method.invoke(target, args);
            System.out.println("返回值为：" + objValue);
        } catch (Exception e) {
            System.out.println("调用异常!" + e.getMessage());
        } finally {
            System.out.println("调用结束，关闭事务...");
        }
        return objValue;
    }

    /**
     * 获取代理实例
     */
    public Object getTargetProxy() {
        // Enhancer类是cglib中的一个字节码增强器，它可以方便的为你所要处理的类进行扩展
        Enhancer eh = new Enhancer();
        // 1.将目标对象所在的类作为Enhancer类的父类
        eh.setSuperclass(target.getClass());
        // 2.通过实现MethodInterceptor实现方法回调
        eh.setCallback(this);
        // 3. 创建代理实例
        return eh.create();
    }
}

public static void main(String[] args) {
    // 1. 创建目标实例
    UserService userService = new UserService();
    // 2. 创建事务拦截器
    TransactionInterceptor transactionInterceptor = new TransactionInterceptor(userService);
    // 3. 创建代理实例
    UserService userServiceProxy = (UserService) transactionInterceptor.getTargetProxy();
    // 4. 使用代理实例调用目标方法
    userServiceProxy.getUserName(6L);
}
```
## 3. AOP术语
- 连接点：程序运行的某个特定的位置。比方类初始化前，初始化后。方法调用前。方法调用后等等
- 切入点：用于定义通知应该切入到哪些连接点上
- 通知（增强）：切面的详细实现，有前置、后置、环绕、异常、最终五种分类
- 切面：由切点和增强组成

## 4. Spring中AOP的实现操作
Spring中AOP操作一般基于AspectJ实现，需要引入spring-aop依赖、spring-aspects依赖、cglib依赖、aopalliance依赖、aspectj-weaver依赖
<img src="D:\Project\IT-notes\框架or中间件\spring\img\spring新增AOP依赖.png" style="width:250px;height:400px;" />

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

### 3. 切面获取类、方法、参数
- `joinPoint.getSignature().getName()`：获取方法名
- `joinPoint.getSignature().getDeclaringType.getSimpleName()`：获取切面方法所属所属的简单类型
- `joinPoint.getSignature().getDeclaringTypeName()`：获取切面方法所属的类型
- `joinPoint.getSignature().getModifiers()`：获取切面方法声明类型
- `joinPoint.getArgs()`：获取切面方法入参
- `proceedingJoinPoint.proceed(Object[])`：向切面方法传入新入参并执行
```java
@Aspect
@Component
public class aopAspect {
    /**
     * 定义一个切入点表达式,用来确定哪些类需要代理
     * execution(* aopdemo.*.*(..))代表aopdemo包下所有类的所有方法都会被代理
     */
    @Pointcut("execution(* aopdemo.*.*(..))")
    public void declareJoinPointerExpression() {}

    /**
     * 前置方法,在目标方法执行前执行
     * @param joinPoint 封装了代理方法信息的对象,若用不到则可以忽略不写
     */
    @Before("declareJoinPointerExpression()")
    public void beforeMethod(JoinPoint joinPoint){
        System.out.println("目标方法名为:" + joinPoint.getSignature().getName());
        System.out.println("目标方法所属类的简单类名:" +        joinPoint.getSignature().getDeclaringType().getSimpleName());
        System.out.println("目标方法所属类的类名:" + joinPoint.getSignature().getDeclaringTypeName());
        System.out.println("目标方法声明类型:" + Modifier.toString(joinPoint.getSignature().getModifiers()));
        //获取传入目标方法的参数
        Object[] args = joinPoint.getArgs();
        for (int i = 0; i < args.length; i++) {
            System.out.println("第" + (i+1) + "个参数为:" + args[i]);
        }
        System.out.println("被代理的对象:" + joinPoint.getTarget());
        System.out.println("代理对象自己:" + joinPoint.getThis());
    }

    /**
     * 环绕方法,可自定义目标方法执行的时机
     * @param pjd JoinPoint的子接口,添加了
     *            Object proceed() throws Throwable 执行目标方法
     *            Object proceed(Object[] var1) throws Throwable 传入的新的参数去执行目标方法
     *            两个方法
     * @return 此方法需要返回值,返回值视为目标方法的返回值
     */
    @Around("declareJoinPointerExpression()")
    public Object aroundMethod(ProceedingJoinPoint pjd){
        Object result = null;

        try {
            //前置通知
            System.out.println("目标方法执行前...");
            //执行目标方法
            //result = pjd.proeed();
            //用新的参数值执行目标方法
            result = pjd.proceed(new Object[]{"newSpring","newAop"});
            //返回通知
            System.out.println("目标方法返回结果后...");
        } catch (Throwable e) {
            //异常通知
            System.out.println("执行目标方法异常后...");
            throw new RuntimeException(e);
        }
        //后置通知
        System.out.println("目标方法执行后...");

        return result;
    }
}

```