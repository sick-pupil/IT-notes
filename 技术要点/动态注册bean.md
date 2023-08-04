## 1. BeanDefinition
```java
public <T> T registerBean(String name, Class<T> clazz, Object... args) {  
      BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(clazz);  
      if (args.length > 0) {  
          for (Object arg : args) {  
              beanDefinitionBuilder.addConstructorArgValue(arg);  
          }  
      }  
      BeanDefinition beanDefinition = beanDefinitionBuilder.getRawBeanDefinition();  
    
      BeanDefinitionRegistry beanFactory = (BeanDefinitionRegistry) applicationContext.getBeanFactory();  
      beanFactory.registerBeanDefinition(name, beanDefinition);  
      return applicationContext.getBean(name, clazz);  
}
```

## 2. BeanDefinitionRegistryPostProcessor
```java
@Slf4j
@Configuration
public class AutoBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        // 注册Bean定义，容器根据定义返回bean

        //构造bean定义
        BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder
                .genericBeanDefinition(AutoBean.class);
        BeanDefinition beanDefinition = beanDefinitionBuilder.getRawBeanDefinition();
        //注册bean定义
        registry.registerBeanDefinition("autoBean", beanDefinition);


        // AutoDIBean 的注入方式
        beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(AutoDIBean.class);
        beanDefinitionBuilder.addConstructorArgValue("自动注入依赖Bean");
        beanDefinition = beanDefinitionBuilder.getBeanDefinition();
        registry.registerBeanDefinition("autoDiBean", beanDefinition);
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory factory) throws BeansException {
        // 注册Bean实例，使用supply接口, 可以创建一个实例，并主动注入一些依赖的Bean；当这个实例对象是通过动态代理这种框架生成时，就比较有用了

        BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(AutoFacDIBean.class, () -> {
            AutoFacDIBean autoFacDIBean = new AutoFacDIBean("autoFac");
            autoFacDIBean.setAutoBean(factory.getBean("autoBean", AutoBean.class));
            autoFacDIBean.setOriginBean(factory.getBean("originBean", OriginBean.class));
            return autoFacDIBean;
        });
        BeanDefinition beanDefinition = builder.getRawBeanDefinition();
        ((DefaultListableBeanFactory) factory).registerBeanDefinition("autoFacDIBean", beanDefinition);
    }
}
```

**基于多数据源的应用场景**
```java
 private synchronized void initTargetPool(Database database) {      
        String wDataSourceId = database.getName() + database.getSeq() + database.getMode() + DATASOURCE_NAME;    
        String wSqlSessionFactoryId = database.getName() + database.getSeq() + database.getMode();     
        BeanDefinitionBuilder wBeanDefBuilder = null;    
        wBeanDefBuilder = BeanDefinitionBuilder.genericBeanDefinition(MyDataSource.class);     
        wBeanDefBuilder.getBeanDefinition().setAttribute(ID, wDataSourceId);   
        wBeanDefBuilder.setScope(SINGLETON);   
        wBeanDefBuilder.addPropertyValue(URL, database.getUrl());   
        wBeanDefBuilder.addPropertyValue(USERNAME, database.getUser());     
        wBeanDefBuilder.addPropertyValue(DB_PW, database.getPassword());    
        wBeanDefBuilder.addPropertyValue(DRIVERCLASSNAME, EnumType.DriverClass.getValue(database.getType()));    
        wBeanDefBuilder.addPropertyValue(DEFAULTAUTOCOMMIT, false);       
        wBeanDefBuilder.addPropertyValue(INITIALSIZE, 1);      
        wBeanDefBuilder.addPropertyValue(MINIDLE, 1);     
        wBeanDefBuilder.addPropertyValue(MAXACTIVE, 10);  
        wBeanDefBuilder.addPropertyValue("testWhileIdle", false);  
        wBeanDefBuilder.addPropertyValue("testOnBorrow", true);     
        wBeanDefBuilder.addPropertyValue("maxWait", 30000);
        //        wBeanDefBuilder.addPropertyValue("validationQuery", getCheckSql(database.getType()));
        
        Properties properties = new Properties();   
        switch (database.getType()) {       
            case ORACLE:             
                properties.setProperty("v$session.program", "dbmu");       
                break;       
            case DB2:          
                properties.setProperty("clientProgramName", "dbmu");         
                break; 
            case MYSQL:    
                properties.setProperty("connectionAttributes", "program_name:dbmu");          
                break;        
            case SQLSERVER:         
                properties.setProperty("applicationName", "dbmu");           
                break;        
            default:             
                throw new AppException("DBMU000", "不支持的数据库类型");    
        }       
        wBeanDefBuilder.addPropertyValue("connectProperties", properties);   
        
        //注册数据源   
        SpringUtil.getApplicationContext().registerBeanDefinition(wDataSourceId,wBeanDefBuilder.getRawBeanDefinition());  
        
        Object wDataSourceBean = SpringUtil.getApplicationContext().getBean(wDataSourceId);  
        wBeanDefBuilder = BeanDefinitionBuilder.genericBeanDefinition(SqlSessionFactoryBean.class);     
        wBeanDefBuilder.getBeanDefinition().setAttribute(ID, wSqlSessionFactoryId);    
        wBeanDefBuilder.addPropertyValue(DATASOURCE, wDataSourceBean);      
        try {            
            wBeanDefBuilder.addPropertyValue("mapperLocations",new PathMatchingResourcePatternResolver().getResources(String.format(DAO_PATH, database.getType().toLowerCase())));     
        } catch (Exception e) {    
            throw new AppException("DBMU000", "添加数据库配置文件失败：" + e.getMessage());    
        }
       wBeanDefBuilder.addPropertyValue("plugins", interceptorsProvider.getIfAvailable());        
        
        // 注册SqlSessionFactoryId 
        SpringUtil.getApplicationContext().registerBeanDefinition(wSqlSessionFactoryId,wBeanDefBuilder.getRawBeanDefinition());
    }
```

`Spring`容器初始化时扫描`Bean`实例，通过`BeanDefinitionRegistryPostProcessor`获取`Bean`
`Spring`容器读取到`Bean`相关定义后，保存在`BeanDefinitionMap`，在实例化所有`bean`之前允许自定义拓展改变`bean`定义，此处发挥作用的是`BeanFactoryPostProcessor`

```java
public void refresh() throws BeansException, IllegalStateException
{
    synchronized(this.startupShutdownMonitor)
    {
        prepareRefresh();
        //这里是在子类中启动refreshBeanFactory（） 的地方
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory(); 
        //Prepare the bean factory for use in this context.
        prepareBeanFactory(beanFactory);
        try {
            //设置BeanFactoy的后置处理
            postProcessBeanFactory(beanFactory);
            //调用BeanFactory的后处理器， 这些后处理器是在Bean定义中向容器注册的
            invokeBeanFactoryPostProcessors(beanFactory);
            //注册Bean的后处理器， 在Bean创建过程中调用
            registerBeanPostProcessors(beanFactory);
            //对上下文中的消息源进行初始化
            initMessageSource();
            //初始化上下文中的事件机制
            initApplicationEventMulticaster();
            //初始化其他的特殊Bean
            onRefresh();
            //检查监听Bean并且将这些Bean向容器注册
            registerListeners(): 
			//实例化所有的（ non - lazy - init） 单件
            finishBeanFactoryInitialization(beanFactory);
            //发布容器事件， 结束Refresh过程
            finishRefresh();
        }
        catch(BeansException ex)
        {
            //为防止Bean资源占用， 在异常处理中， 销毁已经在前面过程中生成的单件Bean
            destroyBeans();
            //重置＇ active＇ 标志 
            cancelRefresh(ex);
            throw ex;
        }
    }
}
```