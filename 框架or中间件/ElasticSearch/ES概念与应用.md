## 1. 背景
`ELK`技术栈：
- `ElasticSearch`：存储、分析
- `Logstach/Beats`：采集、写入数据
- `Kibana`：图形化`UI`的数据交互

`ElasticSearch`不使用列行存储，而是存储序列化的`JSON`文档；`ElasticSearch`使用**倒排索引**的数据结构，**倒排索引**列出任何文档中出现的唯一单词，并标识每个单词出现的所有文档
（倒排索引：通过`value`寻找`key`）
<img src="D:\Project\IT-notes\框架or中间件\ElasticSearch\img\倒排索引结构.png" style="width:700px;height:500px;" />

索引可被认作一种文档的优化集合，且每个文档都是字段的集合，字段是包含数据的键值对

默认情况下，`Elasticsearch`索引每个字段中的所有数据，且每个被索引的字段有一个专用的优化数据结构。例如，文本字段被存储在倒排索引中，数字和地理字段存储在`BKD树`中。使用每个字段的数据结构来聚集和返回搜索结果

当启用动态映射后，`Elasticsearch`自动检测和向索引中添加新的字段。这个默认行为使索引和浏览你的数据更容易——只需开始索引文档，`Elasticsearch`会自动检测和映射布尔值、浮点值和整数值、日期以及字符串到合适的`Elasticsearch`数据类型

也可以自定义映射
- 区分全文字符串字段和精确值字符串字段
- 执行特定语言的文本分析
- 为部分匹配优化字段
- 使用自定义的日期格式
- 使用无法自动检测的数据类型，如`geo_point`和 `geo_shape`

`ElasticSearch`提供`REST API`支持结构化查询（类似`SQL`携带条件查询若干字段）以及全文查询（查询匹配关键串的所有文档）；也提供聚合查询，如最大、平均、分组

`ElasticSearch`提供高可用的**多节点多分片主从分片副本集群**
## 2. 数据格式
<img src="D:\Project\IT-notes\框架or中间件\ElasticSearch\img\ES数据格式与MySQL类比.png" style="width:700px;height:180px;" />

在`ES`中，
- 索引`index`类似于`MySQL`中的数据库`database`
- 类型`type`类似于`MySQL`中的表`table`
- 文档`document`类似于`MySQL`中的行`row`
- 字段`field`类似于`MySQL`中的列`colume`

**Types已逐渐被淡化甚至遗弃，原因：**
1. 在关系型数据库中`table`是独立的（独立存储），但`es`中同一个`index`中不同`type`是存储在同一个索引中的（`lucene`的索引文件），因此不同`type`中相同名字的字段的定义（`field mapping`）必须一致
2. 不同类型记录存储在同一个`index`，影响`luncene`压缩性能
## 3. 数据类型
`ES`中的`mapping`类似于数据库中的表结构定义`schema`
- 定义索引中的字段的名称
- 定义字段的数据类型，比如字符串、数字、布尔
- 字段，倒排索引的相关配置，比如设置某个字段为不被索引、记录`position`等

创建索引时，可以对`mapping`是否动态而设置成`false true strict`

| | `true` | `false` | `strict` |
| ----- | ----- | ----- | ----- |
| 文档可索引 | Y | Y | N |
| 字段可索引 | Y | N | N |
| `mapping`可被更新 | Y | N | N |

- `keyword`：不进行切分的字符串类型，即**在索引时，对keyword类型的数据不进行切分，直接构建倒排索引；在搜索时，对该类型的查询字符串不进行切分后的部分匹配**
- `text`：可切分；**在索引时，可按照相应的切词算法对文本内容进行切分，然后构建倒排索引；在搜索时，对该类型的查询字符串按照用户的切词算法进行切分，然后对切分后的部分匹配打分**

`keyword`与`text`特例：既有`text`类型可以用于全文检索，又有`keyword`类型可以用于聚合分析
```
PUT 索引库名称 
{ 
	"mappings": { 
		"properties": { 
			"my_field": { 
				"type": "text", 
				"fields": { 
					"keyword": { 
						"type": "keyword" 
					} 
				} 
			} 
		} 
	} 
}
```
- `long`
- `integer`
- `short`
- `byte`
- `double`
- `float`
- `scaled_float`
- `half_float`
- `unsigned_long`
- `boolean`
- `date`
```

elasticsearch一般使用如下形式表示日期类型数据
	格式化的日期字符串，例如 2015-01-01 或 2015/01/01 12:10:30
	毫秒级的长整型(一个表示自纪元以来毫秒数的长整形数字)，表示从1970年1月1日0点到现在的毫秒数
	秒级别的整形(表示从纪元开始的秒数的整数)，表示从1970年1月1日0点到现在的秒数

在Elasticsearch内部，日期转换为UTC（如果指定了时区），并存储为毫秒数时间戳

日期类型的默认格式为strict_date_optional_time||epoch_millis。其中，strict_date_optional_time的含义是严格的时间类型，支持yyyy-MM-dd、yyyyMMdd、yyyyMMddHHmmss、yyyy-MM-ddTHH:mm:ss、yyyy-MM-ddTHH:mm:ss.SSS和yyyy-MM-ddTHH:mm:ss:SSSZ等格式；epoch_millis的含义是从1970年1月1日0点到现在的毫秒数

日期类型默认不支持yyyy-MM-dd HH:mm:ss格式,如果经常使用这种格式,可以在索引的mapping中设置日期字段的format属性为自定义格式

```
示例：
```
#一个酒店搜索项目,酒店的索引除了包含酒店名称、城市、价格、星级、评论数、是否满房之外，还需要定义日期等
PUT /hotel/_mapping
{ 
	"properties": { 
		"create_time": { 
			"type": "date" 
		}
	} 
}
#插入数据
POST /hotel/_doc/001
{
	"title":"文雅酒店", 
	"city":"上海", 
	"price":270, 
	"start":10, 
	"comment_count":2, 
	"full_room":true, 
	"create_time":"20210115"
}

#搜索创建日期为2015年的酒店
GET hotel/_search
{
	"query": { 
		"range": { 
			"create_time": { 
				"gte": "20150101", 
				"lt": "20220112" 
			}
		} 
	} 
}

#设置create_time字段的格式为yyyy-MM-dd HH:mm:ss
PUT /hotel
{
	"mappings": { 
		"properties": { 
			"title": { 
				"type": "text"
			}, 
			"city": { 
				"type": "keyword" 
			}, 
			"price": { 
				"type": "double" 
			}, 
			"start": { 
				"type": "byte" 
			}, 
			"comment_count": { 
				"type": "integer"
			}, 
			"full_room": { 
				"type": "boolean" 
			}, 
			"create_time": { 
				"type": "date", 
				"format": "yyyy-MM-dd HH:mm:ss" 
			} 
		} 
	} 
}
```
- 嵌套对象
- 数组
## 4. Restful操作
<img src="D:\Project\IT-notes\框架or中间件\ElasticSearch\img\ES检索类型.png" style="width:700px;height:800px;" />

### 1. 索引
#### 1. 创建
创建某索引：`PUT http://127.0.0.1:9200/#{indexName}`
示例：
```
#建立一个人名索引,设定姓名字段为keyword字段
PUT /user 
{ 
	"mappings": { 
		"properties": { 
			"user_name": { 
				"type": "keyword" 
			}
		}
	}
}
```
#### 2. 查询
查询某索引信息：`GET http://127.0.0.1:9200/#{indexName}`
列出所有索引的信息：`GET http://127.0.0.1:9200/_cat/indices?v`
#### 3. 删除
删除某索引：`DELETE http://127.0.0.1:9200/#{indexName}`
### 2. 文档
#### 1. 创建
创建添加文档：`POST http://127.0.0.1:9200/#{indexName}/_doc`，多次相同的请求会创建多个文档数据，`ID`随机，不能使用`PUT`方法
对应请求体：
```json
{
	"title": "123",
	"category": "小米",
	"images": "https://.....",
	"price": 3999.99
}
```

创建自定义`ID`文档：`POST/PUT http://127.0.0.1:9200/#{indexName}/_doc/#{id}`，返回结果存在自定义`ID`
创建自定义`ID`文档：`PUT http://127.0.0.1:9200/#{indexName}/_create/#{id}`，返回结果存在自定义`ID`
#### 2. 查询
根据主键查询文档数据：`GET http://127.0.0.1:9200/#{indexName}/_doc/#{id}`
全部查询，不带任何查询条件：`GET http://127.0.0.1:9200/#{indexName}/_search`

返回示例：
```json
{
    "took": 76,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 1,
            "relation": "eq"
        },
        "max_score": 1.0,
        "hits": [
            {
                "_index": "customer",
                "_type": "_doc",
                "_id": "1",
                "_score": 1.0,
                "_source": {
                    "name": "John Doe"
                }
            }
        ]
    }
}
```

- `took`：`Elasticsearch`执行查询的耗时（毫秒）
- `timed_out`： 搜索请求是否超时
- `_shards`： 多少分片被搜索，以及成功、失败或跳过的分片详情
- `max_score`：查找到的最相关的文档分数
- `hits.total.value`：查找到匹配文档数量
- `hits.sort`：文档的排序位置（不按相关分数排序时）
- `hits._score`：文档的相关分数（不适用于使用 `match_all`）
#### 3. 修改
完全覆盖修改：`PUT http://127.0.0.1:9200/#{indexName}/_doc/#{id}`，需要带修改请求体
局部覆盖修改：`PUT http://127.0.0.1:9200/#{indexName}/_update/#{id}`
```json
{
	"doc": {
		"title": "456"
	}
}
```
#### 4. 删除
删除某`ID`文档数据：`DELETE http://127.0.0.1:9200/#{indexName}/_doc/#{id}`
#### 5. 复杂查询
条件查询：
1. `GET http://127.0.0.1:9200/shopping/_search?q=categroy:小米`
2. `GET http://127.0.0.1:9200/shopping/_search`
请求体：
```json
{
	”query“ : {
		//按照条件查询，该查询存在分词查询
		"match" : {
			"categroy" : "小米"
		},
		//全词查询
		"match_phrase" : {
			"categroy" : "小米"
		},
		
		//不存在任何条件的全查询
		//”match_all“ : {
		//	
		//}
		
		//分页查询
		//"from": 0,
		//"size": 10
		
		//排序
		//"sort" : {
		//	"price" : {
		//		"order" : "desc"
		//	}
		//}
	}
}
```
3. `GET http://127.0.0.1:9200/shopping/_search`
请求体：
```json
{
	"query" : {
		"bool" : {
			//and
			"must" : [
				{
					"match" : {
						"category":"小米"
					}
				},
				{
					"match" : {
						"price":1999.00
					}
				}
			],
			//or
			"should" : [
				{
					"match" : {
						"category":"小米"
					}
				},
				{
					"match" : {
						"category":"华为"
					}
				}
			],
			//结果过滤
			"filter" : {
				"range" : {
					"price" : {
						"gt" : 5000
					}
				}
			}
		}
	}
}
```

聚合查询：
1. `GET http://127.0.0.1:9200/shopping/_search`
```json
{
	//聚合查询
	"aggs" : {
		//分组
		//聚合查询自定义名称
		"price_group" : {
			//分组
			"terms" : {
				"field" : "price" //分组字段
			}
		},
		
		//平均
		//聚合查询自定义名称
		"price_avg" : {
			//分组
			"avg" : {
				"field" : "price" //分组字段
			}
		},
	},
	//取原始数据条数
	"size" : 0
}
```

普通分组求和求平均与嵌套分组
```json
{  
	"size": 0,  
	"aggs": {  
		"group_by_state": {
			"terms": {
				"field": "state.keyword"
			}
		}
	}
}

{  
	"size": 0,  
	"aggs": {
		"group_by_state": {
			"terms": {
				"field": "state.keyword"
			},
			"aggs": {
				"average_balance": {
					"avg": {
						"field": "balance"
					}
				}
			}
		}
	}
}

{
	"size": 0,
	"aggs": {
		"group_by_state": {
			"terms": {
				"field": "state.keyword",
				"order": {
					"average_balance": "desc"
				}
			},
			"aggs": {
				"average_balance": {
					"avg": {
						"field": "balance"
					}
				}
			}
		}
	}
}
```
### 3. 映射
**做文档的数据约束，即定义每个字段的类型，是否可被索引**

创建索引：`PUT http://127.0.0.1:9200/#{indexName}`
创建映射：`PUT http://127.0.0.1:9200/#{indexName}/_mapping`
请求体：
```json
{
	"properties" : {
		"name" : {
			"type": "text", //类型，能分词索引
			"index": true //是否能索引
		},
		"sex" : {
			"type": "keyword", //类型，不能分词索引
			"index": true //是否能索引
		},
		"tel" : {
			"type": "keyword", //类型，不能分词索引
			"index": false //是否能索引
		},
	}
}
```

## 5. JavaAPI
### 1. 创建连接
```java
import org.apache.http.HttpHost;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;

public class ConnectElasticsearch {
    public static void connect(ElasticsearchTask task) {
        // 创建客户端对象，并自动关闭连接
        try (RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("localhost", 9200, "http")))) {
            task.doSomething(client);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
### 2. 索引创建
```java
import org.apache.http.HttpHost;
import org.elasticsearch.action.admin.indices.create.CreateIndexRequest;
import org.elasticsearch.action.admin.indices.create.CreateIndexResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;

import java.io.IOException;

public class CreateIndex {
    public static void main(String[] args) throws IOException {
        // 创建客户端对象
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("localhost", 9200, "http")));

        // 创建索引 - 请求对象
        CreateIndexRequest request = new CreateIndexRequest("user");
        // 发送请求，获取响应
        CreateIndexResponse response = client.indices()
                .create(request, RequestOptions.DEFAULT);
        boolean acknowledged = response.isAcknowledged();
        // 响应状态
        System.out.println("操作状态 = " + acknowledged);

        // 关闭客户端连接
        client.close();
    }
}
```
### 3. 索引查询
```java
public class SearchIndex {
    public static void main(String[] args) throws IOException {
        // 创建客户端对象
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("localhost", 9200, "http")));

        // 查询索引 - 请求对象
        GetIndexRequest request = new GetIndexRequest("user");
        // 发送请求，获取响应
        GetIndexResponse response = client.indices()
          .get(request, RequestOptions.DEFAULT);
        System.out.println("aliases:" + response.getAliases());
        System.out.println("mappings:" + response.getMappings());
        System.out.println("settings:" + response.getSettings());

        client.close();
    }
}
```
### 4. 索引删除
```java
public class DeleteIndex {
    public static void main(String[] args) throws IOException {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(new HttpHost("localhost", 9200, "http")));
        // 删除索引 - 请求对象
        DeleteIndexRequest request = new DeleteIndexRequest("user");
        // 发送请求，获取响应
        AcknowledgedResponse response = client.indices()
          .delete(request, RequestOptions.DEFAULT);
        // 操作结果
        System.out.println("操作结果：" + response.isAcknowledged());
        client.close();
    }
}
```
### 5. 文档新增
```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.yu.es.test.model.User;
import com.yu.es.test.utils.ConnectElasticsearch;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.action.index.IndexResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.common.xcontent.XContentType;

public class InsertDoc {
    public static void main(String[] args) {
        ConnectElasticsearch.connect(client -> {
            // 新增文档 - 请求对象
            IndexRequest request = new IndexRequest();
            // 设置索引及唯一性标识
            request.index("user").id("1001");

            // 创建数据对象
            User user = new User();
            user.setName("zhangsan");
            user.setAge(30);
            user.setSex("男");

            // Model -> JSON
            ObjectMapper objectMapper = new ObjectMapper();
            String productJson = objectMapper.writeValueAsString(user);
            // 添加文档数据，数据格式为 JSON 格式
            request.source(productJson, XContentType.JSON);
            // 客户端发送请求，获取响应对象
            IndexResponse response = client.index(request, RequestOptions.DEFAULT);

            // 打印结果信息
            System.out.println("_index:" + response.getIndex());
            System.out.println("_id:" + response.getId());
            System.out.println("_result:" + response.getResult());
        });
    }
}
```
### 6. 文档修改
```java
import com.yu.es.test.utils.ConnectElasticsearch;
import org.elasticsearch.action.update.UpdateRequest;
import org.elasticsearch.action.update.UpdateResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.common.xcontent.XContentType;

public class UpdateDoc {
    public static void main(String[] args) {
        ConnectElasticsearch.connect(client -> {
            // 修改文档 - 请求对象
            UpdateRequest request = new UpdateRequest();
            // 配置修改参数
            request.index("user").id("1001");
            // 设置请求体，对数据进行修改
            request.doc(XContentType.JSON, "sex", "女");
            // 客户端发送请求，获取响应对象
            UpdateResponse response = client.update(request, RequestOptions.DEFAULT);
            System.out.println("_index:" + response.getIndex());
            System.out.println("_id:" + response.getId());
            System.out.println("_result:" + response.getResult());
        });
    }
}
```
### 7. 文档查询
```java
import com.yu.es.test.utils.ConnectElasticsearch;
import org.elasticsearch.action.get.GetRequest;
import org.elasticsearch.action.get.GetResponse;
import org.elasticsearch.client.RequestOptions;

public class GetDoc {
    public static void main(String[] args) {
        ConnectElasticsearch.connect(client -> {
            // 1.创建请求对象
            GetRequest request = new GetRequest().index("user").id("1001");
            // 2.客户端发送请求，获取响应对象
            GetResponse response = client.get(request, RequestOptions.DEFAULT);
            // 3.打印结果信息
            System.out.println("_index:" + response.getIndex());
            System.out.println("_type:" + response.getType());
            System.out.println("_id:" + response.getId());
            System.out.println("source:" + response.getSourceAsString());
        });
    }
}
```
### 8. 文档删除
```java
import com.yu.es.test.utils.ConnectElasticsearch;
import org.elasticsearch.action.delete.DeleteRequest;
import org.elasticsearch.action.delete.DeleteResponse;
import org.elasticsearch.client.RequestOptions;

public class DeleteDoc {
    public static void main(String[] args) {
        ConnectElasticsearch.connect(client -> {
            // 1.创建请求对象
            DeleteRequest request = new DeleteRequest().index("user").id("1001");
            // 2.客户端发送请求，获取响应对象
            DeleteResponse response = client.delete(request, RequestOptions.DEFAULT);
            // 3.打印信息
            System.out.println(response.toString());
        });
    }
}
```
### 9. 文档批量新增
```java
import com.yu.es.test.utils.ConnectElasticsearch;
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.action.bulk.BulkResponse;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.common.xcontent.XContentType;

public class BatchInsertDoc {
    public static void main(String[] args) {
        ConnectElasticsearch.connect(client -> {
            // 创建批量新增请求对象
            BulkRequest request = new BulkRequest();
            request.add(new IndexRequest().index("user").id("1001")
                    .source(XContentType.JSON, "name", "zhangsan", "age", 30));
            request.add(new IndexRequest().index("user").id("1002")
                    .source(XContentType.JSON, "name", "lisi"));
            request.add(new IndexRequest().index("user").id("1003")
                    .source(XContentType.JSON, "name", "wangwu"));
            // 客户端发送请求，获取响应对象
            BulkResponse responses = client.bulk(request, RequestOptions.DEFAULT);
            // 打印结果信息
            System.out.println("took:" + responses.getTook());
            System.out.println("items:" + responses.getItems());
        });
    }
}
```
### 10. 文档批量删除
```java
import com.yu.es.test.utils.ConnectElasticsearch;
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.action.bulk.BulkResponse;
import org.elasticsearch.action.delete.DeleteRequest;
import org.elasticsearch.client.RequestOptions;

public class BatchDelDoc {
    public static void main(String[] args) {
        ConnectElasticsearch.connect(client -> {
            // 创建批量删除请求对象
            BulkRequest request = new BulkRequest();
            request.add(new DeleteRequest().index("user").id("1001"));
            request.add(new DeleteRequest().index("user").id("1002"));
            request.add(new DeleteRequest().index("user").id("1003"));
            // 客户端发送请求，获取响应对象
            BulkResponse responses = client.bulk(request, RequestOptions.DEFAULT);
            // 打印结果信息
            System.out.println("took:" + responses.getTook());
            System.out.println("items:" + responses.getItems());
        });
    }
}
```
### 11. 查询
#### 1. 查询大致代码块
```java
// 创建搜索请求对象
SearchRequest request = new SearchRequest();
request.indices("user");

// 构建查询的请求体
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

// 构建....
// 这一块的代码需要根据具体需求来写

// 输出查询结果
SearchHits hits = response.getHits();
System.out.println("took:" + response.getTook());
System.out.println("timeout:" + response.isTimedOut()); // 是否超时
System.out.println("total:" + hits.getTotalHits()); // 查询到数据总数
System.out.println("MaxScore:" + hits.getMaxScore());
System.out.println("hits========>>");
// 输出每条查询的结果信息
for (SearchHit hit : hits)
		System.out.println(hit.getSourceAsString());
System.out.println("<<========");
```
#### 2. 全量查询
```java
// 查询全部
sourceBuilder.query(QueryBuilders.matchAllQuery());
```
#### 3. 条件查询
```java
// 查询条件：age = 30
sourceBuilder.query(QueryBuilders.termQuery("age", "30"));
```
#### 4. 分页查询
```java
// 当前页起始索引(第一条数据的顺序号) from
sourceBuilder.from(0);
// 每页显示多少条 size
sourceBuilder.size(2);
```
#### 5. 排序查询
```java
// 查询全部
sourceBuilder.query(QueryBuilders.matchAllQuery());
// 年龄升序
sourceBuilder.sort("age", SortOrder.ASC);
```
#### 6. 组合查询
```java
// 组合查询
BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
// 必须包含: age = 30
boolQueryBuilder.must(QueryBuilders.matchQuery("age", "30"));
// 一定不含：name = zhangsan
boolQueryBuilder.mustNot(QueryBuilders.matchQuery("name", "zhangsan"));
// 可能包含: sex = 男
boolQueryBuilder.should(QueryBuilders.matchQuery("sex", "男"));
sourceBuilder.query(boolQueryBuilder);
```
#### 7. 范围查询
```java
// 范围查询
RangeQueryBuilder rangeQuery = QueryBuilders.rangeQuery("age");
// rangeQuery.gte("30"); // age 大于等于 30
rangeQuery.lte("40"); // age 小于等于 40
sourceBuilder.query(rangeQuery);
```
#### 8. 模糊查询
```java
// 模糊查询: name 包含 wangwu 的数据
sourceBuilder.query(QueryBuilders.fuzzyQuery("name","wangwu").fuzziness(Fuzziness.ONE)));
```
#### 9. 最大值查询
```java
// 最大值查询
SearchRequest request = new SearchRequest().indices("user");
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.aggregation(AggregationBuilders.max("maxAge").field("age"));
request.source(sourceBuilder);
// 客户端发送请求，获取响应对象
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
System.out.println(response);
```
#### 10. 分组查询
```java
// 分组查询
SearchRequest request = new SearchRequest().indices("user");
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
sourceBuilder.aggregation(AggregationBuilders.terms("age_groupby").field("age"));
request.source(sourceBuilder);
// 客户端发送请求，获取响应对象
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
System.out.println(response);
```

## 6. 核心概念
- 索引：一个拥有相似特征的文档的集合。一个索引由一个名字表示，名称必须为全部小写字母，当对索引中的文档进行搜索、更新、删除，都会使用索引。必须索引，提高查询速度
- 类型：在一个索引中可以定义多种类型，可以理解为索引中的分类分区，默认类型`_doc`
- 文档：文档是可被索引的基础信息单元，即为一条数据
- 字段：相当数据库表的字段，对文档数据根据不同属性进行分类标识
- 映射：处理数据的方式和规则，如字段的数据类型、默认值、分析器、是否能被索引
- 分片：为了一个索引可以存储超出单节点硬件限制的大量数据，把一个索引划分成多份，即分片。创建索引可以指定分片数量。分片允许水平扩容、允许进行分布式的并行操作、提高吞吐量
- 副本：分片存在副本，允许分片节点的故障转移机制
- 分配：即将分片分配给节点的过程，存在分配分片主从副本
## 7. 系统架构
<img src="D:\Project\IT-notes\框架or中间件\ElasticSearch\img\ES系统架构.png" style="width:700px;height:400px;" />

## 8. 集群
**docker-compose部署**：
```yml
version: '2.2'
services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.11.2
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data01:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elastic
  es02:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.11.2
    container_name: es02
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data02:/usr/share/elasticsearch/data
    networks:
      - elastic
  es03:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.11.2
    container_name: es03
    environment:
      - node.name=es03
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data03:/usr/share/elasticsearch/data
    networks:
      - elastic

volumes:
  data01:
    driver: local
  data02:
    driver: local
  data03:
    driver: local

networks:
  elastic:
    driver: bridge
```

设置分片与副本
`PUT http://127.0.0.1:9200/users`
```json
{
	"settings": {
		//设置分片数量
		"number_of_shards": 3,
		//设置分片副本数量
		"number_of_replicas": 1
	}
}
```

集群必须存在一个以上的节点，否则副本不会分配，只存在单主分片工作，数据很有可能丢失
如果在同一台机器启动第二个`ES`节点，只要与第一个`ES`节点存在同样的`cluster.name`配置，则会自动加入集群中
如果在不同机器启动节点，为了加入到同一集群，需要配置一个可连接到的单播主机列表

水平扩容的分片分配，会尽量均匀的迁移主分片与副本分片，会均匀负载每个节点上的分片，且主从分片同时也是错开节点存储并均匀负载的
如果水平扩容的机器节点数量大于分片数量，此时可以动态调整副本数量，扩展副本数量，提高吞吐量
`PUT http://127.0.0.1:9200/users/_settings`
```json
{
	//设置分片副本数量
	"number_of_replicas": 2
}
```

如果节点宕机了，宕机的节点分片，副本会消失，主分片会迁移至其他节点中

文档的存储路由规则按照一定的路由计算出存储位置：`hash(文档主键ID) % 主片数量`
文档的检索读取也是按照一定的分片控制读取分片位置，无论存放该文档的主从分片，比如读取请求负载均衡等
## 9. 读出写入原理
<img src="D:\Project\IT-notes\框架or中间件\ElasticSearch\img\写入原理.png" style="width:700px;height:400px;" />

文档数据写流程：
1. 客户端请求与协调节点
	- 客户端向`Elasticsearch`集群发送一个写入请求，这个请求可以发送到集群中的任何一个节点
	- 接收到请求的节点会充当协调节点的角色。协调节点负责处理客户端的请求，并将请求路由到正确的数据节点
2. 路由与主分片处理
	- 协调节点会根据文档的`_id`和索引的设置（如分片数量）来确定文档应该写入到哪个主分片。这是通过一个哈希函数和模运算来实现的，确保同一个`_id`的文档总是路由到同一个主分片
	- 确定目标主分片后，协调节点将请求转发给该主分片所在的数据节点
	- 数据节点上的主分片接收到请求后，会先将文档写入到内存中的`Lucene`索引结构里。这个过程包括将文档转换成倒排索引的形式，以便后续的搜索和分析
3. 数据同步与副本分片
	- 一旦文档被写入到主分片，主分片会开始将数据同步到其对应的副本分片上。这是为了保证数据的冗余和可用性
	- 副本分片是主分片的完整拷贝，它们可以处理搜索请求并提供数据恢复的能力。当主分片不可用时，副本分片可以被提升为新的主分片
	- 数据同步是异步进行的，这意味着写入请求在主分片处理完毕后就可以返回给客户端，而不需要等待所有副本分片都完成同步
4. 写入确认与响应
	- 当主分片和足够数量的副本分片（根据配置可能是全部或大多数）都成功写入了文档后，协调节点会收到这些分片的确认信息
	- 一旦收到足够的确认信息，协调节点就会向客户端发送一个成功的响应，表示文档已经被成功写入

<img src="D:\Project\IT-notes\框架or中间件\ElasticSearch\img\底层写入机制.png" style="width:700px;height:500px;" />

1. 缓冲区（`Buffer`）和事务日志（`Translog`）
	- 当文档被写入`Elasticsearch`时，它们首先被放置在内存中的一个缓冲区中。这个缓冲区是临时的，用于快速接收并处理写入请求
	- 同时，为了确保数据的持久性和可靠性，每一个写入操作也会被记录到事务日志（`Translog`）中`Translog`是一个追加写入的日志文件，它记录了所有对索引的更改。这种机制类似于数据库中的写前日志（`WAL`）或重做日志（`redo log`），用于在系统崩溃后恢复数据
2. 刷新（Refresh）操作
	- 随着时间的推移，缓冲区中的数据会积累到一定量，此时需要将这些数据刷新（`refresh`）到`Lucene`的索引中。刷新操作会创建一个新的`Lucene`段（`segment`），并将缓冲区中的数据写入这个段中
	- `Lucene`段是不可变的，一旦被写入就不能被修改，这保证了数据的一致性和搜索的高效性。新的段会被添加到索引中，使得新写入的数据可以被搜索到
	- 刷新操作是周期性的，可以通过配置来控制刷新的频率。频繁的刷新会提高数据的实时性，但也会增加`I/O`负担和`CPU`使用率；而较少的刷新则会减少`I/O`操作，但可能会降低数据的实时性
3. `Flush`操作
	- 与刷新不同，`flush`操作会将内存中的数据以及`Translog`中的更改持久化到磁盘上。这意味着数据被真正写入到了物理存储中，而不仅仅是保存在操作系统的文件系统缓存中
	- `Flush`操作会调用操作系统的`fsync`函数来确保数据被写入磁盘，并且会清空相关的缓存和文件（如`Translog`）。这样做可以释放内存空间，并为后续的写入操作做好准备
	- `Flush`操作的频率通常比刷新操作要低得多，因为它涉及到磁盘`I/O`操作，相对较慢。但是，在`Elasticsearch`中，`flush`操作是自动管理的，会根据索引的大小、写入速率和磁盘`I/O`能力等因素来动态调整

<img src="D:\Project\IT-notes\框架or中间件\ElasticSearch\img\读取原理.png" style="width:700px;height:400px;" />

文档数据读流程：
1. 客户端发送请求
	- 当用户想要从`Elasticsearch`中检索数据时，他们会通过`Elasticsearch`的客户端`API`发送一个搜索请求。这个请求包含了查询的详细信息，如要搜索的索引、查询类型（如匹配查询、范围查询等）、过滤条件等
2. 请求到达协调节点
	- 请求首先到达`Elasticsearch`集群中的一个节点，这个节点被称为协调节点（`Coordinating Node`）。协调节点负责接收客户端的请求，处理请求的路由逻辑，并与数据节点（`Data Node`）进行通信以获取实际的数据
3. 解析查询并确定目标分片
	- 协调节点接收到请求后，会解析查询语句，并根据索引的映射（`Mapping`）和设置（`Settings`）信息来确定需要查询哪些分片（`Shard`）。`Elasticsearch`中的每个索引都被分割成多个分片，并且这些分片可以分布在集群的多个节点上以提高可扩展性和性能
4. 将请求转发给数据节点
	- 协调节点根据分片的位置信息将查询请求转发给包含目标分片的数据节点。每个数据节点上都存储着一部分索引的数据，并负责处理与这些数据相关的查询请求
5. 在数据节点上执行查询
	- 数据节点接收到查询请求后，会使用Lucene库来执行实际的搜索操作。`Lucene`是一个高性能、全功能的文本搜索引擎库，它提供了强大的索引和搜索功能。数据节点会根据查询条件在Lucene索引中检索匹配的文档，并生成一个结果集
6. 聚合和排序结果
	- 数据节点将查询结果返回给协调节点。如果查询涉及多个分片，协调节点需要聚合来自不同分片的结果，并根据需要对结果进行排序、分页等处理。这个过程可能需要消耗一定的计算资源，特别是当结果集很大时
7. 返回结果给客户端
	- 一旦结果准备好，协调节点会将它们封装成一个统一的响应格式，并返回给客户端。响应中包含了查询的结果、匹配的文档数量、聚合数据（如果有的话）等信息。客户端可以解析这个响应来获取所需的数据
## 10. 倒排索引
<img src="D:\Project\IT-notes\框架or中间件\ElasticSearch\img\倒排索引原理1.png" style="width:700px;height:300px;" />

<img src="D:\Project\IT-notes\框架or中间件\ElasticSearch\img\倒排索引原理2.png" style="width:700px;height:350px;" />

- 单词词典（`Term Dictionary`) ：记录所有文档的单词，记录单词到倒排列表的关联关系
- 倒排列表（`Posting List`）：记录了单词对应的文档结合，由倒排索引项组成
- 倒排索引项（`Posting`）：
	- 文档`ID`
	- 词频`TF`：该单词在文档中出现的次数，用于相关性评分
	- 位置 (`Position`)：单词在文档中分词的位置。用于短语搜索（`match phrase query`)
	- 偏移 (`Offset`)：记录单词的开始结束位置，实现高亮显示
## 11. 分词分析器
文本分析就是**把全文本转换成一系列单词（term/token）的过程**，也叫**分词**。在`ES`中，`Analysis`是通过**分词器（Analyzer）** 来实现的，可使用`ES`内置的分析器或者按需定制化分析器

分词器由三部分组成：
- `character filter`：接收原字符流，通过添加、删除或者替换操作改变原字符流
- `tokenizer`：简单的说就是将一整段文本拆分成一个个的词，即分隔符
- `token filters`：将切分的单词添加、删除或者改变

<img src="D:\Project\IT-notes\框架or中间件\ElasticSearch\img\分词分析器大致原理.png" style="width:700px;height:800px;" />

分词器存在两个使用的地方：
1. 创建索引时
2. 进行搜索时

如果设置手动设置了分词器，将按照下面顺序来确定使用哪个分词器
1. 先判断字段是否有设置分词器，如果有，则使用字段属性上的分词器设置
2. 如果设置了`analysis.analyzer.default`，则使用该设置的分词器
3. 如果上面两个都未设置，则使用默认的`standard`分词器
```
创建索引并定义字段时指定字段分词分析器
PUT my_index
{
  "mappings": {
    "properties": {
      "title":{
        "type":"text",
        "analyzer": "whitespace"
      }
    }
  }
}

创建索引时指定默认的分词分析器
PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default":{
          "type":"simple"
        }
      }
    }
  }
}
```

搜索时通过以下顺序依次查找分析分词器
1. 搜索时指定`analyzer`参数
2. 创建`mapping`时指定字段的`search_analyzer`属性
3. 创建索引时指定`setting`的`analysis.analyzer.default_search`
4. 查看创建索引时字段指定的`analyzer`属性
5. 如果上面几种都未设置，则使用默认的`standard`分词器
```
GET my_index/_search
{
  "query": {
    "match": {
      "message": {
        "query": "Quick foxes",
        "analyzer": "stop"
      }
    }
  }
}

PUT my_index
{
  "mappings": {
    "properties": {
      "title":{
        "type":"text",
        "analyzer": "whitespace",
        "search_analyzer": "simple"
      }
    }
  }
}

PUT my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "default":{
          "type":"simple"
        },
        "default_search":{
          "type":"whitespace"
        }
      }
    }
  }
}
```

内置分词分析器类别

| 分析器                                                                                                          | 描述                                                                                            | 分词对象                                                     | 结果                                                                       |
| ------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------- | -------------------------------------------------------- | ------------------------------------------------------------------------ |
| standard                                                                                                     | 标准分析器是默认的分析器，如果没有指定，则使用该分析器。它提供了基于文法的标记化(基于 Unicode 文本分割算法，如 Unicode 标准附件 29所规定) ，并且对大多数语言都有效 | The 2 QUICK Brown-Foxes jumped over the lazy dog’s bone. | \[ the, 2, quick, brown, foxes, jumped, over, the, lazy, dog’s, bone \]  |
| simple                                                                                                       | 简单分析器将文本分解为任何非字母字符的标记，如数字、空格、连字符和撇号、放弃非字母字符，并将大写字母更改为小写字母                                     | The 2 QUICK Brown-Foxes jumped over the lazy dog’s bone. | \[ the, quick, brown, foxes, jumped, over, the, lazy, dog, s, bone \]    |
| whitespace                                                                                                   | 空格分析器在遇到空白字符时将文本分解为术语                                                                         | The 2 QUICK Brown-Foxes jumped over the lazy dog’s bone. | \[ The, 2, QUICK, Brown-Foxes, jumped, over, the, lazy, dog’s, bone. \]  |
| stop                                                                                                         | 停止分析器与简单分析器相同，但增加了删除停止字的支持。默认使用的是 `_english_` 停止词                                             | The 2 QUICK Brown-Foxes jumped over the lazy dog’s bone. | \[ quick, brown, foxes, jumped, over, lazy, dog, s, bone \]              |
| keyword                                                                                                      | 不分词，把整个字段当做一个整体返回                                                                             | The 2 QUICK Brown-Foxes jumped over the lazy dog’s bone. | \[The 2 QUICK Brown-Foxes jumped over the lazy dog’s bone.\]             |
| pattern                                                                                                      | 模式分析器使用正则表达式将文本拆分为术语。正则表达式应该匹配令牌分隔符，而不是令牌本身。正则表达式默认为 `w+` (或所有非单词字符)                          | The 2 QUICK Brown-Foxes jumped over the lazy dog’s bone. | \[ the, 2, quick, brown, foxes, jumped, over, the, lazy, dog, s, bone \] |
| 多种西语系 arabic, armenian, basque, bengali, brazilian, bulgarian, catalan, cjk, czech, danish, dutch, english等等 | 一组旨在分析特定语言文本的分析程序                                                                             |                                                          |                                                                          |
| ik_smart                                                                                                     | ik分词器中的简单分词器，支持自定义字典，远程字典                                                                     | 学如逆水行舟，不进则退                                              | \[学如逆水行舟,不进则退\]                                                          |
| ik_max_word                                                                                                  | ik_分词器的全量分词器，支持自定义字典，远程字典                                                                     | 学如逆水行舟，不进则退                                              | \[学如逆水行舟,学如逆水,逆水行舟,逆水,行舟,不进则退,不进,则,退\]                                   |
## 12. 锁

## 13. Kibana
**docker部署**
```yml
version: "3"
services:
  elasticsearch:
    container_name: single-es
    image: docker.elastic.co/elasticsearch/elasticsearch:7.11.1
    environment:
      - ES_JAVA_OPTS=-Xms1024m -Xmx1024m
      - TZ=Asia/Shanghai
      - discovery.type=single-node
    ports:
      - 9200:9200
      - 9300:9300
    networks:
      elastic-net:
        ipv4_address: 11.11.11.10
  kibana:
    container_name: single-kibana
    image: docker.elastic.co/kibana/kibana:7.11.1
    environment:
      - TZ=Asia/Shanghai
      - I18N_LOCALE=zh-CN
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    ports:
      - 5601:5601
    networks:
      elastic-net:
        ipv4_address: 11.11.11.11
    depends_on:
      - elasticsearch
networks:
  elastic-net:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 11.11.0.0/16
          gateway: 11.11.0.1
```
## 14. FileBeat
`Beats`为轻量日志采集器，包含六种工具
- `Packetbeat`：网络数据，收集网络流量数据
- `Metricbeat`：指标，收集系统、进程和文件系统级别的CPU和内存使用情况等数据
- `Filebeat`：日志文件，收集文件数据
- `Winlogbeat`：`windows`事件日志，收集`Windows`事件日志数据
- `Auditbeat`：审计数据，收集审计日志
- `Heartbeat`：运行时间监控，收集系统运行时的数据

`Filebeat`用于转发和集中日志数据的轻量级传送工具，监视指定的日志文件，收集日志时间，转发到`ES`或`logstash`

工作方式：启动`Filebeat`时，它将启动一个或多个输入，这些输入将在为日志数据指定的位置中查找。对于`Filebeat`所找到的每个日志，`Filebeat`都会启动收集器。每个收集器都读取单个日志以获取新内容，并将新日志数据发送到`libbeat`，`libbeat`将聚集事件，并将聚集的数据发送到为`Filebeat`配置的输出

<img src="D:\Project\IT-notes\框架or中间件\ElasticSearch\img\filebeat原理.png" style="width:700px;height:500px;" />

`filebeat`结构：由两个组件构成，分别是`inputs`（输入）和`harvesters`（收集器），这些组件一起工作来跟踪文件并将事件数据发送到您指定的输出
- `harvester`负责读取单个文件的内容，逐行读取每个文件，并将内容发送到输出，为每个文件启动一个`harvester`。`harvester`负责打开和关闭文件，这意味着文件描述符在`harvester`运行时保持打开状态。如果在收集文件时删除或重命名文件，`Filebeat`将继续读取该文件。这样做的副作用是，磁盘上的空间一直保留到`harvester`关闭
- 一个`input`负责管理`harvesters`和寻找所有来源读取。如果`input`类型是`log`，则`input`将查找驱动器上与定义的路径匹配的所有文件，并为每个文件启动一个`harvester`。每个`input`在它自己的`Go`进程中运行，`Filebeat`当前支持多种输入类型。每个输入类型可以定义多次。日志输入检查每个文件，以查看是否需要启动`harvester`、是否已经在运行`harvester`或是否可以忽略该文件

`Filebeat`保留每个文件的状态，并经常将状态刷新到磁盘中的**注册表文件**中：
- 该状态用于记住`harvester`读取的最后一个偏移量，并确保发送所有日志行。如果无法访问输出（如`Elasticsearch`或`Logstash`），`Filebeat`将跟踪最后发送的行，并在输出再次可用时继续读取文件
- 当`Filebeat`运行时，每个输入的状态信息也保存在内存中。当`Filebeat`重新启动时，来自注册表文件的数据用于重建状态，`Filebeat`在最后一个已知位置继续每个`harvester`。对于每个输入，`Filebeat`都会保留它找到的每个文件的状态。由于文件可以重命名或移动，文件名和路径不足以标识文件。对于每个文件，`Filebeat`存储唯一的标识符，以检测文件是否以前被捕获

**示例：filebeat收集nginx日志**
`filebeat.yml`
```yml
setup.kibana:
	host: "localhost:5601"
output.elasticsearch:
	hosts: ["localhost:9200"]
```

执行`shell`命令
`./filebeat modules enable nginx`

`modules.d/nginx.yml`
```yml
- module: nginx # Access logs
	access: 
		enabled: true
		# Set custom paths for the log files. If left empty,
		# Filebeat will choose the paths depending on your OS.
		var.paths: ["/Users/liuxg/data/nginx.log"]
	# Error logs
	error: 
		enabled: true
		# Set custom paths for the log files. If left empty,
		# Filebeat will choose the paths depending on your OS.
		#var.paths: ["/var/log/nginx/error.log"]
```

执行`shell`命令
`./filebeat setup`
`./filebeat -e`
## 15. Logstash
**示例：filebeat读取nginx，交由logstash处理，再输出至es**
1. 配置`nginx`日志输出格式，`vim nginx.conf`
```
log_format main '$remote_addr - $remote_user [$time_local] '
				'"$request" $status $body_bytes_sent '
				'"$http_referer" "$http_user_agent"';
access_log /var/log/nginx/access.log main
error_log /var/log/nginx/error.log main
```
2. 编写`logstash`针对`nginx`的日志解析规则，`vim nginx-patterns`
```
NGINX_ACCESS %{IPORHOST:remote_addr} - %{USERNAME:remote_user} \[%{HTTPDATE:time_local}\] \"%{DATA:request}\" %{INT:status} %{NUMBER:bytes_sent} \"%{DATA:http_referer}\" \"%{DATA:http_user_agent}\"
```
*tips：grok存在预定义匹配字段*

| 别名       | 正则                                          | 释义                                      |
| -------- | ------------------------------------------- | --------------------------------------- |
| IPORHOST | (?:%{HOSTNAME}\|%{IP})                      | 主机名称或者IP                                |
| USERNAME | \[a-zA-Z0-9._-\]+                           | 用户名，由数字、大小写及特殊字符 .\_- 组成的字符串            |
| HTTPDATE | %{MONTHDAY}/%{MONTH}/%{YEAR}:%{TIME} %{INT} | http默认日期格式，如：03/Jul/2016:00:36:53 +0800 |
| DATA     | .*?                                         | 匹配换行符                                   |
| INT      | (?:\[+-\]?(?:\[0-9\]+))                     | 整数，包括0和正负整数                             |
| NUMBER   | (?:%{BASE10NUM})                            | 十进制数字，包括整数和小数                           |
3. 配置`filebeat`，`filebeat-nginx.yml`
```yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
  tags: ["log"]
  fields:
    from: nginx
  fields_under_root: false
output.logstash:
  hosts: ["192.168.1.7:5044"]
```
4. 启动`filebeat`，`./filebeat -e -c filebeat-nginx.yml`
5. 配置`logstash`，`logstash-nginx.conf`
```
input {
	beats {
		port => "5044"
	}
}

filter {
	grok {
		patterns_dir => "/logstash-7.11.1/nginx-patterns"
		match => { "message" => "%{NGINX_ACCESS}" }
		remove_tag => [ "_grokparsefailure" ]
		add_tag => [ "nginx_access" ]
	}
}

output {
	elasticsearch {
		hosts => [ "192.168.1.7:9200","192.168.1.7:9201","192.168.1.7:9202"  ]
		index => "logstash-poem"
		document_type => "_doc"
	}
}
```
6. 启动`logstash`，`logstash -f logstash-nginx.conf --config.test_and_exit`测试连接
7. 启动`logstash`，`logstash -f logstash-nginx.conf --config.reload.automatic`，热加载`conf`配置文件
## 16. SpringBoot整合ES
**依赖**
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.15</version>
    <relativePath />
</parent>
<!--springBoot2.5.15对应Elasticsearch7.12.0版本-->
<!--elasticsearch-->
<dependency>
    <groupId>org.elasticsearch</groupId>
    <artifactId>elasticsearch</artifactId>
    <version>7.12.0</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

**applicatio.yml**
```yml
spring:
	elasticsearch:
		rest:
			uris: 192.168.1.36:9200
			connection-timeout: 1s
			read-timeout: 30s
```

**实体类**
```java
import lombok.Data;
import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Field;
import org.springframework.data.elasticsearch.annotations.FieldType;

//@Document 文档对象 （索引信息、文档类型 ）
@Document(indexName="blog3")
@Data
public class ElasticSearch {

    //@Id 文档主键 唯一标识
    @Id
    //@Field 每个文档的字段配置（类型、是否分词、是否存储、分词器 ）
    @Field(store=true, index = false,type = FieldType.Integer)
    private Integer id;

    @Field(index=true,analyzer="ik_smart",store=true,searchAnalyzer="ik_smart",type = FieldType.Text)
    private String title;

    @Field(index=true,analyzer="ik_smart",store=true,searchAnalyzer="ik_smart",type = FieldType.Text)
    private String content;

    @Field(index=true,store=true,type = FieldType.Double)
    private Double price;
}
```

**mapper**
```java
import com.economics.project.es.domain.ElasticSearch;
import org.springframework.data.elasticsearch.annotations.Highlight;
import org.springframework.data.elasticsearch.annotations.HighlightField;
import org.springframework.data.elasticsearch.annotations.HighlightParameters;
import org.springframework.data.elasticsearch.core.SearchHit;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;
import org.springframework.stereotype.Repository;
import java.util.List;

@Repository
public interface ElasticSearchRepository extends ElasticsearchRepository<ElasticSearch, Integer> {

    /**
     * 查询内容标题查询
     * @param title 标题
     * @param content 内容
     * @return 返回关键字高亮的结果集
     */
    @Highlight(
            fields = {@HighlightField(name = "title"), @HighlightField(name = "content")},
            parameters = @HighlightParameters(preTags = {"<span style='color:red'>"}, postTags = {"</span>"}, numberOfFragments = 0)
    )
    List<SearchHit<ElasticSearch>> findByTitleOrContent(String title, String content);

}
```

**service**
```java
import com.economics.project.es.domain.ElasticSearch;
import org.springframework.data.elasticsearch.core.SearchHit;
import java.util.List;

public interface ElasticSearchService {

    //保存和修改
    void save(ElasticSearch article);
    //查询id
    ElasticSearch findById(Integer id);
    //删除指定ID数据
    void   deleteById(Integer id);

    long count();
    
    boolean existsById(Integer id);

    List<SearchHit<ElasticSearch>> findByTitleOrContent(String title, String content);

}


import com.economics.project.es.domain.ElasticSearch;
import com.economics.project.es.service.ElasticSearchService;
import com.economics.project.es.service.ElasticSearchRepository;
import org.springframework.data.elasticsearch.core.SearchHit;
import org.springframework.stereotype.Service;
import javax.annotation.Resource;
import java.util.List;

@Service
public class ElasticSearchServiceImpl implements ElasticSearchService {

    @Resource
    private ElasticSearchRepository ElasticSearchRepository;


    @Override
    public void save(ElasticSearch ElasticSearch) {
        ElasticSearchRepository.save(ElasticSearch);
    }

    @Override
    public ElasticSearch findById(Integer id) {
        return ElasticSearchRepository.findById(id).orElse(new ElasticSearch());
    }

    @Override
    public void deleteById(Integer id) {
        ElasticSearchRepository.deleteById(id);
    }

    @Override
    public long count() {
        return ElasticSearchRepository.count();
    }

    @Override
    public boolean existsById(Integer id) {
        return ElasticSearchRepository.existsById(id);
    }

    @Override
    public List<SearchHit<ElasticSearch>> findByTitleOrContent(String title, String content) {
        return ElasticSearchRepository.findByTitleOrContent(title,content);
    }

}
```

**自定义查询：自定义方法的前提是我们需要继承ElasticsearchRepository接口，利用强大的Spring Data来实现。比如：你的方法名叫做：findByTitle，那么它就知道你是根据title查询，然后自动帮你完成，无需写实现类**

| 关键字                   | 解释                      | 方法                                                   |
| --------------------- | ----------------------- | ---------------------------------------------------- |
| `and`                 | 根据`Field1`和`Field2`获得数据 | `findByTitleAndContent(String title,String content)` |
| `or`                  | 根据`Field1`或`Field2`获得数据 | `findByTitleOrContent(String title,String content)`  |
| `is`                  | 根据`Field`获得数据           | `findByTitle(String title)`                          |
| `not`                 | 根据`Field`获得相反数据         | `findByTitleNot(String title)`                       |
| `between`             | 获得指定范围的数据               | `findByPriceBetween(double price1, double price2)`   |
| `lessThanEqual`       | 获得小于等于指定值的数据            | `findByPriceLessThan(double price)`                  |
| `GreaterThanEqual`    | 获得大于等于指定值的数据            | `findByPriceGreaterThan(double price)`               |
| `Before`              |                         | `findByPriceBefore`                                  |
| `After`               |                         | `findByPriceAfter`                                   |
| `Like`                | 比较相识的数据                 | `findByNameLike`                                     |
| `StartingWith`        | 以`xx`开头的数据              | `findByNameStartingWith(String Name)`                |
| `EndingWith`          | 以`xx`结尾的数据              | `findByNameEndingWith(String Name)`                  |
| `Contains/Containing` | 包含的数据                   | `findByNameContaining(String Name)`                  |
| `In`                  | 多值匹配                    | `findByNameIn(Collectionnames)`                      |
| `NotIn`               | 多值不匹配                   | `findByNameNotIn(Collectionnames)`                   |
| `OrderBy`             | 排序后的数据                  | `findByxxxxxOrderByNameDesc(String xxx)`             |
## 17. Easy-ES
**依赖**
```xml
<!-- 排除springboot中内置的es依赖,以防和easy-es中的依赖冲突-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<exclusions>
		<exclusion>
			<groupId>org.elasticsearch.client</groupId>
			<artifactId>elasticsearch-rest-high-level-client</artifactId>
		</exclusion>
		<exclusion>
			<groupId>org.elasticsearch</groupId>
			<artifactId>elasticsearch</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<dependency>
	<groupId>org.elasticsearch.client</groupId>
	<artifactId>elasticsearch-rest-high-level-client</artifactId>
	<version>7.14.0</version>
</dependency>
<dependency>
	<groupId>org.elasticsearch</groupId>
	<artifactId>elasticsearch</artifactId>
	<version>7.14.0</version>
</dependency>
<dependency>
	<groupId>cn.easy-es</groupId>
	<artifactId>easy-es-boot-starter</artifactId>
	<version>2.0.0-beta1</version>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>
```

**yml配置**
```yml
easy-es:
  enable: true
  address : ES地址:9200
  global-config:
    process_index_mode: manual
```

**扫描mapper**
```java
@EsMapperScan("com.mine.easyEs.mapper")
```

**实体类**
```java
@Data
public class Document {
    @Id
    /**
     * es中的唯一id
     */
    private String id;
    /**
     * 文档标题
     */
    private String title;
    /**
     * 文档内容
     */
    private String content;
    /**
     * 创建时间
     */
    private Date createTime;
}
```

**controller**
```java
@RestController
@RequestMapping("/ee")
@RequiredArgsConstructor(onConstructor = @__(@Autowired))
public class DocumentController {

    private final IDocumentService documentService;

    /**
     * 创建索引
     * @return 结果信息
     * @throws Exception
     */
    @GetMapping("/createIndex")
    public String createIndex() throws Exception {
        return documentService.createIndex();
    }

    /**
     * 删除索引
     * @return 结果信息
     */
    @GetMapping("/deleteIndex")
    public String deleteIndex(){
        return documentService.deleteIndex();
    }

    /**
     * 查询ES所有数据
     * @return 查询Document结果对象集合
     */
    @GetMapping("/findAll")
    public List<Document> findAll(){
        return documentService.findAllData();
    }

    /**
     * ES新增数据
     * @param document 新增数据对象
     * @return 结果信息
     * @throws Exception
     */
    @GetMapping("/add")
    public String addData(Document document) throws Exception {
        return documentService.addData(document);
    }

    /**
     * 修改ES数据
     * @param document 修改数据对象
     */
    @GetMapping("/update")
    public String updateData(Document document){
        return documentService.updateData(document);
    }

    /**
     * 根据id删除ES数据
     * @param id 需要删除的数据的id
     * @return
     */
    @GetMapping("/delete")
    public String deleteData(String id){
        return documentService.deleteDataById(id);
    }

    /**
     * 分词匹配查询content字段
     * @param value 查询内容
     * @return
     */
    @GetMapping("/match")
    public List<Document> findMatch(String value){
        return documentService.findMatch(value);
    }
}
```

**service**
```java
public interface IDocumentService {

    /**
     * 查询ES所有数据
     * @return 查询Document结果对象集合
     */
    List<Document> findAllData();

    /**
     * 创建索引
     * @return 结果信息
     * @throws Exception
     */
    String createIndex() throws Exception;

    /**
     * 删除索引
     * @return 结果信息
     */
    String deleteIndex();

    /**
     * ES新增数据
     * @param document 新增数据实体类
     * @return 结果信息
     * @throws Exception
     */
    String addData(Document document) throws Exception;

    /**
     * 根据id删除ES数据
     * @param id 需要删除的数据的id
     * @return
     */
    String deleteDataById(String id);

    /**
     * 修改ES数据
     * @param document 修改数据对象
     */
    String updateData(Document document);

    /**
     * 分词匹配查询content字段
     * @param value 查询内容
     * @return
     */
    List<Document> findMatch(String value);
}

@Service
@RequiredArgsConstructor(onConstructor = @__(@Autowired))
public class DocumentServiceImpl implements IDocumentService {

    private final DocumentMapper documentMapper;

    /**
     * 查询ES所有数据
     * @return 查询Document结果对象集合
     */
    @Override
    public List<Document> findAllData() {
        LambdaEsQueryWrapper<Document> wrapper = new LambdaEsQueryWrapper<>();
        wrapper.matchAllQuery();
        return documentMapper.selectList(wrapper);
    }

    /**
     * 创建索引
     * @return 结果信息
     * @throws Exception
     */
    @Override
    public String createIndex() throws Exception {
        StringBuilder msg = new StringBuilder();
        String indexName = Document.class.getSimpleName().toLowerCase();
        boolean existsIndex = documentMapper.existsIndex(indexName);
        if (existsIndex){
            throw new Exception("Document实体对应索引已存在,删除索引接口：deleteIndex");
        }
        boolean success = documentMapper.createIndex();
        if (success){
            msg.append("Document索引创建成功");
        }else {
            msg.append("索引创建失败");
        }
        return msg.toString();
    }

    /**
     * 删除索引
     * @return 结果信息
     */
    @Override
    public String deleteIndex() {
        StringBuilder msg = new StringBuilder();
        String indexName = Document.class.getSimpleName().toLowerCase();
        if (documentMapper.deleteIndex(indexName)){
            msg.append("删除成功");
        }else {
            msg.append("删除失败");
        }
        return msg.toString();
    }

    /**
     * ES新增数据
     * @param document 新增数据实体类
     * @return 结果信息
     * @throws Exception
     */
    @Override
    public String addData(Document document) throws Exception {
        if (StringUtils.isEmpty(document.getTitle()) || StringUtils.isEmpty(document.getContent())) {
            throw new Exception("请补全title及content数据");
        }
        document.setCreateTime(new Date());
        documentMapper.insert(document);
        return "Added successfully！";
    }

    /**
     * 根据id删除ES数据
     * @param id 需要删除的数据的id
     * @return
     */
    @Override
    public String deleteDataById(String id) {
        documentMapper.deleteById(id);
        return "Success";
    }

    /**
     * 修改ES数据
     * @param document 修改数据对象
     */
    @Override
    public String updateData(Document document) {
        documentMapper.updateById(document);
        return "Success";
    }


    /**
     * 分词匹配查询content字段
     * @param value 查询内容
     * @return
     */
    @Override
    public List<Document> findMatch(String value) {
        LambdaEsQueryWrapper<Document> wrapper = new LambdaEsQueryWrapper<>();
        wrapper.match(Document::getContent,value);
        wrapper.orderByDesc(Document::getCreateTime);
        List<Document> documents = documentMapper.selectList(wrapper);
        return documents;
    }
}
```

**mapper**
```java
public interface DocumentMapper extends BaseEsMapper<Document> {
}
```
## 18. MySQL同步ES