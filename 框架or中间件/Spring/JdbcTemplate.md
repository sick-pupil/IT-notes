```java
@Configuration
@EnableAspectJAutoProxy
@EnableTransactionManagement
@ComponentScan(basePackages = {"..."})
//@PropertySource("classpath:jdbcTemplate.properties")
public class MainConfig {
	
	@Value("${jdbc.username}")
	private String username;
	
	@Value("${jdbc.password}")
	private String password;
	
	@Value("${jdbc.url}")
	private String url;
	
	@Value("${jdbc.driverClassName}")
	private String driverClassName;
	
	@Bean
	public DataSource dataSource() {
		DruidDataSource dataSource = new DruidDataSource();
		dataSource.setUsername(username);
		dataSource.setPassword(password);
		dataSource.setUrl(url);
		dataSource.setDriverClassName(driverClassName);
		return dataSource;
	}
	
	@Bean
	public JdbcTemplate jdbcTemplate(DataSource dataSource) {
		return new JdbcTemplate(dataSource);
	}
}
```

```java
jdbcTemplate.update(sql, args)
jdbcTemplate.batchUpdate(sql, batchArgs)
jdbcTemplate.queryForObject(sql, rowMapper, int)
jdbcTemplate.query(sql, rowMapper)
```