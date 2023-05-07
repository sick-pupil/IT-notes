## 1. 逆向工程
```java
//省略了import
public class MysqlGenerator {

    public static void main(String[] args) {
		// 1、全局配置
        GlobalConfig globalConfig = new GlobalConfig();//构建全局配置对象
        String projectPath = System.getProperty("user.dir");// 获取当前用户的目录
        globalConfig
                .setOutputDir(projectPath + "/mybatis-plus-01-start/src/main/java")// 输出文件路径
                .setAuthor("微信搜一搜：贺贺学编程")// 设置作者名字
                .setOpen(false)// 是否打开资源管理器
                .setFileOverride(true)// 是否覆盖原来生成的
                .setIdType(IdType.AUTO)// 主键策略
                .setBaseResultMap(true)// 生成resultMap
                .setBaseColumnList(true)// XML中生成基础列
                .setServiceName("%sService");// 生成的service接口名字首字母是否为I，这样设置就没有I

        // 2、数据源配置
        DataSourceConfig dataSourceConfig = new DataSourceConfig();// 创建数据源配置
        dataSourceConfig
                .setUrl("jdbc:mysql://127.0.0.1:3306/mybatis_plus?userSSL=false&useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC")
                .setDriverName("com.mysql.cj.jdbc.Driver")
                .setUsername("root")
                .setPassword("root")
                .setDbType(DbType.MYSQL);

        // 3、包配置
        PackageConfig packageConfig = new PackageConfig();
        packageConfig
                .setParent("com.hzy")
                .setEntity("entity")
                .setController("controller")
                .setService("service")
                .setMapper("mapper");

        // 4、策略配置
        StrategyConfig strategyConfig = new StrategyConfig();
        strategyConfig
                .setCapitalMode(true)// 开启全局大写命名
                .setInclude("user")// 设置要映射的表
                .setNaming(NamingStrategy.underline_to_camel)// 下划线到驼峰的命名方式
                .setColumnNaming(NamingStrategy.underline_to_camel)// 下划线到驼峰的命名方式
                .setEntityLombokModel(false)// 是否使用lombok
                .setRestControllerStyle(true)// 是否开启rest风格
                .setControllerMappingHyphenStyle(true);// localhost:8080/hello_a_2


        // 5、自定义配置（配置输出xml文件到resources下）
        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                // to do nothing
            }
        };
        List<FileOutConfig> focList = new ArrayList<>();
        String templatePath = "/templates/mapper.xml.vm";
		
		// 如果模板引擎是 freemarker
        //String templatePath = "/templates/mapper.xml.ftl";
        // 如果模板引擎是 velocity
        // String templatePath = "/templates/mapper.xml.vm";
		
		
        // 自定义配置会被优先输出
        focList.add(new FileOutConfig(templatePath) {
            @Override
            public String outputFile(TableInfo tableInfo) {
                // 自定义输出文件名 ， 如果你 Entity 设置了前后缀、此处注意 xml 的名称会跟着发生变化！！
                return projectPath + "/mybatis-plus-01-start/src/main/resources/mapper/"
                        + tableInfo.getEntityName() + "Mapper" + StringPool.DOT_XML;
            }
        });
        cfg.setFileOutConfigList(focList);

        // 6、整合配置
        AutoGenerator autoGenerator = new AutoGenerator();// 构建代码生自动成器对象
        autoGenerator
                .setGlobalConfig(globalConfig)// 将全局配置放到代码生成器对象中
                .setDataSource(dataSourceConfig)// 将数据源配置放到代码生成器对象中
                .setPackageInfo(packageConfig)// 将包配置放到代码生成器对象中
                .setStrategy(strategyConfig)// 将策略配置放到代码生成器对象中
                .setCfg(cfg)// 将自定义配置放到代码生成器对象中
                .execute();// 执行！
    }

}
```

## 2. 基本使用
```xml
<dependency>
	<groupId>com.baomidou</groupId>
	<artifactId>mybatis-plus-boot-starter</artifactId>
	<version>3.0.5</version>
</dependency>

<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.3.2</version>
</dependency>

<!-- 代码生成器模板 -->
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
</dependency>
```

```java
@SpringBootApplication
@MapperScan("cn.com.vicente.demo.mapper")
public class BdDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(BdDemoApplication.class, args);
    }

}
```

```java
@Data
@TableName(value = "tb_employee")//指定表名
public class Employee {
    //value与数据库主键列名一致，若实体类属性名与表主键列名一致可省略value
    @TableId(value = "id",type = IdType.AUTO)//指定自增策略
    private Integer id;
    //若没有开启驼峰命名，或者表中列名不符合驼峰规则，可通过该注解指定数据库表中的列名，exist标明数据表中有没有对应列
    @TableField(value = "last_name",exist = true)
    private String lastName;
    private String email;
    private Integer gender;
    private Integer age;
}
```

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SampleTest {

    private static Logger log = LoggerFactory.getLogger(SampleTest.class);

    @Autowired
    private MpUserService mpUserService;

    @Test
    public void test1() {
        // 插入新记录
        MpUser mpUser = new MpUser();
        //mpUser.setId(1L);
        mpUser.setEmail("test66@baomidou.com");
        mpUser.setAge(22);
        mpUser.setName("David Hong");
        mpUserService.save(mpUser);
        // 或者
        mpUser.insertOrUpdate();
        // 更新完成后，mpUser对象的id会被补全
        log.info("mpUser={}", mpUser.toString());

    }

    @Test
    public void test2() {
        // 通过主键id查询
        MpUser mpUser = mpUserService.getById(1);
        log.info("mpUser={}", mpUser.toString());
    }

    @Test
    public void test3() {
        // 条件查询，下面相当于xml中的 select * from mp_user where name = 'Tom' and age = '28' limit 1
        MpUser mpUser = mpUserService.getOne(new QueryWrapper<MpUser>().eq("name", "Tom").eq("age", "28").last("limit 1"));
        log.info("mpUser={}", mpUser.toString());
        // 批量查询
        List<MpUser> mpUserList = mpUserService.list();
        System.out.println("------------------------------all");
        mpUserList.forEach(System.out::println);
        // 分页查询 
        int pageNum = 1;
        int pageSize = 10;
        IPage<MpUser> mpUserIPage = mpUserService.page(new Page<>(pageNum, pageSize), new QueryWrapper<MpUser>().gt("age", "20"));
        // IPage to List
        List<MpUser> mpUserList1 = mpUserIPage.getRecords();
        System.out.println("------------------------------page");
        mpUserList1.forEach(System.out::println);
        // 总页数
        long allPageNum = mpUserIPage.getPages();
        System.out.println("------------------------------allPageNum");
        System.out.println(allPageNum);
    }

     @Test
    public void test4() {
        MpUser mpUser = mpUserService.getById(2);
        // 修改更新
        mpUser.setName("广东广州");
        //mpUserService.updateById(mpUser);
        // 或者
        mpUser.insertOrUpdate();
        // 通过主键id删除
        mpUserService.removeById(1);
        // 或者
        //mpUser.deleteById();
    }


}
```

## 3. Wrapper条件筛选
```java
int pageNum = 1;
int pageSize = 10;
IPage<MpUser> mpUserIPage = mpUserService.selectPage(new Page<>(pageNum, pageSize), new QueryWrapper<MpUser>().gt("age", "20"));
//Page通过缓存来获得全部数据中再进行的分页，会浪费一定性能

List<Employee> employees = emplopyeeDao.selectPage(new Page<Employee>(1,3),
     new EntityWrapper<Employee>()
        .between("age",18,50)
        .eq("gender",0)
        .eq("last_name","tom")
);

List<Employee> employees = emplopyeeDao.selectList(
                new EntityWrapper<Employee>()
               .eq("gender",0)
               .like("last_name","老师")
                //.or()//和or new 区别不大
               .orNew()
               .like("email","a")
);

List<Employee> employees = emplopyeeDao.selectList(
                new EntityWrapper<Employee>()
                .eq("gender",0)
                .orderBy("age")//直接orderby 是升序，asc
                .last("desc limit 1,3")//在sql语句后面追加last里面的内容(改为降序，同时分页)
);

List<Employee> employees = emplopyeeDao.selectPage(
			new Page<Employee>(1,2),
			Condition.create()
					.between("age",18,50)
					.eq("gender","0")
);

public void testEntityWrapperUpdate(){
        Employee employee = new Employee();
        employee.setLastName("苍老师");
        employee.setEmail("cjk@sina.com");
        employee.setGender(0);
        emplopyeeDao.update(employee,
                new EntityWrapper<Employee>()
                .eq("last_name","tom")
                .eq("age",25)
        );
}

emplopyeeDao.delete(
        new EntityWrapper<Employee>()
        .eq("last_name","tom")
        .eq("age",16)
);

```

## 4. 分页拦截器
```java
//mybatis-plus 3.4.0版本之前用法
@Configuration
@MapperScan(basePackages = {"com.zimug.**.mapper"})
public class MybatisPlusConfig {

    @Bean
    public PaginationInterceptor paginationInterceptor() {
        PaginationInterceptor paginationInterceptor = new PaginationInterceptor();
        // 设置请求的页面大于最大页后操作， true调回到首页，false 继续请求  默认false
        // paginationInterceptor.setOverflow(false);
        // 设置最大单页限制数量，默认 500 条，-1 不受限制
        // paginationInterceptor.setLimit(500);
        // 开启 count 的 join 优化,只针对部分 left join
        paginationInterceptor.setCountSqlParser(new JsqlParserCountOptimize(true));
        return paginationInterceptor;
    }
}

//mybatis-plus 3.4.0版本之后用法
@Configuration
@MapperScan(basePackages = {"com.zimug.**.mapper"})
public class MybatisPlusConfig {

  /**
   * 新的分页插件,一缓和二缓遵循mybatis的规则,需要设置 MybatisConfiguration#useDeprecatedExecutor = false 避免缓存出现问题(该属性会在旧插件移除后一同移除)
   */
  @Bean
  public MybatisPlusInterceptor mybatisPlusInterceptor() {
    MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
    //向Mybatis过滤器链中添加分页拦截器
    interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
    //还可以添加i他的拦截器
    return interceptor;
  }

  @Bean
  public ConfigurationCustomizer configurationCustomizer() {
    return configuration -> configuration.setUseDeprecatedExecutor(false);
  }
}
```

至于分页查询使用，mybatis-plus与mybatis用法无差别