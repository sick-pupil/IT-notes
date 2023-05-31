## 1. Dubbo分层架构
<img src="D:\Project\IT-notes\框架or中间件\Dubbo\img\Dubbo底层分层.png" style="width:700px;height:500px;" />

- `Service`，业务层，就是咱们开发的业务逻辑层。
- `Config`，配置层，主要围绕`ServiceConfig`和`ReferenceConfig`，初始化配置信息。
- `Proxy`，代理层，服务提供者还是消费者都会生成一个代理类，使得服务接口透明化，代理层做远程调用和返回结果。
- `Register`，注册层，封装了服务注册和发现。
- `Cluster`，路由和集群容错层，负责选取具体调用的节点，处理特殊的调用要求和负责远程调用失败的容错措施。
- `Monitor`，监控层，负责监控统计调用时间和次数。
- `Portocol`，远程调用层，主要是封装RPC调用，主要负责管理`Invoker`，`Invoker`代表一个抽象封装了的执行体，之后再做详解。
- `Exchange`，信息交换层，用来封装请求响应模型，同步转异步。
- `Transport`，网络传输层，抽象了网络传输的统一接口，这样用户想用`Netty`就用`Netty`，想用`Mina`就用`Mina`。
- `Serialize`，序列化层，将数据序列化成二进制流，当然也做反序列化

## 2. 配置文件解析
Spring解析配置文件标签存在一个总接口`BeanDefinitionParser`，而Dubbo存在`DubboBeanDefinitionParser`，其中主要是`parser方法`进行解析

### parse方法
```java
@SuppressWarnings("unchecked")
private static BeanDefinition parse(Element element, ParserContext parserContext, Class<?> beanClass, boolean required) {
	//beanClass对应每一个配置标签，每个标签对应一个Config类
	...
	
	if (ProtocolConfig.class.equals(beanClass)) {
		for (String name : parserContext.getRegistry().getBeanDefinitionNames()) {
			BeanDefinition definition = parserContext.getRegistry().getBeanDefinition(name);
			PropertyValue property = definition.getPropertyValues().getPropertyValue("protocol");
			if (property != null) {
				Object value = property.getValue();
				if (value instanceof ProtocolConfig && id.equals(((ProtocolConfig) value).getName())) {
					definition.getPropertyValues().addPropertyValue("protocol", new RuntimeBeanReference(id));
				}
			}
		}
	} else if (ServiceBean.class.equals(beanClass)) {
		String className = element.getAttribute("class");
		if (className != null && className.length() > 0) {
			RootBeanDefinition classDefinition = new RootBeanDefinition();
			classDefinition.setBeanClass(ReflectUtils.forName(className));
			classDefinition.setLazyInit(false);
			parseProperties(element.getChildNodes(), classDefinition);
			beanDefinition.getPropertyValues().addPropertyValue("ref", new BeanDefinitionHolder(classDefinition, id + "Impl"));
		}
	}
	...
}
```

1. Spring容器启动
2. 加载配置文件
3. `DubboNameSpaceHandler`名称空间处理器会创建dubbo标签解析器`DubboBeanDefinitionParser`
4. 标签解析器`DubboBeanDefinitionParser`会解析Dubbo每个标签，每个标签都会解析出一个Config类与标签一一对应
5. 标签解析出对应类时，存在`<dubbo:service>`标签解析出`ServiceBean`，`ServiceBean`解析的过程涉及到服务暴漏的过程

## 3. 服务暴漏
<img src="D:\Project\IT-notes\框架or中间件\Dubbo\img\服务暴露.png" style="width:700px;height:350px;" />

`ServiceBean`实现了两个重要机制：
1. 实现Spring接口`InitializingBean`，当`ServiceBean`创建成功后，会调用接口的`afterPropertiesSet`方法
2. `ServiceBean`还实现了`ApplicationListener`接口，创建IOC容器并完成Bean注册后，会回调接口的`onApplicationEvent`方法

```java
public void afterPropertiesSet() throws Exception {
	if (getProvider() == null) {
		..............                 
		//获取provider配置
		...
		//保存配置
		setProvider(providerConfig);
	}
	if (getApplication() == null && (getProvider() == null || getProvider().getApplication() == null)) {
		...............　　　　　　　　　 
		//获取application配置       
		...
		//保存配置
		setApplication(applicationConfig);     
	}
	if (getModule() == null && (getProvider() == null || getProvider().getModule() == null)) {
		...............　　　　　　　　　 
		//获取module配置            
	}
	if ((getRegistries() == null || getRegistries().size() == 0)
		&& (getProvider() == null || getProvider().getRegistries() == null || 
		getProvider().getRegistries().size() == 0)
		&& (getApplication() == null || getApplication().getRegistries() == null || 
		getApplication().getRegistries().size() == 0)) {
		.................                
		//获取注册中心的配置                   
		...
		//保存配置
		setRegistries(registriesConfig);
	}
	if (getMonitor() == null && (getProvider() == null || getProvider().getMonitor() == null)
		 && (getApplication() == null || getApplication().getMonitor() == null)) {
		................
		//获取monitor配置                  
		...
		//保存配置
		setMonitor(monitorConfig);
	}
	if ((getProtocols() == null || getProtocols().size() == 0)
		 && (getProvider() == null || getProvider().getProtocols() == null || 
		 getProvider().getProtocols().size() == 0)) {
		...............               
		//获取protocol配置                  
		...
		//保存配置
		setProtocols(protocolConfigs);
	}
	if (getPath() == null || getPath().length() == 0) {
		//获取<dubbo:service/>的path属性，path即服务的发布路径
		if (beanName != null && beanName.length() > 0 
			&& getInterface() != null && getInterface().length() > 0
			&& beanName.startsWith(getInterface())) {
			setPath(beanName);
			//如果没有设置path属性，则默认会以beanName作为path
		}
	}
	if (!isDelay()) {
		//是否延迟暴露
		export();
		//进行服务暴露
	}
}
```

```java
public void onApplicationEvent(ContextRefreshedEvent event) {
	//不是延迟暴漏、还没暴漏、需要暴漏的服务
	if(!isDelay() && !isExported() && !isUnexported()) {
		if(logger.isInfoEnabled()) {
			logger.info("The service ready on spring started. service: " + getInterface());
		}
		export();
	}
}

public synchronized void export() {
	if(provider != null) {
		if(export == null) {
			export = provider.getExport();
		}
		if(delay == null) {
			delay = provider.getDelay();
		}
	}
	//如果只想本地启动服务而不把服务暴露出去，配置<dubbo:provider export="false" />
	if(export != null && !export) {
		return ;
	}
	//如果应该延迟导出，则延迟导出服务
	if(delay != null && delay > 0) {
		delayExportExecutor.schedule(new Runnable() {
			@Override
			public void run() {
				doExport();
			}
		}, delay, TimeUnit.MILLISECONDS);
	} else {
		//立即导出服务
		doExport();
	}
}

protected synchronized void doExport() {
	...
	// 多协议多注册中心暴露服务
	doExportUrls();
	// ProviderModel 表示服务提供者模型，此对象中存储了与服务提供者相关的信息。
	// 比如服务的配置信息，服务实例等。每个被导出的服务对应一个 ProviderModel。
	// ApplicationModel 持有所有的 ProviderModel。
	ProviderModel providerModel = new ProviderModel(getUniqueServiceName(), this, ref);
	ApplicationModel.initProviderModel(getUniqueServiceName(), providerModel);
}

private void doExportUrls() {
	//loadRegistries加载注册中心链接
	List<URL> registryURLs = loadRegistries(true);
	//遍历协议，在每个协议下暴露服务
	for(ProtocolConfig protocolConfig : protocols) {
		doExportUrlsFor1Protocol(protocolConfig, registryURLs);
	}
}

private void doExportUrlsFor1Protocol(ProtocolConfig protocolConfig, List<URL> registryURLs) {
	String name = protocolConfig.getName();
	if(name == null || name.length() == 0) {
		name = "dubbo";
	}
	
	Map<String, String> map = new HashMap<String, String>();
	map.put(Constants.SIDE_KEY, Constants.PROVIDER_SIDE);
	map.put(Constants.DUBBO_VERSION_KEY, Version.getVersion());
	...
	//代理类工厂获取服务接口代理执行类
	Invoker<?> invoker = proxyFactory.getInvoker(ref, (Class)interfaceClass, registryURL.addParameterAndEncoded(Constant....))
	DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);
	
	//暴露服务类
	Exporter<?> exporter = protocol.export(wrapperInvoker);
	exporters.add(exporter);
}

public <T> Exporter<T> export(Invoker<T> invoker) throws RpcException {
	URL url = invoker.getUrl();
	
	String key = serviceKey(url);
	DubboExporter<T> exporter = new DubboExporter<T>(invoker, key, exporterMap);
	exporterMap.put(key, exporter);
	...
	openServer(url);
	...
}

//实际启动netty服务器，监听20880端口
private void openServer(URL url) {
	String key = url.getAddress();
	
}

//注册生产者和消费者
ProviderConsumerRegTable.registerProvider(originInvoker, registryUrl, registedProviderUrl);

public class ProviderConsumerRegTable {
	//注册表，生产者与消费者的URL与执行器缓存，执行器中才有真正的执行对象
	public static ConcurrentHashMap<String, Set<ProviderInvokerWrapper>> providerInvokers = new ...
	public static ConcurrentHashMap<String, Set<ConsumerInvokerWrapper>> consumerInvokers = new ...
	
	public static void registerProvider(Invoker invoker, URL registryUrl, URL providerUrl) {
		
	}
}
```

## 4. 获取服务引用
<img src="D:\Project\IT-notes\框架or中间件\Dubbo\img\服务引用.png" style="width:700px;height:350px;" />

在解析Dubbo配置文件生成`ServiceBean`的同时，也会由<dubbo:reference>标签生成`ReferenceBean`

```java
public class ReferenceBean<T> extends ReferenceConfig<T> implements FactoryBean, ...
{
	//Bean工厂获取代理对象
	@Override
	public Object getObject() throws Exception {...}
}

public synchronized T get() {
	...
	//初始化生成代理对象
	if(ref == null) {
		init();
	}
	return ref;
}

private void init() {
	...
	//收集消费者信息并创建代理对象
	ref = createProxy(map);
	...
}

private T createProxy(Map<String, String> map) {
	...
	if(urls.size() == 1) {
		//远程从注册中心获取引用接口执行器
		invoker = refprotocol.refer(interfaceClass, urls.get(0));
	} else {
		...
	}
	...
}

public <T> Invoker<T> refer(Class<T> serviceType, URL url) throws RpcException {
	...
	//获取注册中心信息
	Registry registry = registryFactory.getRegistry(url);
	...
	//远程引用
	return doRefer(cluster, registry, type, url);
}

private <T> Invoker<T> doRefer(Cluster cluster, Registry registry, Class<T> type, URL url) {
	...
	//订阅生产者服务
	directory.subscribe(subscribeUrl.addParameter(Constants.CATEGORY_KEY,
		Constants.PROVIDERS_CATEGORY + "," + Constants.CONFIGURATORS_CATEGORY + "," + 
		Constants.ROUTERS_CATEGORY));
	//获取了invoker调用者
	Invoker invoker = cluster.join(directory);
	//缓存生产者消费者信息
	ProviderConsumerRegTable.registerConsumer(invoker, url, subscribeUrl, directory);
	return invoker;
}

public <T> Invoker<T> refer(Class<T> serviceType, URL url) throws RpcException {
	...
	//根据生产者地址、服务接口和客户端，返回执行者，底层实则创建netty客户端连接20880端口监听
	DubboInvoker<T> invoker = new DubboInvoker<T>(serviceType, url, getClients(url), invokers);
	invokers.add(invoker);
	return invoker;
}
```

## 5. 服务调用
<img src="D:\Project\IT-notes\框架or中间件\Dubbo\img\服务调用流程.png" style="width:700px;height:600px;" />

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
	...
	return invoker.invoke(new RpcInvocation(method, args)).recreate();
}

public Result invoke(Invocation invocation) throws RpcException {
	...
	//this.invoker为FailoverClusterInvoker，容错Invoker
	result = this.invoker.invoke(invocation);
	...
}

public Result invoke(final Invocation invocation) throws RpcException {
	...
	//从注册中心查询出所有Invokers
	List<Invoker<T>> invokers = list(invocation);
	if(invokers != null && !invokers.isEmpty()) {
		//获取负载均衡机制
		loadbalance = ExtensionLoader.getExtensionLoader(LoadBalance.class)
				.getExtension(invokers.get...)...
		...
	}
	...
	return doInvoke(invocation, invokers, loadbalance);
}

public Result doInvoke(Invocation invocation, final List<Invoker<T>> invokers, 
						LoadBalance loadbalance) {
	...
	//根据负载均衡策略选择invoker
	Invoker<T> invoker = select(loadbalance, invocation, copyinvokers, invoked);
	...
	try {
		...
		Result result = invoker.invoke(invocation);
		...
	}
}

//经过多层filter最后到DubboInvoker
protected Result doInvoke(final Invocation invocation) throws Throwable {
	...
	//获取服务引用的客户端
	ExchangeClient currentClient;
	if(clients.length == 1) {
		currentClient = clients[0];
	} else {
		currentClient = clients[index.getAndIncrement() % clients.length];
	}
	...
	if (...) {
		...
	} else {
		RpcContext.getContext().setFuture(null);
		//客户端发起请求，请求结果获取并返回
		return (Result)currentClient.request(inv, timeout).get();
	}
}

pubilc ResponseFuture request(Object request, int timeout) throws RemotingException {
	return client.request(request, timeout);
}

pubilc ResponseFuture request(Object request, int timeout) throws RemotingException {
	return channel.request(request, timeout);
}

public ResponseFuture request(Object request, int timeout) throws RemotinException {
	...
	Request req = new Request();
	req.set...
	req.set...
	try {
		channel.send(req);
	}...
}
```
