<img src="D:\Project\IT notes\框架or中间件\Mysql\img\基础大纲.png" style="width:700px;height:250px;" />

## 1. 基本的select语句
```sql
select * from students --查询students表中的所有记录的所有字段--

select id,name,age from students --查询students表中的所有记录的id、name、age字段--

select distinct name from students --查询students表中所有记录互不相同的name--

select distinct name,age from students --查询students表中name、age两个字段不同的组合--

select * from students where sex='男' and age > 20 --查询性别为男，年龄大于20的记录--

select * from students order by age desc --查询students表中所有记录，记录之间根据age字段降序排序--
--	desc降序,asc升序
--	order by后可以跟多个不同排序字段
--	第一个排序字段值相同，则会根据第二个排序字段进行排序
--	只有一个排序字段，字段值相同的记录会无序排列

select * from students order by mark desc limit 0,5 --取出成绩前五的记录，起始偏移量为0，取包括起始偏移量后的5条记录--

select count(1) from students --统计students全表记录数--

select first_name, count(1) from students group by first_name --根据first_name分组，检索出各组的first_name与记录数--

select first_name, count(1) from students group by first_name with rollup --with rollup：在group by分组基础上再进行不分组的统计数据--

select first_name, count(1) from students group by first_name having count(1) > 4 --having针对group by分组后对分组进行过滤--

select count(1), min(mark), max(mark) from students

select id,first_name from students where first_name like '_guo%' --模糊查询，guo前面任意匹配一个字符，后面匹配任意多个字符--

select id,first_name from students where id in(1,2)
select id,first_name from students where id in (select ......) --in子查询--
select * from A where exists (select 1 from B where B.id=A.id) --exists子查询--
```

## 2. 运算符
算术运算符：
- 加法运算 `+`
- 减法运算 `-`
- 乘法运算 `*`
- 除法运算 `/` `div`
- 求余运算 `%` `mod`

比较运算符：
- 等于 `=`
- 大于 `>`
- 小于 `<`
- 大于等于 `>=`
- 小于等于 `<=`
- 不等于 `!=` `<>`
- n/a `is null`  `is not null`  `between and`  `in`  `not in`
- 模式匹配 `like`  `not like`
- 正则表达 `regexp`

逻辑运算符：
- 与 `&&`   `AND`
- 或 `||`  `OR`

位运算符：
- `&`
- `|`
- `~`
- `^`
- `<<`
- `>>`

## 3. 排序与分页
`order by`默认`asc`升序，`desc`为降序
单列排序：`select last_name, job_id, department_id, hire_date from employees order by hire_date`
多列排序：`select last_name, department_id, salary from employees order by department_id, salary desc`

分页公式：`select * from table limit (pageNum - 1) * pageSize, pageSize`

## 4. 多表查询
<img src="D:\Project\IT notes\框架or中间件\Mysql\img\多表连结.png" style="width:700px;height:500px;" />

## 5. 单行函数
### 字符串函数
- length('john')
- concat(last_name, '\_', first_name)
- upper('john')
- lower('JOHN')
- substr('zhangjin', 6) \# jin
- substr('zhangjin', 1,5) \# zhang
- instr('zhangjin', 'jin') \# 6
- trim(' zhangjin ')
- lpad('张', 3, '\*') \# \*\*张
- lpad('张三丰', 2, '\*') \# 张三
- replace(str, from_str, to_str)

### 数学函数
- round()四舍五入
- ceil()向上取整
- floor向下取整
- truncate()截断
- mod()取余

### 日期函数
- now()
- curdate()
- curtime()
- year()
- month()
- day()

## 6. 聚合函数
- avg()
- count()
- max()
- min()
- sum()
- group_concat()

## 7. DML
```sql
insert into tablename(field1,field2,...,fieldn) values(val1, val2,...,valn)

insert into tablename(field1,field2,...,fieldn)
values
(val1,val2,...,valn),
(val1,val2,...,valn)
```

```sql
update tablename set field1=val1, field2=val2,..., fieldn=valn
update t1,t2,...,tn set t1.field1=expr1, ... , tn.field=exprn
update a inner join b on a.id=b.id set a.name=b.name where b.property is null
```

```sql
delete from tablename where...
delete a from a left join b on a.id = b.id where b.property is null
```

针对`update a join b`与`delete a from a join b`，可以理解为先使用`select a join b`生成了中间表，之后再执行where过滤中间表数据，剩下的内容则为需要`update`或者`delete`的部分了

## 8. 约束
约束实际上就是表中数据的限制条件，表在设计的时候加入约束的目的就是为了保证表中的记录完整和有效
比如name字段中要让其用户名不重复，这就需要添加约束。或者必须注册的时候需要添加邮箱等

约束种类：
1. 非空约束（not null）
2. 唯一性约束（unique）
3. 主键约束（primary key）
4. 外键约束（foreign key）

```sql
mysql> create table t_user(
-> id int(10),
-> name varchar(32) not null
-> );

mysql> create table t_user(
-> id int(10),
-> name varchar(32) not null,
-> email varchar(128) unique
-> );

mysql> create table t_user(
-> id int(10),
-> name varchar(32) not null,
-> email varchar(128），
-> unique(email)
-> );

mysql> create table t_user(
-> id int(10),
-> name varchar(32) not null,
-> email varchar(128),
-> unique(name,email)
-> );

mysql> create table t_user(
-> id int(10) primary key,
-> name varchar(32)
-> );

mysql> create table t_user(
-> id int(10),
-> name varchar(32) not null,
-> email varchar(128) unique,
-> primary key(id,name)
-> );

mysql> create table t_user(
-> id int(10) primary key auto_increment,
-> name varchar(32) not null
-> );

mysql> create table t_class(
-> cno int(10) primary key,
-> cname varchar(128) not null unique
-> );
mysql> create table t_student(
-> sno int(10) primary key auto_increment,
-> sname varchar(32) not null,
-> classno int(3),
-> foreign key(classno) references t_class(cno)
-> );
```