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

## 4. 通用方法
使用`IService<Model> ServiceImpl<? extends BaseMapper, Model> BaseMapper<Model>`拓展`Service`与`Mapper`层的增删查改方法
```java
public interface IBookService extends IService<Book> { }

@Service
public class BookServerImpl extends ServiceImpl<BookMapper, Book> implements IBookService{ }

public interface BookMapper extends BaseMapper<Book> { 
	@Select("SELECT * FROM `tb_area`") 
	List<Book> selectList(); 
}

@Data 
@EqualsAndHashCode(callSuper = false) 
@Accessors(chain = true) 
@TableName("tb_area") 
public class Book extends Model<Book> implements Serializable { 
	
	@TableId(value = "area_id", type = IdType.AUTO) 
	private Integer areaId; 
	
	private String areaName; 
	private int priority; 
}
```

## 5. 分页拦截器
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

## 6. Mapper层选装方法
`mapper`层提供选装方法，调用该类选装方法时可以设置`insert`、`update`、`delete`操作中，选填某类字段作更新操作，包含`mapper`层中的三个方法：
- `alwaysUpdateSomeColumnById`：用于在更新操作时，无论实体对象的某些字段是否有变化，都会强制更新这些字段
```java
@Component
public class MySqlInjector extends DefaultSqlInjector {
    @Override
    public List<AbstractMethod> getMethodList(Class<?> mapperClass) {
        List<AbstractMethod> methodList = super.getMethodList(mapperClass);
        
        //省略其他选装件和自定义方法...

        //3.官方选装件，根据Id更新固定几个字段
        methodList.add(new AlwaysUpdateSomeColumnById(new Predicate<TableFieldInfo>() {
            @Override
            public boolean test(TableFieldInfo info) {
                //当更新时，忽略name字段，其他非逻辑删除字段会进行更新
                return !info.getColumn().equals("name");
            }
        }));
        return methodList;
    }
}

public interface MyMapper<T> extends BaseMapper<T> {
    //省略其他方法...

    /**
     * 根据Id，更新固定几个字段
     */
    int alwaysUpdateSomeColumnById(@Param(Constants.ENTITY) T entity);
}
```

- `insertBatchSomeColumn`：用于批量插入实体对象，但只插入实体对象中指定的某些字段
```java
@Component
public class MySqlInjector extends DefaultSqlInjector {
    @Override
    public List<AbstractMethod> getMethodList(Class<?> mapperClass) {
        List<AbstractMethod> methodList = super.getMethodList(mapperClass);
        //加入自定义的删除所有的方法
        methodList.add(new DeleteAllMethod());
        //1.官方选装件，批量插入，支持排除某些字段不进行批量插入的字段中
        methodList.add(new InsertBatchSomeColumn(new Predicate<TableFieldInfo>() {
            @Override
            public boolean test(TableFieldInfo info) {
                //返回true，代表加入插入字段中，false则为不加入
                //除了逻辑删除的字段，其他都加入
                return !info.isLogicDelete();
                //如果还想排除其他字段，使用getColumn()方法获取列名
                //return !info.isLogicDelete() && !info.getColumn().equals("age");
            }
        }));
        return methodList;
    }
}


public interface MyMapper<T> extends BaseMapper<T> {
    /**
     * 自定义sql注入方法，即使不写sql语句，也能删除所有，返回影响行数
     */
    int deleteAll();

    /**
     * 批量插入
     */
    int insertBatchSomeColumn(List<T> list);
}
```

- `logicDeleteByIdWithFill`：用于逻辑删除记录，并填充实体对象中的某些字段
```java
@Component
public class MySqlInjector extends DefaultSqlInjector {
    @Override
    public List<AbstractMethod> getMethodList(Class<?> mapperClass) {
        List<AbstractMethod> methodList = super.getMethodList(mapperClass);

        //省略其他选装件和自定义方法...

        //2.官方选装件，逻辑删除时，自动填充其他字段（例如逻辑删除时，自动填充删除人是谁，什么时候删除的）
        methodList.add(new LogicDeleteByIdWithFill());        
        return methodList;
    }
}

public interface MyMapper<T> extends BaseMapper<T> {
    //省略其他方法...

    /**
     * 逻辑删除，并且带自动填充
     */
    int deleteByIdWithFill(T entity);
}
```

## 7. 链式操作查询更新
- `QueryChainWrapper<T> LambdaQueryChainWrapper<T>`：
```java
// 普通链式查询示例
query().eq("name", "John").list(); // 查询 name 为 "John" 的所有记录

// lambda 链式查询示例
lambdaQuery().eq(User::getAge, 30).one(); // 查询年龄为 30 的单条记录
```
- `UpdateChainWrapper<T> LambdaUpdateChainWrapper<T>`：
```java
// 普通链式更新示例
update().set("status", "inactive").eq("name", "John").update(); // 将 name 为 "John" 的记录 status 更新为 "inactive"

// lambda 链式更新示例
User updateUser = new User();
updateUser.setEmail("new.email@example.com");
lambdaUpdate().set(User::getEmail, updateUser.getEmail()).eq(User::getId, 1).update(); // 更新 ID 为 1 的用户的邮箱
```
## 8. Wrappers
```java
// 创建 QueryWrapper
QueryWrapper<User> queryWrapper = Wrappers.query();
queryWrapper.eq("name", "张三");

// 创建 LambdaQueryWrapper
LambdaQueryWrapper<User> lambdaQueryWrapper = Wrappers.lambdaQuery();
lambdaQueryWrapper.eq(User::getName, "张三");

// 创建 UpdateWrapper
UpdateWrapper<User> updateWrapper = Wrappers.update();
updateWrapper.set("name", "李四");

// 创建 LambdaUpdateWrapper
LambdaUpdateWrapper<User> lambdaUpdateWrapper = Wrappers.lambdaUpdate();
lambdaUpdateWrapper.set(User::getName, "李四");
```
## 9. 查询流式处理
- `getResultObject`：获取数据库中的每一条记录
- `getResultCount`：获取当前处理的结果集条数，每处理一条记录，该计数器会加1，计数从1开始
- `stop`：停止继续处理结果集，相当于在循环中使用 `break` 语句

```java
// 结合分页，按批次从数据库拉取数据出来跑批，例如从数据库获取10万记录，做数据处理
Page<H2User> page = new Page<>(1, 100000);
baseMapper.selectList(page, Wrappers.emptyWrapper(), new ResultHandler<H2User>() {
    int count = 0;
    @Override
    public void handleResult(ResultContext<? extends H2User> resultContext) {
        H2User h2User = resultContext.getResultObject();
        System.out.println("当前处理第" + (++count) + "条记录: " + h2User);
        // 在这里进行你的业务处理，比如分发任务
    }
});

// 从数据库获取表所有记录，做数据处理
baseMapper.selectList(Wrappers.emptyWrapper(), new ResultHandler<H2User>() {
    int count = 0;
    @Override
    public void handleResult(ResultContext<? extends H2User> resultContext) {
        H2User h2User = resultContext.getResultObject();
        System.out.println("当前处理第" + (++count) + "条记录: " + h2User);
        // 在这里进行你的业务处理，比如分发任务
    }
});
```
## 10. 字段自动填充
1. 使用 `@TableField` 注解来标记哪些字段需要自动填充，并指定填充的策略
```java
public class User {
    @TableField(fill = FieldFill.INSERT)
    private String createTime;

    @TableField(fill = FieldFill.UPDATE)
    private String updateTime;

    // 其他字段...
}
```
2. 创建一个类来实现 `MetaObjectHandler` 接口，并重写 `insertFill` 和 `updateFill` 方法
```java
// java example
@Slf4j
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {

    @Override
    public void insertFill(MetaObject metaObject) {
        log.info("开始插入填充...");
        this.strictInsertFill(metaObject, "createUserId", Long.class, 123456L)
        this.strictInsertFill(metaObject, "createTime", LocalDateTime.class, LocalDateTime.now());
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        log.info("开始更新填充...");
        this.strictInsertFill(metaObject, "updateUserId", Long.class, 123456L)
        this.strictUpdateFill(metaObject, "updateTime", LocalDateTime.class, LocalDateTime.now());
    }
}
```
注意：
1. 使用 `strictInsertFill` 或 `strictUpdateFill` 方法可以根据注解 `FieldFill.xxx`、字段名和字段类型来区分填充逻辑；如果不需区分，可以使用 `fillStrategy` 方法
2. 若是自定义方法，则需要所在`mapper`的实体类参数名为`et`：
`updateFillByCustomMethod3(@Param("coll") Collection<Long> ids, @Param("user") H2User h2User);`

## 11. 主键生成策略
`@TableId`中`type`参数决定主键生成策略：
- `AUTO`：库表主键没有自动填充值规则，则报错
- `NONE`：使用默认主键生成策略，即`ASSIGN_ID`
- `INPUT`：`insert`时需要主动填充主键值，采用`IKeyGenerator`类型`ID`生成器时需要为此类
- `ASSIGN_ID`：分配`ID`，使用接口`IdentifierGenerator`中的方法`nextId`生成
- `ASSIGN_UUID`：分配`UUID`，使用接口`IdentifierGenerator`中的方法`nextUUID`生成

`IdentifireGenerator`
```java
public interface IdentifierGenerator {
	//根据id是否为null判断是否需要主动分配Id
	default boolean assignId(object idvalue) {
		return StringUtils.checkValNull(idvalue);
	}
	
	//生成数值型Id
	Number nextId(object entity);
	
	//生成字符型uuid
	default String nextuuID(object entity) {
		return Idworker.get32UUID();
	}
```

`IKeyGenerator`
```java
public interface IKeyGenerator {
	//执行sql生成id
	 String executesql(String incrementerName);
	//获取数据库类型
	 DbType dbType();
 }
```

## 12. 逻辑删除
1. 设置全局配置
```yaml
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: deleted # 全局逻辑删除字段名
      logic-delete-value: 1 # 逻辑已删除值
      logic-not-delete-value: 0 # 逻辑未删除值
```
2. 标志软删字段
```java
import com.baomidou.mybatisplus.annotation.TableLogic;

public class User {
    // 其他字段...

    @TableLogic
    private Integer deleted;
}
```

## 13. SQL注入器
提供了灵活的机制来注入自定义的`SQL`方法，这通过`sqlInjector`全局配置实现。通过实现`ISqlInjector`接口或继承`AbstractSqlInjector`抽象类，可以注入自定义的通用方法到 MyBatis 容器中

1. 定义`SQL`：
```java
public class MysqlInsertAllBatch extends AbstractMethod {

    /**
     * @since 3.5.0
     */
    public MysqlInsertAllBatch() {
        super("mysqlInsertAllBatch");
    }

    @Override
    public MappedStatement injectMappedStatement(Class<?> mapperClass, Class<?> modelClass, TableInfo tableInfo) {
     // 定义SQL语句 (主键+普通字段)  insert into table(....) values(....),(....)
        String sql = "INSERT INTO " + tableInfo.getTableName() + "(" + tableInfo.getKeyColumn() + "," +
                tableInfo.getFieldList().stream().map(TableFieldInfo::getColumn).collect(Collectors.joining(",")) + ") VALUES ";
        String value = "(" + "#{" + ENTITY + DOT + tableInfo.getKeyProperty() + "}" + ","
                + tableInfo.getFieldList().stream().map(tableFieldInfo -> "#{" + ENTITY + DOT + tableFieldInfo.getProperty() + "}")
                .collect(Collectors.joining(",")) + ")";
        String valuesScript = SqlScriptUtils.convertForeach(value, "list", null, ENTITY, COMMA);
        SqlSource sqlSource = super.createSqlSource(configuration, "<script>" + sql + valuesScript + "</script>", modelClass);
        KeyGenerator keyGenerator = tableInfo.getIdType() == IdType.AUTO ? Jdbc3KeyGenerator.INSTANCE : NoKeyGenerator.INSTANCE;
        // 第三个参数必须和baseMapper的自定义方法名一致
        return this.addInsertMappedStatement(mapperClass, modelClass, this.methodName, sqlSource, keyGenerator,tableInfo.getKeyProperty(), tableInfo.getKeyColumn());
    }
}
```
2. 注册自定义方法
```java
public class MyLogicSqlInjector extends DefaultSqlInjector {

    @Override
    public List<AbstractMethod> getMethodList(Class<?> mapperClass) {
        List<AbstractMethod> methodList = super.getMethodList(mapperClass);
        methodList.add(new DeleteAll());
        methodList.add(new MyInsertAll());
        methodList.add(new MysqlInsertAllBatch());
        return methodList;
    }
}
```
3. 定义`BaseMapper`
```java
public interface MyBaseMapper<T> extends BaseMapper<T> {

    Integer deleteAll();

    int myInsertAll(T entity);

    int mysqlInsertAllBatch(@Param("list") List<T> batchList);
}
```
4. 配置`SqlInjector`
```yaml
mybatis-plus:
  global-config:
    sql-injector: com.example.MyLogicSqlInjector
```

## 14. 字段类型处理器
类型处理器（TypeHandler）扮演着 JavaType 与 JdbcType 之间转换的桥梁角色。它们用于在执行 SQL 语句时，将 Java 对象的值设置到 PreparedStatement 中，或者从 ResultSet 或 CallableStatement 中取出值

1. 自定义类型处理器
```java
import com.baomidou.mybatisplus.extension.handlers.JacksonTypeHandler;
import org.apache.ibatis.type.JdbcType;
import org.apache.ibatis.type.MappedJdbcTypes;
import org.apache.ibatis.type.MappedTypes;
import org.postgresql.util.PGobject;

import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

@MappedTypes({Object.class})
@MappedJdbcTypes(JdbcType.VARCHAR)
public class JsonbTypeHandler<T> extends JacksonTypeHandler<T> {

    private final Class<T> clazz;

    public JsonbTypeHandler(Class<T> clazz) {
        if (clazz == null) {
            throw new IllegalArgumentException("Type argument cannot be null");
        }
        this.clazz = clazz;
    }

    // 自3.5.6版本开始支持泛型,需要加上此构造.
    public JsonbTypeHandler(Class<?> type, Field field) {
        super(type, field);
    }

    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException {
        PGobject jsonbObject = new PGobject();
        jsonbObject.setType("jsonb");
        jsonObject.setValue(toJson(parameter));
        ps.setObject(i, jsonbObject);
    }
}
```
2. 使用自定义类型处理器
```java
@Data
@Accessors(chain = true)
@TableName(autoResultMap = true)
public class User {
    private Long id;

    ...

    /**
     * 使用自定义的 JSONB 类型处理器
     */
    @TableField(typeHandler = JsonbTypeHandler.class)
    private OtherInfo otherInfo;
}
```

## 15. 插件
### 1. 分页插件
```java
@Configuration
@MapperScan("scan.your.mapper.package")
public class MybatisPlusConfig {

    /**
     * 添加分页插件
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL)); // 如果配置多个插件, 切记分页最后添加
        // 如果有多数据源可以不配具体类型, 否则都建议配上具体的 DbType
        return interceptor;
    }
}

IPage<UserVo> selectPageVo(IPage<?> page, Integer state);
// 或者自定义分页类
MyPage selectPageVo(MyPage page);
// 或者返回 List
List<UserVo> selectPageVo(IPage<UserVo> page, Integer state);
```
### 2. 乐观锁插件
```java
@Configuration
@MapperScan("按需修改")
public class MybatisPlusConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return interceptor;
    }
}

import com.baomidou.mybatisplus.annotation.Version;

public class YourEntity {
    @Version
    private Integer version;
    // 其他字段...
}
```
### 3. 多租户插件
```java
public interface TenantLineHandler {

    /**
     * 获取租户 ID 值表达式，只支持单个 ID 值
     *
     * @return 租户 ID 值表达式
     */
    Expression getTenantId();

    /**
     * 获取租户字段名
     * 默认字段名叫: tenant_id
     *
     * @return 租户字段名
     */
    default String getTenantIdColumn() {
        return "tenant_id";
    }

    /**
     * 根据表名判断是否忽略拼接多租户条件
     * 默认都要进行解析并拼接多租户条件
     *
     * @param tableName 表名
     * @return 是否忽略, true:表示忽略，false:需要解析并拼接多租户条件
     */
    default boolean ignoreTable(String tableName) {
        return false;
    }

    /**
     * 忽略插入租户字段逻辑
     *
     * @param columns        插入字段
     * @param tenantIdColumn 租户 ID 字段
     * @return
     */
    default boolean ignoreInsert(List<Column> columns, String tenantIdColumn) {
        return columns.stream().map(Column::getColumnName).anyMatch(i -> i.equalsIgnoreCase(tenantIdColumn));
    }
}

@Component
public class CustomTenantHandler implements TenantLineHandler {

    @Override
    public Expression getTenantId() {
        // 假设有一个租户上下文，能够从中获取当前用户的租户
         Long tenantId = TenantContextHolder.getCurrentTenantId();
        // 返回租户ID的表达式，LongValue 是 JSQLParser 中表示 bigint 类型的 class
        return new LongValue(tenantId);;
    }

    @Override
    public String getTenantIdColumn() {
        return "tenant_id";
    }

    @Override
    public boolean ignoreTable(String tableName) {
        // 根据需要返回是否忽略该表
        return false;
    }

}

@Configuration
@MapperScan("com.yourpackage.mapper")
public class MybatisPlusConfig {

    @Autowired
    private CustomTenantHandler customTenantHandler;

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        TenantLineInnerInterceptor tenantInterceptor = new TenantLineInnerInterceptor();
        tenantInterceptor.setTenantLineHandler(customTenantHandler);
        interceptor.addInnerInterceptor(tenantInterceptor);
        return interceptor;
    }
}
```

注意：默认插入 SQL 是需要判断租户条件，因此需要配合自动填充字段功能填充租户字段，否则租户字段不会自动保存到数据库
### 4. 动态表名插件
```java
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.DynamicTableNameInnerInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MybatisPlusConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        DynamicTableNameInnerInterceptor dynamicTableNameInnerInterceptor = new DynamicTableNameInnerInterceptor();
        dynamicTableNameInnerInterceptor.setTableNameHandler((sql, tableName) -> {
            // 获取参数方法
            Map<String, Object> paramMap = RequestDataHelper.getRequestData();
            paramMap.forEach((k, v) -> System.err.println(k + "----" + v));

            String year = "_2018";
            int random = new Random().nextInt(10);
            if (random % 2 == 1) {
                year = "_2019";
            }
            return tableName + year;
        });
        interceptor.addInnerInterceptor(dynamicTableNameInnerInterceptor);
        return interceptor;
    }
}
```