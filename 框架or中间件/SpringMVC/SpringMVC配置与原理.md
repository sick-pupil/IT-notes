## 1. 过程原理
SpringMVC处理请求过程：
1. 浏览器发送请求，若请求地址符合前端请求分发器`DispatcherServlet`的`url-pattern`，该请求则会被`DispatcherServlet`处理
2. 前端请求分发器读取SpringMVC配置文件，通过扫描IOC中的bean找到对应Controller控制器，并通过`@RequestMapping`进行匹配。若匹配成功，则由该控制器方法处理该请求
3. Controller控制器处理完成后可能会返回字符串类型的视图名称，该视图名称会被Spring配置文件中注册的视图解析器解析，加上前缀后缀得出文件的绝对位置，最终返回视图对应页面

**详细原理**
1. url地址发送请求，通过`web.xml`判断是否为`url-pattern`的路径或子路径
2. 将请求提交给`DispatcherServlet`，会委托应用系统的其他模块负责对请求进行真正的处理工作
3. 进入spring配置文件，查询一个或多个`HandlerMapping`这个类的配置
4. 通过配置找到请求的key，再找到这个key的值的对应id名的bean配置
5. 通过这个bean，进入controller类进行处理
6. 返回一个视图`ModelAndView`(比如“hello”)
7. Dispathcher查询一个或多个`ViewResolver`视图解析器，并按`ModelAndView`的要求找到对应的视图对象文件
8. 响应（视图对象负责渲染返回给客户端）
<img src="D:\Project\IT-notes\框架or中间件\SpringMVC\img\SpringMVC处理请求详细过程.png" style="width:700px;height:200px;" />

## 2. 相关配置
### 1. 在web.xml注册DispatcherServlet
```xml
<!--当web.xml中定义DispatcherServlet没有明确DispatcherServlet配置文件位置时，则默认该配置文件在WEB-INF文件夹中-->
<servlet>
	<servlet-name>springDispatcherServlet</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	<load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
	<servlet-name>springmvc</servlet-name>
	<url-pattern>*.action</url-pattern>
</servlet-mapping>

<!--DispatcherServlet配置文件自定义位置-->
<servlet>
	<servlet-name>springmvc</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	<init-param>
		<!--contextConfigLocation为默认DispatcherServlet属性，指上下文配置文件路径-->
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:springmvc.xml</param-value>
	</init-param>
	<!--servlet容器是否应该在web应用程序启动时就加载servlet-->
	<load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
	<servlet-name>springmvc</servlet-name>
	<!--
		当url-pattern为/时，DispatcherServlet接收转发除了.jsp以外的所有请求，包括.html、.css、.js等静态资源请求，关于.jsp的请求交由JspServlet处理
		当url-pattern为/*时，DispatcherServlet接收转发所有请求
	-->
	<url-pattern>/</url-pattern>
</servlet-mapping>
```

### 2. 配置SpringMVC配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
	    http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context 
        http://www.springframework.org/schema/context/spring-context-4.1.xsd
        http://www.springframework.org/schema/mvc 
        http://www.springframework.org/schema/mvc/spring-mvc-4.1.xsd">                    

    <!--让IOC容器扫描包中类并注册入IOC容器中-->
    <context:component-scan base-package="test.SpringMVC"/>

    <!--
	    开启该配置会在SpringMVC上下文定义一个
	    org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler，
	    即默认Servlet
	    如果请求URL符合DispatcherServlet的url-pattern，则交由DispatcherServlet处理；
	    如果URL无法匹配DispatcherServlet，请求URL为静态资源请求，则该请求会被转发给DefaultServletHttpRequestHandler处理；
    -->
    <mvc:default-servlet-handler />
    <!--开启SpringMVC注解驱动，使用mvc:default-servlet-handler一定要加上mvc:annotation-driven-->
    <mvc:annotation-driven />
	
	<!--
	   <mvc:resources location="/resource/" mapping="/resource/**">与<mvc:default-servlet-handler>可起到相同作用
	   location为webapp目录中的路径，mapping为请求静态资源URL 
    -->
	
    <!--注册ViewResolver视图解析器-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" 
            id="internalResourceViewResolver">
        <!--前缀-->
        <property name="prefix" value="/WEB-INF/jsp/" />
        <!--后缀-->
        <property name="suffix" value=".jsp" />
    </bean>
</beans>
```

### 3. 配置Spring配置文件

### 4. 编写Controller代码
```java
@Controller
@RequestMapping("/user")
public class UserController {
	
	@RequestMapping("/add")
	public void add() {
		...
		return ...
	}
}
```

## 3. 相关MVC注解
- ### @RequestMapping
- ### @GetMapping
- ### @PostMapping
- ### @PutMapping
- ### @DeleteMapping
- ### @RequestParam
- ### @RequestHeader
- ### @CookieValue
- ### @RequestBody
- ### @ResponseBody
- ### @RestController

## 4. SpringMVC实现RESTful
1. 针对GET请求，`@RequestMapping`请求路径中需要设置占位符获取路径参数：`@RequestMapping("/testRest/{id}/{username}")`，使用`@PathVariable`获取
```java
@RequestMapping("/testRest/{id}/{username}")
public String testRest(@PathVariable("id") String id, @PathVariable("username")
        String username) {
    System.out.println("id:" + id + ",username:" + username);
    return "success";
}
```
2. 设置`HiddenHttpMethodFilter`对请求进行过滤
```xml
<filter>
	<filter-name>HiddenHttpMethodFilter</filter-name>
	<filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
	<filter-name>HiddenHttpMethodFilter</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
```
由于浏览器默认只支持发送`GET`和`POST`方法的请求，因此我们需要在`web.xml`中使用 Spring MVC 提供的`HiddenHttpMethodFilter`对请求进行过滤。这个过滤器可以帮助我们将`POST`请求转换为`PUT`或`DELETE`请求

`HiddenHttpMethodFilter`处理`PUT`和`DELETE`请求时，必须满足以下 2 个条件：
-   当前请求的请求方式必须为`POST`；
-   **当前请求必须传输请求参数\_method，过滤器就会将当前请求的请求方式转换为请求参数 _method 的值**
3. POST、DELETE、PUT、GET四种请求方式分别对应增删改查

## 5. 获取请求参数
1. 通过原生ServletAPI获取
```java
@RequestMapping("/testParam")
public String testParam(HttpServletRequest request) {
	String username = request.getParameter("username");
	String password = request.getParameter("password");
	System.out.println("username:" + username + " ,password:" + password);
	return "success";
}
```
2. 通过控制器方法的形参获取
```java
@RequestMapping("/testParam")
public String testParam(String username, String password) {
	System.out.println("username:" + username + " ,password:" + password);
	return "success";
}
```
3. 通过`@RequestParam`获取
```java
@RequestMapping("/testParam")
public String testParam(@RequestParam(value="user_name",required=false,defaultValue="aaa") String username,
						@RequestParam(value="password",required=false) String password) {
	System.out.println("username:" + username + " ,password:" + password);
	return "success";
}
```
4. 通过实体类获取
```java
@RequestMapping("/testParam")
public String testParam(User user) {
	System.out.println("username:" + user.username + " ,password:" + user.password);
	return "success";
}
```

## 6. 请求参数乱码
- **GET请求乱码**，可以通过更改tomcat配置文件`server.xml`实现
```xml
...
<Connector port="8080" URIEncoding="UTF-8" protocol="HTTP/1.1"
			connectionTimeout="20000" 
			redirectPort="8443" />
...
```
- **POST请求乱码**，可以通过在`web.xml`增加编码过滤器实现
```xml
<filter>
	<filter-name>CharacterEncodingFilter</filter-name>
	<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
	<init-param>
		<param-name>encoding</param-name>
		<param-value>UTF-8</param-value>
	</init-param>
	<init-param>
		<param-name>forceResponseEncoding</param-name>
		<param-value>true</param-value>
	</init-param>
</filter>
<filter-mapping>
	<filter-name>CharacterEncodingFilter</filter-name>
	<url-pattern>/*</url-pattern>
</filter-mapping>
```

**注意：CharacterEncodingFilter一定要配置在HiddenHttpMethodFilter前面，因为CharacterEncodingFilter中存在request.setCharacterEncoding()：用来确保发往服务器的参数的编码格式。指定后可以通过request.getParameter()获取自己想要的；如果没有提前指定，则会按照服务器端默认的“iso-8859-1”来进行编码

## 7. 共享域设置参数
### request域设置
1. 使用servletAPI设置
```java
@RequestMapping("/testParam")
public String testParam(HttpServletRequest request) {
	request.setAttribute("","");
	return "success";
}
```
2. 使用`ModelAndView Model ModelMap`设置
```java
@RequestMapping("/testParam")
public String testParam() {
	ModelAndView mav = new ModelAndView();
	mav.addObject("",""); //addObject其实就是向request域中设置attribute
	mav.setViewName("");
	return mav;
}

@RequestMapping("/testParam")
public String testParam(Model model) {
	model.addAttribute("",""); //addAttribute其实就是向request域中设置attribute
	return "..."; //返回视图名称
}

@RequestMapping("/testParam")
public String testParam(ModelMap modelMap) {
	modelMap.addAttribute("",""); //addAttribute其实就是向request域中设置attribute
	return "..."; //返回视图名称
}
```
3. 使用map设置
```java
@RequestMapping("/testParam")
public String testParam(Map<String, Object> map) {
	map.put("", "");
	return "..."; //返回视图名称
}
```

### session域设置
```java
@RequestMapping("/testSession")
public String testSession(HttpSession session) {
	session.setAttribute("testSessionScope", "hello,session");
	return "success";
}
```

### application域设置
```java
@RequestMapping("/testApplication")
public String testApplication(HttpSession session) {
	ServletContext application = session.getServletContext();
	application.setAttribute("testApplicationScope", "hello,application");
	return "success";
}
```

## 8. 文件上传、下载
### 1. 文件上传方法
前提：引入依赖`apache.commons.fileupload apache.commons.io`
```xml
<bean id="multipartResolver" 
	  class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <!--设置请求编码-->
    <property name="defaultEncoding" value="UTF-8" />
    <property name="uploadTempDir" value="tmp" />
    <!--设置允许单个上传文件的最大值，不要在这里配置-->
    <!--<property name="maxUploadSizePerFile" value="31457280"/>-->
    <!--延迟解析，在Controller中抛出异常-->
    <property name="resolveLazily" value="true" />
</bean>
<!--设置文件上传拦截器-->
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/*upload*" />
        <bean class="com.wenshixin.interceptor.FileUploadInterceptor">
            <property name="maxSize" value="31457280" />
        </bean>
    </mvc:interceptor>
</mvc:interceptors>
```

```java
public class FileUploadInterceptor implements HandlerInterceptor {
	private long maxSize;
	@Override
	    public Boolean preHandle(HttpServletRequest httpServletRequest, 
		    HttpServletResponse httpServletResponse, Object o) throws Exception {
		if (httpServletRequest != null && 
				ServletFileUpload.isMultipartContent(httpServletRequest)) {
			ServletRequestContext servletRequestContext = new ServletRequestContext(httpServletRequest);
			long requestSize = servletRequestContext.contentLength();
			if (requestSize > maxSize) {
				// 抛出异常
				throw new MaxUploadSizeExceededException(maxSize);
			}
		}
		return true;
	}
	@Override
	public void postHandle(HttpServletRequest httpServletRequest, 
		    HttpServletResponse httpServletResponse, Object o, 
		    ModelAndView modelAndView) throws Exception {
	}
	@Override
	public void afterCompletion(HttpServletRequest httpServletRequest, 
		HttpServletResponse httpServletResponse, Object o, 
		Exception e) throws Exception {
	}
	public void setMaxSize(long maxSize) {
		this.maxSize = maxSize;
	}
}
```

```java
@ExceptionHandler(MaxUploadSizeExceededException.class)
public String handException(MaxUploadSizeExceededException e, HttpServletRequest request) {
	request.setAttribute("msg", "文件超过了指定大小，上传失败！");
	return "fileupload";
}
```

```java
@PostMapping(value = "/fileupload")
public String fileUpload(@RequestParam(value = "file") List<MultipartFile> files, HttpServletRequest request) {
	String msg = "";
	// 判断文件是否上传
	if (!files.isEmpty()) {
		// 设置上传文件的保存目录
		String basePath = request.getServletContext().getRealPath("/upload/");
		// 判断文件目录是否存在
		File uploadFile = new File(basePath);
		if (!uploadFile.exists()) {
			uploadFile.mkdirs();
		}
		for (MultipartFile file : files) {
			String originalFilename = file.getOriginalFilename();
			if (originalFilename != null && !originalFilename.equals("")) {
				try {
					// 对文件名做加UUID值处理
					originalFilename = UUID.randomUUID() + "_" + originalFilename;
					file.transferTo(new File(basePath + originalFilename));
				}
				catch (IOException e) {
					e.printStackTrace();
					msg = "文件上传失败！";
				}
			} else {
				msg = "上传的文件为空！";
			}
		}
		msg = "文件上传成功！";
	} else {
		msg = "没有文件被上传！";
	}
	request.setAttribute("msg", msg);
	return "fileupload";
}
```

### 2. 文件下载
```java
@RequestMapping("/downloadFile")
public void downloadFile(String fileName, HttpServletRequest request) throws IOException {
	//设置下载类型
	response.setContentType(MediaType.APPLICATION_OCTET_STREAM_VALUE);
	String userAgent = request.getHeader("User-Agent");
	if(userAgent.contains("IE")) {
		response.setHeader("Content-Disposition", "attachment;filename=" + 
			URLEncoder.encode(fileName, "UTF-8"));
	} else {
		response.setHeader("Content-Disposition", "attachment;filename=" + 
			new String(fileName.getBytes("UTF-8"), "ISO-8859-1"));
	}
	
	//读取服务端本地文件
	String path = request.getServletContext().getRealPath("/static");
	File file = new File(path, fileName);
	InputStream input = new FileInputStream(file);
	OutputStream out = response.getOutputStream();
	
	//向response写文件内容
	byte[] buff = new byte[file.available()];
	input.read(buff);
	out.write(buff);
	
	input.close();
	out.close();
}
```

```java
@RequestMapping("/downloadFile")
public ResponseEntity<byte[]> downloadFile(String fileName, HttpSession session) throws IOException {
	ServletContext servletContext = session.getServletContext();
	String path = servletContext.getRealPath("/static");
	File file = new File(path, fileName);
	
	InputStream input = new FileInputStream(file);
	byte[] buff = new byte[file.available()];
	input.read(buff)
	HttpHeaders headers = new HttpHeaders();
	headers.setContentDispositionFormData("attachment", fileName);
	headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
	
	return new ResponseEntity<byte[]>(buff, headers, HttpStatus.OK);
}
```

## 9. 拦截器
```java
package interceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

public class TestInterceptor implements HandlerInterceptor {
    @Override
    public void afterCompletion(HttpServletRequest request,
            HttpServletResponse response, Object handler, Exception ex)
            throws Exception {
        System.out.println("afterCompletion方法在控制器的处理请求方法执行完成后执行，即视图渲染结束之后执行");

    }

    @Override
    public void postHandle(HttpServletRequest request,
            HttpServletResponse response, Object handler,
            ModelAndView modelAndView) throws Exception {
        System.out.println("postHandle方法在控制器的处理请求方法调用之后，解析视图之前执行");
    }

    @Override
    public boolean preHandle(HttpServletRequest request,
            HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle方法在控制器的处理请求方法调用之后，解析视图之前执行");
        return false;
    }
}
```

```xml
<!-- 配置拦截器 -->
<mvc:interceptors>
    <!-- 配置一个全局拦截器，拦截所有请求 -->
    <bean class="interceptor.TestInterceptor" /> 
    <mvc:interceptor>
        <!-- 配置拦截器作用的路径 -->
        <mvc:mapping path="/**" />
        <!-- 配置不需要拦截作用的路径 -->
        <mvc:exclude-mapping path="" />
        <!-- 定义<mvc:interceptor>元素中，表示匹配指定路径的请求才进行拦截 -->
        <bean class="interceptor.Interceptor1" />
    </mvc:interceptor>
    <mvc:interceptor>
        <!-- 配置拦截器作用的路径 -->
        <mvc:mapping path="/gotoTest" />
        <!-- 定义在<mvc: interceptor>元素中，表示匹配指定路径的请求才进行拦截 -->
        <bean class="interceptor.Interceptor2" />
    </mvc:interceptor>
</mvc:interceptors>
```

## 10. 异常处理器
### 1. @ExceptionHandler
```java
@RequestMapping("/testExceptionHandle")
public String testExceptionHandle(@RequestParam("i") Integer i) {
    System.out.println(10 / i);
    return "success";
}

//在同一个类中定义处理异常的方法
@ExceptionHandler({ ArithmeticException.class })
public String testArithmeticException(Exception e) {
    System.out.println("打印错误信息 ===> ArithmeticException:" + e);
    // 跳转到指定页面
    return "error";
}
```

### 2. 自定义全局异常处理HandlerExceptionResolver
Spring MVC 通过`HandlerExceptionResolver`处理程序异常
```java
package net.biancheng.exception;

import java.util.HashMap;
import java.util.Map;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.HandlerExceptionResolver;
import org.springframework.web.servlet.ModelAndView;

public class MyExceptionHandler implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest arg0, HttpServletResponse arg1, Object arg2,
            Exception arg3) {
        Map<String, Object> model = new HashMap<String, Object>();
        // 根据不同错误转向不同页面（统一处理），即异常与View的对应关系
        if (arg3 instanceof ArithmeticException) {
            return new ModelAndView("error", model);
        }
        return new ModelAndView("error-2", model);
    }
}
```

```xml
<!--托管MyExceptionHandler-->
<bean class="net.biancheng.exception.MyExceptionHandler"/>
```

### 3. SimpleMappingExceptionResolver
```xml
<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
    <!-- 定义默认的异常处理页面，当该异常类型注册时使用 -->
    <property name="defaultErrorView" value="error"></property>
    <!-- 定义异常处理页面用来获取异常信息的变量名，默认名为exception -->
    <property name="exceptionAttribute" value="ex"></property>
    <!-- 定义需要特殊处理的异常，用类名或完全路径名作为key，异常页名作为值 -->
    <property name="exceptionMappings">
        <props>
            <prop key="ArithmeticException">error</prop>
            <!-- 在这里还可以继续扩展对不同异常类型的处理 -->
        </props>
    </property>
</bean>
```