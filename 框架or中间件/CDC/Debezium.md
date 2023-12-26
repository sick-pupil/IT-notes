## 1. SpringBoot整合Embed Debezium
**application.yml**
```yml
task:  
  pool:  
    # 核心线程池大小  
    core-pool-size: 10  
    # 最大线程数  
    max-pool-size: 30  
    # 活跃时间  
    keep-alive-seconds: 60  
    # 队列容量  
    queue-capacity: 50  
    # 线程池线程名称前缀  
    prefix-name: common-llny-task-executor

debezium:  
  name: db_table_listener
  connector-class: io.debezium.connector.mysql.MySqlConnector
  # 记录同步的binlog偏移量
  offset-storage: org.apache.kafka.connect.storage.FileOffsetBackingStore
  # 记录同步偏移量的文件
  offset-storage-file-filename: /data/debezium/offsets.dat
  # 偏移量每次记录的刷新间隔
  offset-flush-interval-ms: 1000
  # 是否同步记录库表的结构变化
  include-schema-changes: false
  # 需要同步记录的库，多个值则以逗号为分隔符
  database-include-list: test_db
  # 需要同步记录的库表，多个值则以逗号为分隔符
  table-include-list: test_db.test_tb
  # 数据库服务ip
  host: 192.168.0.218
  # 数据库服务port
  port: 3308
  # 数据库服务登录用户
  username: root
  # 数据库服务登录用户密码
  password: 123456
  # 数据库服务连接设置时区
  server-time-zone: UTC
  database-allowPublicKeyRetrieval: true
  # 从机数据库服务实例serverId，不配置则在范围内随机设置server-id
  # database-server-id: 1000
  # 数据库服务名称
  database-server-name: mysql_instance
  # 数据库历史数据快照同步
  database-history: io.debezium.relational.history.FileDatabaseHistory
  database-history-file-filename: /data/debezium/history_offsets.dat
  database-history-store-only-captured-tables-ddl: true
  # 快照同步模式
  # initial: 连接器执行数据库的初始一致性快照，快照完成后，连接器开始为后续数据库更改流式传输事件记录
  # initial_only: 连接器只执行数据库的初始一致性快照，不允许捕获任何后续更改的事件
  # schema_only: 连接器只捕获所有相关表的表结构，不捕获初始数据，但是会同步后续数据库的更改记录
  # schema_only_revocery: 设置此选项可恢复丢失或损坏的数据库历史主题
  # when_needed: 当没有可用的偏移量时，或者当先前记录的偏移量指定了服务器中不可用的 binlog 位置或 GTID 时，运行快照
  # never: 连接器从不使用快照。首次使用逻辑服务器名称启动时，连接器从 binlog 的开头读取。谨慎配置此行为。只有当 binlog 保证包含数据库的全部历史时才有效
  snapshot-mode: schema_only
```

**自定义全局线程池**
```java
@Slf4j  
@Configuration  
public class TaskExecutorConfig {  
  
    @Value("${task.pool.core-pool-size}")  
    private Integer corePoolSize;  
  
    @Value("${task.pool.max-pool-size}")  
    private Integer maxPoolSize;  
  
    @Value("${task.pool.keep-alive-seconds}")  
    private Integer keepAliveSeconds;  
  
    @Value("${task.pool.queue-capacity}")  
    private Integer queueCapacity;  
  
    @Value("${task.pool.prefix-name}")  
    private String prefixName;  
  
    @Bean(name = "commonTaskExecutor")  
    public Executor taskExecutor() {  
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();  
        executor.setCorePoolSize(this.corePoolSize);  
        executor.setMaxPoolSize(this.maxPoolSize);  
        executor.setKeepAliveSeconds(this.keepAliveSeconds);  
        executor.setQueueCapacity(queueCapacity);  
        executor.setThreadNamePrefix(this.prefixName);  
        executor.setRejectedExecutionHandler(new TaskExecutorRejectHandler());  
        executor.initialize();  
        return executor;  
    }  
  
    private class TaskExecutorRejectHandler implements RejectedExecutionHandler {  
  
        @Override  
        public void rejectedExecution(Runnable runnable, ThreadPoolExecutor threadPoolExecutor) {  
            log.error("框架自定义线程池中线程被拒绝");  
            throw new RejectedExecutionException("Task " + runnable.toString() + " rejected from " + threadPoolExecutor.toString());  
        }  
    }  
}
```

**debezium自定义监听器**
```java
@DependsOn("commonTaskExecutor")  
@Slf4j  
@Component  
public class DebeziumListenerConfig {  
  
    @Value("${debezium.table-include-list}")  
    private String debeziumTableIncludeList;  
  
    private final static String SOURCE = "source";  
  
    private final static String AFTER = "after";  
  
    private final static String BEFORE = "before";  
  
    private final static String META_TABLE_KEY = "table";  
  
    private final static String META_DB_KEY = "db";  
  
    private final static String SQL_OPERATION_KEY = "op";  
  
    private final static String SQL_UPDATE_OPERATION = "u";  
  
    private final static String SQL_INSERT_OPERATION = "c";  
  
    private final static String SQL_DELETE_OPERATION = "d";  
  
    private final static String STRING_TYPE = "string";  
  
    private final static String DECIMAL_TYPE = "decimal";  
  
    private final static String DATE_TYPE = "date";  
  
    @Resource  
    private DebeziumSyncConfigMapper debeziumSyncConfigMapper;  
  
    @Resource  
    private DebeziumSyncTableClassConfigMapper debeziumSyncTableClassConfigMapper;  
  
    @Autowired  
    @Qualifier(value = "commonTaskExecutor")  
    private ThreadPoolTaskExecutor taskExecutor;  
  
    private final DebeziumEngine<RecordChangeEvent<SourceRecord>> debeziumEngine;  
  
    public DebeziumListenerConfig(Configuration customerConnectorConfiguration) {  
        this.debeziumEngine = DebeziumEngine.create(ChangeEventFormat.of(Connect.class))  
                .using(customerConnectorConfiguration.asProperties())  
                .notifying(this::handleChangeEvent)  
                .build();  
    }  
  
    private void handleChangeEvent(List<RecordChangeEvent<SourceRecord>> recordChangeEvents,  
                                   DebeziumEngine.RecordCommitter<RecordChangeEvent<SourceRecord>> recordCommitter) {  
        Map<String, Map<String, List<Object>>> changeDataByDbTb = new HashMap<>();  
  
        //获取配置  
        List<DebeziumSyncConfigDto> dbAndTbList = new ArrayList<>();  
        for(String dbAndTb : debeziumTableIncludeList.split(",")) {  
            String[] dbAndTbArr = dbAndTb.split("\\.");  
            dbAndTbList.add(new DebeziumSyncConfigDto(dbAndTbArr[0], dbAndTbArr[1]));  
        }  
        List<DebeziumSyncConfig> syncConfigs = debeziumSyncConfigMapper.getConfigsByDbAndTb(dbAndTbList);  
        Map<String, DebeziumSyncConfig> syncConfigByDbTb = syncConfigs.stream().collect(Collectors.toMap(o -> o.getDbName() + "-" + o.getTbName(), o -> o));  
  
        LambdaQueryWrapper<DebeziumSyncTableClassConfig> queryWrapper = new LambdaQueryWrapper<>();  
        queryWrapper.in(DebeziumSyncTableClassConfig::getSyncConfigId, syncConfigs.stream().map(DebeziumSyncConfig::getId).collect(Collectors.toList()));  
        Map<Long, List<DebeziumSyncTableClassConfig>> tableFieldMapBySyncId = debeziumSyncTableClassConfigMapper.selectList(queryWrapper).stream().collect(Collectors.groupingBy(DebeziumSyncTableClassConfig::getSyncConfigId));  
        Map<Long, Map<String, DebeziumSyncTableClassConfig>> fieldNameBySyncConfigId = tableFieldMapBySyncId.keySet().stream()  
                .collect(Collectors.toMap(syncConfigId -> syncConfigId,  
                                            syncConfigId -> tableFieldMapBySyncId.get(syncConfigId).stream()  
                                                            .collect(Collectors.toMap(fieldConfig -> fieldConfig.getTableFieldName(),  
                                                                                        fieldConfig -> fieldConfig))));  
  
        //变化记录  
        for (RecordChangeEvent<SourceRecord> r : recordChangeEvents) {  
            Struct sourceRecordValue = (Struct) r.record().value();  
            //变化元信息，变化前后  
            if(Objects.nonNull(sourceRecordValue)) {  
                Struct sourceStruct = (Struct) sourceRecordValue.get(SOURCE), beforeStruct = null, afterStruct = null;  
                Map<String, Object> sourceStructMap = null, beforeStructMap = null, afterStructMap = null;  
                String operation = String.valueOf(sourceRecordValue.get(SQL_OPERATION_KEY));  
                //插入操作  
                if(operation.equals(SQL_INSERT_OPERATION)) {  
                    afterStruct = (Struct) sourceRecordValue.get(AFTER);  
                }  
                //更新操作  
                else if(operation.equals(SQL_UPDATE_OPERATION)) {  
                    beforeStruct = (Struct) sourceRecordValue.get(BEFORE);  
                    afterStruct = (Struct) sourceRecordValue.get(AFTER);  
                }  
                //删除操作  
                else if(operation.equals(SQL_DELETE_OPERATION)) {  
                    beforeStruct = (Struct) sourceRecordValue.get(BEFORE);  
                }  
                //插入、更新、删除以外的操作  
                else {  
                    continue;  
                }  
                sourceStructMap = sourceStruct.schema().fields().stream()  
                        .map(Field::name)  
                        .filter(fieldName -> Objects.nonNull(sourceStruct.get(fieldName)))  
                        .collect(Collectors.toMap(fieldName -> fieldName, fieldName -> sourceStruct.get(fieldName)));  
  
                if(Objects.nonNull(beforeStruct)) {  
                    Struct beforeTmpStruct = beforeStruct;  
                    beforeStructMap = beforeStruct.schema().fields().stream()  
                            .map(Field::name)  
                            .filter(fieldName -> Objects.nonNull(beforeTmpStruct.get(fieldName)))  
                            .collect(Collectors.toMap(fieldName -> fieldName, fieldName -> beforeTmpStruct.get(fieldName)));  
                }  
  
                if(Objects.nonNull(afterStruct)) {  
                    Struct afterTmpStruct = afterStruct;  
                    afterStructMap = afterStruct.schema().fields().stream()  
                            .map(Field::name)  
                            .filter(fieldName -> Objects.nonNull(afterTmpStruct.get(fieldName)))  
                            .collect(Collectors.toMap(fieldName -> fieldName, fieldName -> afterTmpStruct.get(fieldName)));  
                }  
  
                String dbTbKey = sourceStructMap.get(META_DB_KEY) + "-" + sourceStructMap.get(META_TABLE_KEY);  
                //获取对应库表配置  
                DebeziumSyncConfig debeziumSyncConfig = syncConfigByDbTb.get(dbTbKey);  
                //获取对应表字段配置  
                Map<String, DebeziumSyncTableClassConfig> tableSyncField = fieldNameBySyncConfigId.get(debeziumSyncConfig.getId());  
  
                //生成对应对象  
                Object beforeTargetObj = MapUtils.isNotEmpty(beforeStructMap) ? transferToObj(beforeStructMap, debeziumSyncConfig, tableSyncField) : null;  
                Object afterTargetObj = MapUtils.isNotEmpty(afterStructMap) ? transferToObj(afterStructMap, debeziumSyncConfig, tableSyncField) : null;  
                saveIntoList(operation, dbTbKey, beforeTargetObj, afterTargetObj, changeDataByDbTb);  
            }  
        }  
        try {  
            recordCommitter.markBatchFinished();  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
        executeHandler(changeDataByDbTb, syncConfigByDbTb);  
    }  
  
    /**  
     * 将原始变化数据afterStructMap通过配置转换为实体类  
     * @param recordRow  
     * @param syncConfig  
     * @param tableSyncFieldConfig  
     * @return  
     */  
    private Object transferToObj(Map<String, Object> recordRow, DebeziumSyncConfig syncConfig, Map<String, DebeziumSyncTableClassConfig> tableSyncFieldConfig) {  
        Object result = null;  
        try {  
            //生成实际对象  
            Class<?> clazz = Class.forName(syncConfig.getModelClass());  
            result = ConstructorUtils.invokeConstructor(clazz);  
            for(String fieldName : tableSyncFieldConfig.keySet()) {  
                //字段类型转换  
                if(tableSyncFieldConfig.get(fieldName).getFieldType().equals(STRING_TYPE)) {  
                    FieldUtils.writeField(result, tableSyncFieldConfig.get(fieldName).getClassFieldName(), Objects.nonNull(recordRow.get(fieldName)) ? String.valueOf(recordRow.get(fieldName)) : null, true);  
                } else if(tableSyncFieldConfig.get(fieldName).getFieldType().equals(DECIMAL_TYPE)) {  
                    FieldUtils.writeField(result, tableSyncFieldConfig.get(fieldName).getClassFieldName(), Objects.nonNull(recordRow.get(fieldName)) ? NumberUtils.createBigDecimal(String.valueOf(recordRow.get(fieldName))) : null, true);  
                } else if(tableSyncFieldConfig.get(fieldName).getFieldType().equals(DATE_TYPE)) {  
                    FieldUtils.writeField(result, tableSyncFieldConfig.get(fieldName).getClassFieldName(), Objects.nonNull(recordRow.get(fieldName)) ? new Date(Long.valueOf(String.valueOf(recordRow.get(fieldName)))) : null, true);  
                }  
            }  
        } catch (Exception ex) {  
            ex.printStackTrace();  
        }  
        return result;  
    }  
  
    /**  
     * 根据SQL类型将改变数据存入  
     * @param operation  
     * @param beforeTargetObj  
     * @param afterTargetObj  
     * @param changeDataByDbTb  
     */  
    private void saveIntoList(String operation, String dbTbKey, Object beforeTargetObj, Object afterTargetObj, Map<String, Map<String, List<Object>>> changeDataByDbTb) {  
        //将变化对象存入list  
        if(changeDataByDbTb.containsKey(dbTbKey)) {  
            Map<String, List<Object>> changeDataByOperation = changeDataByDbTb.get(dbTbKey);  
            //插入操作  
            if(operation.equals(SQL_INSERT_OPERATION)) {  
                if(changeDataByOperation.containsKey(SQL_INSERT_OPERATION)) {  
                    changeDataByOperation.get(SQL_INSERT_OPERATION).add(afterTargetObj);  
                } else {  
                    List<Object> insertObjList = new ArrayList<>();  
                    insertObjList.add(afterTargetObj);  
                    changeDataByOperation.put(SQL_INSERT_OPERATION, insertObjList);  
                }  
            }  
            //更新操作  
            else if(operation.equals(SQL_UPDATE_OPERATION)) {  
                if(changeDataByOperation.containsKey(SQL_UPDATE_OPERATION)) {  
                    changeDataByOperation.get(SQL_UPDATE_OPERATION).add(afterTargetObj);  
                } else {  
                    List<Object> updateObjList = new ArrayList<>();  
                    updateObjList.add(afterTargetObj);  
                    changeDataByOperation.put(SQL_UPDATE_OPERATION, updateObjList);  
                }  
            }  
            //删除操作  
            else if(operation.equals(SQL_DELETE_OPERATION)) {  
                if(changeDataByOperation.containsKey(SQL_DELETE_OPERATION)) {  
                    changeDataByOperation.get(SQL_DELETE_OPERATION).add(beforeTargetObj);  
                } else {  
                    List<Object> deleteObjList = new ArrayList<>();  
                    deleteObjList.add(beforeTargetObj);  
                    changeDataByOperation.put(SQL_DELETE_OPERATION, deleteObjList);  
                }  
            }  
        } else {  
            Map<String, List<Object>> changeDataByOperation = new HashMap<>();  
            changeDataByDbTb.put(dbTbKey, changeDataByOperation);  
            List<Object> operationObjList = new ArrayList<>();  
            if(operation.equals(SQL_INSERT_OPERATION) || operation.equals(SQL_UPDATE_OPERATION)) {  
                operationObjList.add(afterTargetObj);  
            } else if(operation.equals(SQL_DELETE_OPERATION)) {  
                operationObjList.add(beforeTargetObj);  
            }  
            changeDataByOperation.put(operation, operationObjList);  
        }  
    }  
  
    /**  
     * 执行处理器  
     * @param changeDataByDbTb  
     * @param syncConfigByDbTb  
     */  
    private void executeHandler(Map<String, Map<String, List<Object>>> changeDataByDbTb, Map<String, DebeziumSyncConfig> syncConfigByDbTb) {  
        try {  
            for(String dbAndTbKey : changeDataByDbTb.keySet()) {  
                DebeziumSyncConfig debeziumSyncConfig = syncConfigByDbTb.get(dbAndTbKey);  
                DebeziumHandler handler = (DebeziumHandler) SpringContextHolder.getBean(debeziumSyncConfig.getHandlerBeanName());  
                handler.init();  
                handler.handle(changeDataByDbTb.get(dbAndTbKey));  
                handler.destory();  
            }  
        } catch (Exception ex) {  
            ex.printStackTrace();  
        }  
    }  
  
    @PostConstruct  
    private void start() {  
        this.taskExecutor.execute(debeziumEngine);  
    }  
  
    @PreDestroy  
    private void stop() throws IOException {  
        if (Objects.nonNull(this.debeziumEngine)) {  
            this.debeziumEngine.close();  
        }  
    }  
}
```

**中间处理器插件**
```java
public interface DebeziumHandler<T> {  
  
    String SQL_UPDATE_OPERATION = "u";  
  
    String SQL_INSERT_OPERATION = "c";  
  
    String SQL_DELETE_OPERATION = "d";  
  
    void init();  
  
    void handle(Map<String, List<T>> changeDataList);  
  
    void destory();  
}

@Component  
public class DianguanHandler implements DebeziumHandler<TcsDeviceToolDianguan> {  
  
    @Value("${zjTools.publishUrl}")
    private String zjToolsPublishUrl;  
  
    @Value("${zjTools.tokenName}")
    private String zjToolsTokenName;  
  
    @Value("${zjTools.timestampName}")
    private String zjToolsTimestampName;  
  
    @Value("${zjTools.publicRsaKey}")
    private String zjToolsPublicRsaKey;
  
    private static final String REQUEST_HEADER_TOKEN_NAME = "token";
  
    private static final String REQUEST_HEADER_TIMESTAMP_NAME = "timestamp";
  
    private static final String REQUEST_HEADER_CONTENT_TYPE_NAME = "content-type";
  
    private static final String ZJ_DEPT_CODE = "0308";
  
    @Autowired  
    private RedisUtils redisUtils;
  
    @Autowired  
    private ZjToolToSyncService zjToolToSyncService;
  
    @Override  
    public void init() {  
        //登录  
        Long currentTimeMillis = System.currentTimeMillis();
        //如果存在登录信息  
        if(redisUtils.hasKey(zjToolsTokenName) && redisUtils.hasKey(zjToolsTimestampName)) {  
            //验证是否登录超时  
            Long lastLoginTimeByMillis = Long.valueOf(redisUtils.get(zjToolsTimestampName).toString());
            LocalDateTime now = Instant.ofEpochMilli(currentTimeMillis).atZone(ZoneId.systemDefault()).toLocalDateTime();
            LocalDateTime lastLoginTime = Instant.ofEpochMilli(lastLoginTimeByMillis).atZone(ZoneId.systemDefault()).toLocalDateTime();
            Duration between = Duration.between(lastLoginTime, now);  
            //登录未超时，刷新剩余登录时间  
            if(between.toMinutes() <= 10) {  
                redisUtils.expire(zjToolsTokenName, 10, TimeUnit.MINUTES);  
                redisUtils.expire(zjToolsTimestampName, 10, TimeUnit.MINUTES);  
            }  
        }  
        //不存在登录信息或者已经登录超时  
        else {  
            //重新设置登录状态  
            try {  
                redisUtils.set(zjToolsTokenName, RsaUtils.encryptByPublicKey(zjToolsPublicRsaKey, String.valueOf(currentTimeMillis)), 10, TimeUnit.MINUTES);  
            } catch(Exception ex) {  
                ex.printStackTrace();  
            }  
            redisUtils.set(zjToolsTimestampName, currentTimeMillis.toString(), 10, TimeUnit.MINUTES);  
        }  
    }  
  
    @Override  
    public void handle(Map<String, List<TcsDeviceToolDianguan>> changeDataListByOperation) {  
        for(String operation : changeDataListByOperation.keySet()) {  
            List<TcsDeviceToolDianguan> changeDataList = changeDataListByOperation.get(operation);  
            List<TcsDeviceToolDianguan> changeData = changeDataList.stream()  
                    .filter(item -> Objects.nonNull(item.getBureauCode()) && item.getBureauCode().equals(ZJ_DEPT_CODE))  
                    .collect(Collectors.toList());  
            try {  
                //新增和更新同步  
                if(CollectionUtils.isNotEmpty(changeData) && (operation.equals(SQL_UPDATE_OPERATION) || operation.equals(SQL_INSERT_OPERATION))) {  
                    ToolsSyncDto syncResult = HttpClientUtils.post(zjToolsPublishUrl, changeData, ToolsSyncDto.class, createHeaders());  
                    //更新台账记录同步状态  
                    if (syncResult.getSuccess()) {  
                        List<String> syncIds = changeData.stream().map(TcsDeviceToolDianguan::getId).collect(Collectors.toList());  
                        List<ZjToolToSync> syncRecords = syncIds.stream().map(id -> {  
                            ZjToolToSync syncRecord = new ZjToolToSync();  
                            syncRecord.setSyncId(id);  
                            syncRecord.setSyncTime(String.valueOf(new Timestamp(System.currentTimeMillis())));  
                            return syncRecord;  
                        }).collect(Collectors.toList());  
                        zjToolToSyncService.saveOrUpdateBatch(syncRecords);  
                    }  
                }  
            } catch(Exception ex) {  
                ex.printStackTrace();  
            }  
        }  
    }  
  
    @Override  
    public void destory() {  
  
    }  
  
    private Header[] createHeaders() {  
        List<Header> requestHeaders = new ArrayList<>();  
        Header token = new Header() {  
            @Override  
            public boolean isSensitive() {  
                return false;  
            }  
  
            @Override  
            public String getName() {  
                return REQUEST_HEADER_TOKEN_NAME;  
            }  
  
            @Override  
            public String getValue() {  
                return redisUtils.get(zjToolsTokenName).toString();  
            }  
        };  
        Header timestamp = new Header() {  
            @Override  
            public boolean isSensitive() {  
                return false;  
            }  
  
            @Override  
            public String getName() {  
                return REQUEST_HEADER_TIMESTAMP_NAME;  
            }  
  
            @Override  
            public String getValue() {  
                return redisUtils.get(zjToolsTimestampName).toString();  
            }  
        };  
        Header contentType = new Header() {  
            @Override  
            public boolean isSensitive() {  
                return false;  
            }  
  
            @Override  
            public String getName() {  
                return REQUEST_HEADER_CONTENT_TYPE_NAME;  
            }  
  
            @Override  
            public String getValue() {  
                return "application/json";  
            }  
        };  
        requestHeaders.add(token);  
        requestHeaders.add(timestamp);  
        requestHeaders.add(contentType);  
        return requestHeaders.toArray(new Header[requestHeaders.size()]);  
    }  
}
```