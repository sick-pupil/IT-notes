## 1. 通用表达式
### 1. 临时表
```sql
with tmp(a) as (select 1 union all select 2) select * from tmp

SELECT * FROM
(
	WITH recursive seq_date (log_date) AS
	(
		SELECT '2020-01-01'
		UNION ALL
		SELECT log_date + INTERVAL 1 DAY
		FROM seq_date
		WHERE log_date + interval 1 day  < '2020-02-01'
	) select * FROM seq_date
) X LIMIT 10
```

### 2. `with insert`
```sql
INSERT y1 (r1,log_date)
WITH recursive tmp (a, b) AS
(
	SELECT 1, '2021-04-20'
	UNION ALL 
	SELECT ROUND(RAND() * 10), b - INTERVAL ROUND(RAND() * 1000) DAY
	FROM tmp 
	LIMIT 100
) TABLE tmp;
```

### 3. `with update`
```sql
WITH recursive tmp (a, b, c) AS
(
     SELECT 1, 1, '2021-04-20'
     UNION ALL
     SELECT a + 2, 100, DATE_SUB(CURRENT_DATE(), INTERVAL ROUND(RAND() * 1000, 0) DAY)
     FROM tmp
     WHERE a < 100
) UPDATE tmp AS a, y1 AS b
	SET b.r1 = a.b 
	WHERE a.a = b.id;
```

### 4. `with delete`
```sql
WITH recursive tmp (a) AS
(
	SELECT 1
	UNION ALL
	SELECT a + 2
	FROM tmp
	WHERE a < 100
) DELETE FROM y1 WHERE id IN (TABLE tmp);
```

### 5. 嵌套`with`
```sql
SELECT * FROM  
(
	WITH tmp1 (a, b, c) AS 
	(
		VALUES 
			ROW (1, 2, 3),
			ROW (3, 4, 5),
			ROW (6, 7, 8)
	) SELECT * FROM 
	(
		WITH tmp2 (d, e, f) AS (
        VALUES
	        ROW (100, 200, 300),
            ROW (400, 500, 600)
        ) TABLE tmp2
    ) X JOIN tmp1 Y
) Z ORDER BY a;
```

## 2. 窗口函数
```sql
-- 分区中的序号
select stu_no, 
	course, 
	score, 
	row_number() over(partition by course order by score desc) rn
from tb_score;

-- 分区中排序值相同的使用同一序号，但排序值相同的若干行之后，下一个序号为上一个+1
select stu_no, 
	course, 
	score, 
	dense_rank() over(partition by course order by score desc) rn 
from tb_score;

-- 分区中排序值相同的使用同一序号，但排序值相同的若干行之后，下一个序号为上若干n行排序号相同的序号+n
select stu_no, 
	course, 
	score, 
	rank() over(partition by course order by score desc) rn 
from tb_score;

-- 分区后再分区
select stu_no, 
	course, 
	score, 
	rank() over(partition by course order by score desc) rn,
	NTILE(2) over(partition by course order by score desc) rn_group 
from tb_score;
```

|**类别**|**函数**|**说明**|
|---|---|---|
|排序|ROW_NUMBER|为表中的每一行分配一个序号，可以指定分组（也可以不指定）及排序字段|
|排序|DENSE_RANK|根据排序字段为每个分组中的每一行分配一个序号。 排名值相同时，序号相同，序号中没有间隙（1,1,2,3这种）|
|排序|RANK|根据排序字段为每个分组中的每一行分配一个序号。 排名值相同时，序号相同，但序号中存在间隙（1,1,3,4这种）|
|排序|NTILE|根据排序字段为每个分组中根据指定字段的排序再分成对应的组|
|分布|PERCENT_RANK|计算各分组或结果集中行的百分数等级|
|分布|CUME_DIST|计算某个值在一组有序的数据中累计的分布|
|前后|LEAD|返回分组中当前行之后的第N行的值。如果不存在对应行，则返回NULL。比如N=1时，第一名对应的值是第二名的，最后一名结果是NULL|
|前后|LAG|返回分组中当前行之前的第N行的值。如果不存在对应行，则返回NULL。比如N=1时，第一名对应的值是是NUL，最后一名结果是倒数第2的值|
|首尾中|FIRST_VALUE|返回每个分组中第一名对应的字段（或表达式）的值，例如本文中可以是第一名的分数、学号等任意字段的值|
|首尾中|LAST_VALUE|返回每个分组中最后一名对应的字段（或表达式）的值，例如本文中可以是最后一名的分数、学号等任意字段的值|
|首尾中|NTH_VALUE|返回每个分组中排名第N的对应字段（或表达式）的值，但小于N的行对应的值是NULL|

## 3. JSON类型
```sql
-- 使用JSON_ARRAY方法定义JSON数组；
SELECT JSON_ARRAY(1, "abc", NULL, TRUE, CURTIME())
-- 结果：[1, "abc", null, true, "11:30:24.000000"]  

-- JSON_OBJECT 方法定义JSON对象
SELECT JSON_OBJECT('id', 87, 'name', 'carrot')
-- 结果{"id": 87, "name": "carrot"}

-- 数组 与 对象嵌套的场景；
[99, {"id": "HK500", "cost": 75.99}, ["hot", "cold"]] {"k1": "value", "k2": [10, 20]}

-- 日期/时间类型定义
["12:18:29.000000", "2015-07-29", "2015-07-29 12:18:29.000000"]

-- JSON_QUOTE 将JSON对象转义成String, 就是将内部的符  号进行转义，并整体包裹上双引号；
JSON_QUOTE(' "null" ')
-- 结果 "\"null\""

-- 将JSON内容美化并输出；
JSON_PRETTY()

-- 可以将JSON/JSON内部的元素转化为其他数据类型;
-- 如下将JSON jdoc 中的id元素，转化为 unsigned int;
ORDER BY CAST(JSON_EXTRACT(jdoc, '$.id') AS   UNSIGNED);

-- {"mascot": "Our mascot is a dolphin named \"Sakila\"."}
mysql> SELECT col->"$.mascot" FROM qtest;
-- 结果：| "Our mascot is a dolphin named \"Sakila\"." |
SELECT sentence->>"$.mascot" FROM facts;
-- 结果： | Our mascot is a dolphin named "Sakila". |

-- [3, {"a": [5, 6], "b": 10}, [99, 100]]
-- $[0] = 3 ;
-- $[1] = {"a": [5, 6], "b": 10};
-- $[2] = [99, 100];

-- $[1].a = [5,6]
-- $[1].a[1] = 6
-- $[1].b = 10;
-- $[2][0] = 99;

-- 如上, 应该可以用-->语法取代;
mysql> SELECT JSON_EXTRACT('{"a": 1, "b": 2, "c": [3, 4, 5]}', '$.*');
-- [1, 2, [3, 4, 5]]  
SELECT JSON_EXTRACT('{"a": 1, "b": 2, "c": [3, 4, 5]}', '$.c[*]')
-- [3, 4, 5]
SELECT JSON_EXTRACT('{"a": {"b": 1}, "c": {"b": 2}}', '$**.b');
-- [1, 2]
SELECT JSON_EXTRACT('[1, 2, 3, 4, 5]', '$[1 to 3]');
-- [2, 3, 4]

-- JSON_SET JSON_INSERT JSON_REPLACE JSON_REMOVE
SET @j = '["a", {"b": [true, false]}, [10, 20]]';
SELECT JSON_SET(@j, '$[1].b[0]', 1, '$[2][2]', 2);
-- | ["a", {"b": [1, false]}, [10, 20, 2]]    

SELECT JSON_INSERT(@j, '$[1].b[0]', 1, '$[2][2]', 2);
-- ["a", {"b": [true, false]}, [10, 20, 2]]

JSON_REPLACE(@j, '$[1].b[0]', 1, '$[2][2]', 2)
-- ["a", {"b": [1, false]}, [10, 20]]

SELECT JSON_REMOVE(@j, '$[2]', '$[1].b[1]', '$[1].b[1]');
-- ["a", {"b": [true]}]
```

## 4. 原子DDL

## 5. 全局变量修改持久化
在8之前的版本中，对于全局变量的修改，其只会影响其内存值，而不会持久化到配置文件中。数据库重启，又会恢复成修改前的值。从8开始，可通过SET PERSIST命令将全局变量的修改持久化到配置文件中

```shell
show variables like '%max_connections%';
set persist max_connections=300;
show variables like '%max_connections%';
```

## 6. 行锁改进
```sql
select * from tx where c1=12 for update skip locked; -- 需要加锁的记录若被其它线程占有锁，则跳过，而不是等待

select * from tx where c1=12 for update nowait; -- 需要加锁的记录有锁则报错
```