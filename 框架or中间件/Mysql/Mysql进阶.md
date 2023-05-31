<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\进阶大纲.png" style="width:700px;height:250px;" />

## 1. Mysql逻辑架构
<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\Mysql逻辑架构.png" style="width:700px;height:600px;" />

### SQL查询流程
<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\SQL查询流程.png" style="width:700px;height:500px;" />

1. 客户端发送一条SQL查询给服务器
2. 服务器先检查查询缓存，如果命中了缓存，则立刻返回存储在缓存中的结果，否则进入下一阶段
3. 服务器端进行SQL解析、预处理，再由优化器生成对应的执行计划
4. Mysql根据优化器生成的执行计划，调用存储引擎的API来执行查询
5. 将结果返回给客户端

可使用`show profiles`、`show profile for query 1`查询SQL执行过程

#### 客户端连接池
部署在服务器中的应用存在多个线程都获取各自的数据库连接访问数据库，避免每次需要访问都创建一个新的数据库连接使用，使用完毕又销毁的效率低下情况。客户端存在连接池，而Mysql的连接器中也存在连接池

#### 连接器
接受客户端的连接，Mysql会通过连接器管理连接、认证用户、获取权限
连接器建立连接时会根据用户账号，进行账号认证以及权限认证（到用户库表以及权限库表认证），因此在连接建立成功后修改连接对应的库表权限是不影响该连接的，只有在重新建立连接后才生效

至于连接器中的连接存在生命时长（默认8小时，超时即断开）以及状态，`show processlist`可查看连接状态：
- `sleep`：空闲，等待客户端发送新请求
- `query`：正在查询或者发送结果
- `locked`：连接正在等待锁释放
- `analyzing and statistics`：正在生成执行计划（SQL经过分析器与优化器）
- `copying to tmp table`：执行查询并把结果集复制到临时表中
- `sorting result`：对结果集进行排序
- `sending data`：生成并返回结果集

#### 查询缓存
解析查询语句前会优先到查询缓存中查找该SQL是否能命中缓存，缓存以key-value形式存在。在命中缓存后会检查用户的相关库表权限决定是否返回缓存结果集
当然，查询缓存非常容易失效，因为涉及缓存的相关表只要经过增删改的操作，相应缓存都会失效，所以缓存命中率非常低

#### 分析器
分析器会对SQL进行词法语法分析，根据语法规则识别得出语法树，检查SQL语句是否合法、SQL语句中的操作类型、操作对象、过滤条件

#### 优化器
针对一条查询语句生成自认为执行成本最小的执行计划，经过优化的查询语句与原语句不一定完全相同

#### 执行器与存储引擎
执行计划交给执行器后，执行器根据执行计划的步骤调用存储引擎的接口API
而表的引擎定义不一致，存储表数据的磁盘文件结构或者内存数据结构也是不一样的，只是存储引擎暴露出来的统一接口屏蔽掉这些区别
在调用存储引擎接口时，存储引擎会查询或更新内容缓存数据、磁盘数据，因此存储引擎与底层数据存取的物理实现相关

## 2. Mysql引擎
我们平时下载电影是不是同一个电影有mp4、rmvb、avi等格式，这些不同的格式的同一个电影清晰度、占用磁盘的空间可能会不同，但是它们的内容都是一样的
存储引擎和上述所说的类似，不同的存储引擎存入到数据中存储的形式不同，所以导致占用空间、性能等不同，但是给用户展现的数据都是相同的

使用`show engines`查看当前数据库可以支持的存储引擎
<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\Mysql支持的存储引擎.png" style="width:700px;height:180px;" />

1. MyISAM
每个MyISAM在磁盘上存储成三个文件。这三个文件第一个文件的名字以表的名字开始，扩展名指出文件类型。.frm文件存储表定义。数据文件的扩展名为.MYD (MYData)。索引文件的扩展名是.MYI (MYIndex)
-   不支持事务
-   表级锁定（更新时锁定整个表）
-   读写互相阻塞（写入时阻塞读入、读时阻塞写入；但是读不会互相阻塞）
-   只会缓存索引（通过key_buffer_size缓存索引，但是不会缓存数据）
-   读取速度快
-   不支持外键，但支持全文索引
2. InnoDB
InnoDB支持事务（ACID）以及外键支持，InnoDB引擎的表在磁盘上保留了一个frm扩展名的文件
-   支持事务，支持4个事务隔离级别
-   行级锁定（更新时锁定当前行）
-   读写阻塞与事务隔离级别相关
-   既能缓存索引又能缓存数据
-   支持外键
-   InnoDB更消耗资源，读取速度没有MyISAM快

InnoDB与MyISAM主要区别：
1.  InnoDB支持外键和事务，MyISAM不支持外键和事务；对于InnoDB每一条SQL语句都默认封装成事务，自带提交
2.  InnoDB是聚焦索引，MyISAM是非聚焦索引。 InnoDB是聚集索引，数据文件是和索引绑在一起的，必须要有主键，通过主键索引效率很高。但是辅助索引需要两次查询，先查询到主键，然后再通过主键查询到数据。因此，主键不应该过大，因为主键太大，其他索引也都会很大。而MyISAM是非聚集索引，数据文件是分离的，索引保存的是数据文件的指针。主键索引和辅助索引是独立的
3.  InnoDB不保存表的具体行数，执行select count(\*) from table时需要全表扫描。而MyISAM用一个变量保存了整个表的行数，执行上述语句时只需要读出该变量即可，速度很快
4.  InnoDB不支持全文索引，而MyISAM支持全文索引，查询效率上MyISAM要高

修改Mysql默认引擎方式
1. 在mysql配置文件ini中更改设置，添加`default-storage-engine=INNODB/MYISAM`
2. 在创建表时指定引擎，`create table if not exists tablename (id int) engine = InnoDB`

## 3. 系统文件存储结构
`datadir`是Mysql的数据目录位置，使用`show variables like 'datadir'`查看，在该文件目录下存在以数据库库名为文件夹名的文件夹

Mysql存在默认数据库：
- `mysql`：这个数据库的核心，它存储了MySQL的用户账户和权限信息，一些存储过程、事件的定义信息，一些运行过程中产生的日志信息，一些帮助信息以及时区信息等
- `information_schema`：这个数据库保存着MySQL服务器所有其他数据库的信息，比如表、视图、触发器、列、索引、锁、事务等等。这些信息并不是真实的用户数据，而是一些描述性信息，也称之为元数据
- `performance_schema`：这个数据库主要保存MySQL服务器运行过程中的一些状态信息，包括统计最近执行了哪些语句，在执行过程的每个阶段都花费了多长时间，内存的使用情况等等信息
- `sys`：这个数据库主要是通过视图的形式把`information_schema`和`performance_schema`结合起来，让程序员可以更方便的了解MySQL服务器的一些性能信息

在Mysql数据文件目录下可看到：
- `db.opt`：记录这个数据库默认使用的字符集和校验规则。
- `ib_logfile0、ib_logfile1`：Redo log 日志文件。
- `.frm`：存储表相关的元数据信息，包含表结构的定义信息等，每一张表都会有一个frm文件与之对应。
	InnoDB数据文件：
	- `.ibd`：使用**独享表空间**存储表数据和索引信息，一张表对应一个ibd文件
	- `.ibdata`：使用**共享表空间**存储表数据和索引信息，所有表共同使用一个或者多个ibdata文件
	MyISAM数据文件：
	- `.MYD`：主要用来存储表数据信息
	- `.MYI`：主要用来存储表数据文件中任务索引的数据树

Mysql配置信息文件：`my.cnf`、`my.ini`

## 4. InnoDB逻辑存储结构（表空间内存储结构）
<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\InnoDB逻辑存储结构.png" style="width:700px;height:550px;" />

### 1. 表空间
表空间可以看做是InnoDB存储引擎逻辑结构的最高层，所有的数据都存放在表空间中。在默认情况下InnoDB存储引擎有一个共享表空间`ibdata1`，所有数据都存放在这个表空间内

如果启用了参数`innodb_file_per_table`，则每张表内的数据可以单独放到一个表空间内。需要注意的是，这些单独的表空间文件仅存储该表的数据、索引和插入缓冲Bitmap等信息，其余信息还是存放在共享表空间中，例如 undo日志、插入缓冲索引页、系统事务信息、二次写缓冲等

因此即使在启用了参数`innodb_file_per_table`之后，共享表空间的大小还是会不断地增加，例如事务中写入了undo日志，就算回滚了，共享表空间的大小也不会缩小。但是会判断这些undo信息是否还需要，不需要的话，就会将这些空间标记为可用空间，供下次重复使用

### 2. 行
InnoDB的数据是按行进行存放的，每个页存放的行记录最多允许存放`16KB / 2 -200`行的记录，即`7992`行记录
每行记录根据不同的行格式、不同的数据类型，会有不同的存储方式。每行除了记录我们保存的数据之外，还可能会记录事务ID（DB_TRX_ID），回滚指针（DB_ROLL_PTR）等

### 3. 页
`页（Page）`是 InnoDB 磁盘管理的最小单位，默认每个页的大小为`16KB`，也就是最多能保证16KB的连续存储空间
InnoDB 将数据划分为若干个页，以页作为磁盘和内存之间交互的基本单位，也就是一次最少从磁盘中读取一页16KB的内容到内存中，一次最少把内存中的16KB内容刷新到磁盘中

### 4. 索引组织表
在InnoDB中，表都是根据主键顺序存放数据的，这种存储方式的表称为索引组织表
在InnoDB表中，每张表都有个主键，如果在创建表时没有显式地定义主键，则InnoDB会按如下方式选择或创建主键：
-   首先判断表中是否有非空的唯一索引，如果有，则该列即为主键。当表中有多个非空唯一索引时，将选择建表时第一个定义的非空唯一索引为主键
-   如果不符合上述条件，InnoDB会自动创建一个名为`row_id`的6字节的隐藏列作为主键

为了能快速的从磁盘中检索出数据，InnoDB采用`B+树`结构来组织数据
<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\InnoDB中B+树组织结构.png" style="width:700px;height:400px;" />

`B+树`是多层的，`B+树`每一层中的页都会形成一个双向链表。在这棵树中，只有最底层的叶节点才存储数据，这些数据是按主键顺序存储的。上层的非叶子节点存储的则是索引目录，索引目录则根据主键区间划分了多个页和层级，这样就可以通过类似二分法的方式快速找到某条数据所在的页，然后通过主键定位到具体的某条数据

#### 5. 区
在默认情况下，InnoDB存储引擎页的大小为`16KB`，表空间中的页就太多了。为了更好的管理这些页，InnoDB 将**物理位置上连续的64个页划为一个区**，任何情况下，每个区的大小都为`1MB`

B+树中每一层都是通过双向链表连接起来的，如果是以页为单位来分配存储空间，本来链表中相邻的两个页之间的物理位置就可能离得非常远，那么磁盘查询时就会有大量的随机I/O，随机I/O是非常慢的。所以应该尽量**让链表中相邻的页的物理位置也相邻**，这样可以消除很多的随机I/O，使用顺序I/O，尤其是在进行范围查询的时候
所以在表中数据量大的时候，为某个索引分配空间的时候就不再按照页为单位分配了，而是按照区为单位分配，甚至在表中的数据非常多的时候，可以一次性分配多个连续的区（因为区中的页也是物理连续的）

不论是系统表空间还是独立表空间，都可以看成是由若干个区组成的，每个区64个页，然后每256个区又被划分成一组

<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\InnoDB的256个区组成组的某些固定结构.png" style="width:700px;height:600px;" />

#### 6. 段
从前面B+树的结构知道，B+树分为叶子节点和非叶子节点，最底层的叶子节点才存储了数据，非叶子节点是索引目录。如果将叶子节点页和非叶子节点页混合在一起存储，那在检索数据的时候同样也会有大量的随机I/O

所以 InnoDB 又提出了段的概念，常见的段有数据段、索引段、回滚段等。段是一个逻辑上的概念，并不对应表空间中某一个连续的物理区域，它由若干个完整的区组成（还会包含一些碎片页），不同的段不能使用同一个区

存放叶子节点的区的集合就是**数据段**，存放非叶子节点的区的集合就是**索引段**。也就是说一个索引会生成2个段，一个叶子节点段（数据段），一个非叶子节点段（索引段）

## 5. InnoDB行记录格式
InnoDB支持四种行记录格式：
1. Compact
2. Redundant
3. Dynamic
4. Compressed

可使用`show variables like 'innodb_default_row_format'`查看设置的行格式

<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\行格式中的记录头信息.png" style="width:700px;height:100px;" />

- `delete_mark`：标记当前记录是否被删除
- `min_rec_mask`：B+树的每层非叶子节点中的最小记录都会添加该标记，并设置为1，否则为0
- `n_owned`
- `heap_no`：记录在页中的位置
- `record_type`：0表示普通记录，1表示B+树非叶子节点记录，2表示最小记录，3表示最大记录，1xx表示保留
- `next_record`：当前记录真实数据到下一记录的真实数据的地址偏移量

### 1. Compact行记录格式
<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\Compact行记录格式.png" style="width:700px;height:150px;" />

- 变长字段长度列表
	**变长字段长度列表**按照非null变长列的逆序记录变长字段真实占用字节数；若字段占用字节数过多导致单页无法完全存储，则会把一部分数据存储在**溢出页**中
- NULL标志位
	将值为NULL的列标志出来，减少存储空间浪费；每一列对应一个二进制位，为1代表该列为NULL，为0代表该列不为NULL
- 记录头信息
- 记录的真实数据

### 2. Redundant行记录格式
<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\Redundant行记录格式.png" style="width:700px;height:150px;" />

- 字段长度偏移列表
	将字段长度偏移列表中的各个列对应的偏移量的第一个比特位作为是否为NULL的依据
- 记录头信息
- 记录的真实数据

### 3. Compressed和Dynamic行记录格式
Compressed 和 Dynamic 行记录格式与 Compact 行记录格式是类似的，只不过在处理行溢出数据时有些区别
Compact行记录溢出：
<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\Compact行记录格式溢出.png" style="width:700px;height:250px;" />

Compressed、Dynamic行记录溢出：
<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\Compressed和Dynamic行记录格式溢出.png" style="width:700px;height:300px;" />

## 6. InnoDB页结构
<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\InnoDB页结构.png" style="width:700px;height:700px;" />

- `file header`：记录页的头信息，其中包括
	- `fil_page_offset`：页号
	- `fil_page_prev`：与逻辑上的前一页链接起来
	- `fil_page_next`：与逻辑上的后一页链接起来
	- `fil_page_type`：当前页的类型
- `page header`：记录数据页状态信息，其中包括
	- `page_n_dir_slots`：页中的记录会按主键顺序分为多个组，每个组会对应到一个槽（Slot），该项记录页中的槽数量
	- `page_heap_top`：free space地址，能快速从free space分配空间到user records
	- `page_n_heap`：页中记录数，包括最小最大记录以及被删除记录
	- `page_free`：已删除的记录会通过`next_record`连成一个单链表，这个单链表中的记录空间可以被重新利用，`page_free`指向第一个标记为删除的记录地址
	- `page_garbage`：被删除的记录数占用总字节
	- `page_n_recs`：页中记录数量，不包括最小最大记录以及被删除记录
- `infimum与supremum`
	InnoDB每个数据页中有两个虚拟的行记录，用来限定记录的边界。`Infimum记录`是比该页中任何主键值都要小的记录，`Supremum记录`是该页中任何主键值都要大的记录。这两个记录在页创建时被建立，并且在任何情况下不会被删除
	
	`Infimum`记录头的`next_record`指向该页主键最小的记录，该页主键最大的记录的`next_record`则指向 `Supremum`，`Infimum`和`Supremum`就构成了记录的边界
- `user records与free space`：实际存储行记录与空闲行记录的空间
- `page directory`：页目录，存在复杂的生成逻辑，对行记录分组分槽，方便提升取行记录的效率
- `file trailer`：用于校验页的完整性，通过使用`file trailer`与`file header`中部分数据比较保证页的完整


## 7. 索引
### 1. 索引基本操作
```sql
create table user(
    id varchar(11) primary key, -- 主键索引 --
    name varchar(20),
    age int
    index name_index (name) -- 普通索引 --
    -- index name_age_index (name,age) 联合索引 --
    -- unique index name_index (name) 唯一索引 --
)
show index from user;

-- 单列索引 --
create index name_index on user(name);

-- 唯一索引 --
create unique index age_index on user(age);

-- 复合索引 --
create index name_age_index on user(name,age);
```

### 2. 索引类型
- 聚簇索引：每个InnoDB表都有一个聚簇索引，聚簇索引使用B+树构建，叶子节点的data阈存储的是整行记录。一般情况下，聚簇索引等同于主键索引
- 辅助索引：InnoDB在创建表时，默认会创建一个主键的聚簇索引，而除此之外的其它索引都属于辅助索引，也被称为二级索引或非聚簇索引，二级索引也会创建B+树，而且结构与聚簇索引十分相似，唯一的区别是二级索引的B+树叶子节点存储的是**索引列+主键值**，而非完整的用户记录，**需要回表**

### 3. 使用索引注意事项
- 全值匹配：查询条件中的列与索引中的列一致
- 匹配最左前缀：对于联合索引，可以只使用左边的部分列，可以不用包含全部联合索引中的列，但只能是左边连续的列
- 匹配列前缀：只匹配某一列的值的开头部分，比如`like 'A%'`
- 匹配范围值：对联合索引的多个列进行范围查找，只会在对索引最左边的第一个列进行范围查找时用上索引
- 精确匹配某一列并范围匹配另一列：在联合索引中，最左列精确匹配，随后的一列用上索引
- 排序：根据联合索引中的索引顺序，使用order by进行排序

### 4. 索引数据结构类型
Mysql中不同的存储引擎支持的索引都是不一致的，即使都支持同一种索引但可能底层实现都不一样

| 索引类型 | InnoDB引擎 | MyISAM引擎 | Memory引擎 |
| ----- | ----- | ----- | ----- |
| B+Tree索引 | Y | Y | Y |
| Hash索引 | N | N | Y |
| R-Tree索引 | N | Y | N |
| Full-Text索引 | N | Y | N |

#### 1. Hash表
#### 2. 二叉树
#### 3. AVL平衡二叉树
#### 4. B树
#### 5. B+树

### 5. InnoDB索引
相比MyISAM，索引文件和数据文件是分离的，其表数据文件本身就是按照B+Tree组织的一个索引结构，树的叶节点data域保存了完整的数据记录。这个索引的Key是数据表的主键，因此InnoDB表数据文件本身就是主索引。这被称为“**聚簇索引（聚集索引）**”。而其余的索引都作为辅助索引，辅助索引的data域存储相应记录主键的值而不是地址，这也是和MyISAM不同的地方

<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\InnoDB索引原理.png" style="width:700px;height:400px;" />

- 在根据主索引搜索时，直接找到Key所在的节点即可取出数据
- 在根据辅助索引查找时，则需要先取出主键的值，再走一遍主索引
- 因此在设计表的时候，不建议使用过长的字段作为主键，也不建议使用非单调的字段作为主键，这样会造成主索引频繁分裂

### 6. MyISAM索引
B+Tree叶节点的data域存放的是数据记录的地址。在索引检索的时候，首先按照B+Tree搜索算法搜索索引，如果指定的Key存在，则根据data域中磁盘地址到磁盘中寻址定位到对应的磁盘块，然后读取相应的数据记录，这被称为“非聚簇索引

<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\MyISAM索引原理.png" style="width:700px;height:600px;" />

- 用户数据按照记录的插入顺序单独存储在一个文件中，称之为数据文件，也就是`.MYD`为后缀的文件。这个文件并不划分数据页，所有记录都按照插入顺序插入就行了，然后通过每行数据的物理地址来快速访问到一条记录
- 索引信息则另外存储到一个单独的索引文件中，就是`.MYI`为后缀的文件。MyISAM 会单独为表的主键创建一个索引，只不过在索引的叶子节点中存储的不是完整的用户记录，而是主键值 + 物理地址的组合。也就是先通过索引找到行对应的物理地址，再通过物理地址去找对应的记录

## 8. 调优
### 1. 性能分析
#### 1. 慢查询
**Mysql的慢查询，全名慢查询日志，是Mysql提供的一种日志记录，用来记录在Mysql中响应时间超过阈值的语句**

慢查询相关参数查看：
```sql
mysql> show variables like 'slow_query%';
+---------------------------+----------------------------------+
| Variable_name             | Value                            |
+---------------------------+----------------------------------+
| slow_query_log            | OFF                              |
| slow_query_log_file       | /mysql/data/localhost-slow.log   |
+---------------------------+----------------------------------+

mysql> show variables like 'long_query_time';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
```

慢查询设置方式：
1. 配置文件开启慢查询
```
[mysqlld]
long_query_time=10

5.0 5.1版本对应配置
log-slow-queries="mysql_slow_query.log"

5.5以上版本对应配置
slow-query-log=On
slow_query_log_file="mysql_slow_query.log"
```
2. 设置Mysql全局变量
```sql
set global slow_query_log=ON;
set global slow_query_log_file="mysql_slow_query.log"
set global long_query_time = 3600;
set global log_querise_not_using_indexes = ON;
```

#### 2. 慢查询分析工具：mysqldumpslow
```sheet
mysqldumpslow -t 10 /var/lib/mysql/mysql-slow.log
对慢查询日志进行分析
```

#### 3. SQL执行成本：show profile
```sql
mysql> show variables like 'profiling';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| profiling     | OFF   |
+---------------+-------+
mysql> set profiling=on;
```

`show profiles`是mysql提供的可以用来分析当前会话中sql语句执行的资源消耗情况的工具，可用于sql调优的测量。默认情况下处于关闭状态，并保存最近15次的运行结果
```sql
mysql> show variables like 'profiling';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| profiling     | OFF   |
+---------------+-------+

mysql> set profiling=1;

mysql> show profiles;
+----------+------------+---------------------------------+
| Query_ID | Duration   | Query                           |
+----------+------------+---------------------------------+
|        1 | 0.00127100 | show variables like 'profiling' |
|        2 | 0.00025800 | select * from api_config        |
|        3 | 0.00023650 | select * from api_header        |
|        4 | 0.00021625 | select * from api_param_config  |
+----------+------------+---------------------------------+

mysql> show profile for query 2;
+----------------------+----------+
| Status               | Duration |
+----------------------+----------+
| starting             | 0.000055 |
| checking permissions | 0.000003 |
| Opening tables       | 0.000025 |
| init                 | 0.000005 |
| System lock          | 0.000008 |
| optimizing           | 0.000002 |
| statistics           | 0.000013 |
| preparing            | 0.000006 |
| executing            | 0.000002 |
| Sending data         | 0.000045 |
| end                  | 0.000002 |
| query end            | 0.000007 |
| closing tables       | 0.000004 |
| freeing items        | 0.000073 |
| cleaning up          | 0.000009 |
+----------------------+----------+
```

`show profile`的常用查询参数：
- `all`：显示所有开销
- `block io`：IO开销
- `context switches`：上下文切换开销
- `cpu`：CPU开销
- `ipc`：发送接收开销
- `memory`：内存开销
- `page faults`：页面错误开销
- `source`
- `swaps`：交换次数开销

#### 4. 统计SQL查询成本：last_query_cost
MySQL中可以通过`show status like 'last_query_cost'`来查看查上一个查询的代价，而且它是`io_cost`和`cpu_cost`的开销总和，它通常也是我们评价一个查询的执行效率的一个常用指标

#### 5. 分析查询语句：explain
```sql
mysql> explain select * from servers;
+----+-------------+---------+------+---------------+------+---------+------+------+-------+
| id | select_type | table   | type | possible_keys | key  | key_len | ref  | rows | Extra |
+----+-------------+---------+------+---------------+------+---------+------+------+-------+
|  1 | SIMPLE      | servers | ALL  | NULL          | NULL | NULL    | NULL |    1 | NULL  |
+----+-------------+---------+------+---------------+------+---------+------+------+-------+
1 row in set (0.03 sec)
```

- `id`
	1. id相同时，执行顺序由上至下
	2. 若为子查询，id序号会递增，id值越大优先级越高，越会先被执行
	3. id相同则被认为是同一组，执行顺序从上至下；所有组中，id值越大执行优先级越高
- `select_type`
	1. `SIMPLE`：简单select不使用union或子查询
	2. `PRIMARY`：查询中包含复杂的子部分，最外层select会被标记为`PRIMARY`
	3. `UNION`：`UNION`中的第二个或后面的select语句
	4. `DEPENDENT UNION`：`UNION`中的第二个或后面的SELECT语句，取决于外面的查询
	5. `UNION RESULT`：`UNION`的结果
	6. `SUBQUERY`：子查询中第一个select
	7. `DEPENDENT SUBQUERY`：子查询中的第一个SELECT，取决于外面的查询
	8. `DERIVED`：派生表的SELECT, FROM子句的子查询，`select id from (select id from a) as tmp`
	9. `UNCACHEABLE SUBQUERY`：一个子查询的结果不能被缓存，必须重新评估外链接的第一行
- `table`：显示这一行的数据是关于哪张表的，有时不是真实的表名字
- `type`：表示MySQL在表中找到所需行的方式，又称“访问类型”，性能从最优到最差的顺序：`system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL`
	1. `ALL`：Full Table Scan， MySQL将遍历全表以找到匹配的行
	2. `index`：Full Index Scan，index与ALL区别为index类型只遍历索引树
	3. `range`：只检索给定范围的行，使用一个索引来选择行，`select * from a where id > 2`
	4. `ref`：不使用唯一索引，而是使用普通索引或者唯一索引的部分前缀，索引要和某个值相比较，可能会找到多个符合条件的行，`select * from film where name = "film1"`，其中name为普通索引
	5. `eq_ref`：类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用`primary key`或者`unique key`作为关联条件，`select * from film_actor left join film on film_actor.film_id = film.id`
	6. `const`、`system`：当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。如将主键置于where列表中，MySQL就能将该查询转换为一个常量，system是const类型的特例，当查询的表只有一行的情况下，使用system，`select * from (select * from a where id = 1) tmp`
	7. `NULL`：MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成`select min(id) from a`
- `possible_keys`：这一列显示查询可能使用哪些索引来查找
- `key`：这一列显示mysql实际采用哪个索引来优化对该表的访问，如果想强制mysql使用或忽视possible_keys列中的索引，在查询中使用`force index`、`ignore index`
- `key_len`：这一列显示了mysql在索引里实际使用的字节数，可以通过这个判断具体使用了联合索引中的哪个索引
- `ref`：这一列显示了在key列记录的索引中，表查找值所用到的列或常量
- `rows`：mysql估计要读取并检测的行数
- `Extra`：展示的是额外信息，如`distinct`、`using index`、`using where`、`using temporary`、`using filesort`

### 2. SQL优化
select语句语法顺序：
1. `select`
2. `distinct <select_list>`
3. `from <left_table>`
4. `<join_type> join <right_table>`
5. `on <join_condition>`
6. `where <where_condition>`
7. `group by <group_by_list>`
8. `having <having_condition>`
9. `order by <order_by_condition>`
10. `limit <limit_number>`

select语句执行顺序：
1. `from`
	<表名>，选取表，将多个表数据通过笛卡尔积变成一个表
2. `on`
	<筛选条件>，对笛卡尔积的虚表进行筛选
3. `join`
	<join表>，指定join，用于添加数据到on之后的虚表中，例如left join会将左表的剩余数据添加到虚表中
4. `where`
	<where条件>，对上述虚表进行筛选
5. `group by`
	<分组条件>，分组
6. `having`
	<分组筛选>，对分组后的结果进行聚合筛选
7. `select`
	<返回数据列表>
8. `distinct`
	数据去重
9. `order by`
	<排序条件>
10. `limit`
	<行数限制>

#### SQL优化策略
- 模糊查询优化
	1. 使用类似`instr(str,substr) > 0`的Mysql内置函数匹配
	2. 使用`like`尽量遵循匹配最左前缀走索引查询
	3. 创建`fulltext`全文索引，用`match against`检索
- 关联查询优化
	1. 小表驱动大表
	2. 尽可能减少join语句中内层循环次数，可以对被驱动表的on连结条件字段创建索引
	3. `inner join`会自动优化成小表驱动大表
- 子查询优化
	1. 使用`exists`代替`in`
	2. 若存在`exists`字句的外层查询的主表数据量大，可以将sql改为join，并小表驱动大表
	3. 尽量不使用`not in`或`not exists`，可以改写为`join ... on ... where ... is null`
- 排序优化：mysql中存在两种排序方式，`filesort`和`index`排序，`filesort`一般在内存中排序，待排结果较大存在产生临时文件IO到磁盘排序的可能，效率比`index`低
	1. 对需要排序的列添加索引，可以跳过排序过程，避免`filesort`，优化排序速度
	2. 对于无法使用`index`排序的，可以使用`filesort`调优
		- 双路排序：对需要排序的记录生成`<sort_key,rowid>`的元数据进行排序，该元数据仅包含排序字段和rowid。排序完成后只有按字段排序的`rowid`，因此还需要通过`rowid`进行**回表操作获取所需要的列的值**，可能会导致**大量的随机IO读消耗**
		- 单路排序：对需要排序的记录生成`<sort_key,additional_fields>`的元数据，该元数据包含排序字段和需要返回的所有列。排序完后不需要回表，但是元数据要比第一种方法长得多，**需要更多的空间用于排序**
- `group by`优化：和对`order by`的优化策略一样，尽量使用索引以及符合匹配最左前缀
- 尽量避免使用`in`和`not in`，会导致走全表扫描
	1. 若过滤条件为连续数值，可以使用`between`
	2. 如果是子查询，使用`exists`代替，如`select * from A where exists (select * from B where B.id = A.id)`
- 尽量避免使用`or`，会导致走全表扫描，可以使用`union`代替，需要去重则使用`union all`
- 尽量避免`null`值的判断，会导致走全表扫描，可以设置字段默认值为常量，进行常量判断
- 尽量避免where条件中等号左侧进行表达式、函数操作，会导致走全表扫描
- 尽量避免对索引列进行类似`<>`或`!=`的不等于条件判断
- 避免隐式类型转换
- 使用`hint`优化语句
	1. `use index`，如`select col from table use index (index1,index2,...) where...`
	2. `ignore index`，如`select col from table ignore index (index1,index2,...) where...`
	3. `force index`，如`select col from table force index (index1,index2,...) where...`
- 大批量插入或更新时，把多条插入或更新sql合并为一条
- 对于复杂查询使用中间临时表缓存数据
- 使用`truncate`代替`delete`
- 分页可以使用索引的方式提高分页效率，如先检索出该页记录的所有索引，再根据这若干索引查找记录

### 3. 数据库调优
- 冷热数据分离，实际为垂直分表的一种
- 增加中间表，也为垂直分表的一种，把经常需要通过连结获取的字段放到一个表中
- 增加冗余字段
- 优化数据类型，优先选择符合存储需要的最小的数据类型
	1. 对证书类型进行优化
	2. 对既可以使用文本类型也可以使用整数类型的字段，要使用整数类型
	3. 避免使用`text`、`blob`数据类型
	4. 避免使用`enum`
	5. 使用`timestamp`存储时间
	6. 使用`decimal`存储浮点数，不会存在精度丢失
- 大表优化
	1. 限制查询范围，查询只按照某个维度进行，返回部分数据，避免大数据检索
	2. 读写分离
		<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\读写分离模式.png" style="width:700px;height:300px;" />
	3. 分库分表
		<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\分库分表模式1.png" style="width:700px;height:400px;" />
		<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\分库分表模式2.png" style="width:700px;height:500px;" />
		<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\分库分表模式3.png" style="width:700px;height:600px;" />

## 9. 事务
事务必须满足的四个条件：
- **原子性**，`atomicity`
	一个事务`transaction`中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚`Rollback`到事务开始前的状态
- **一致性**，`consistency`
	在事务开始之前和事务结束以后，数据库的完整性没有被破坏
- **隔离性**，`lsolation`
	数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离包括读未提交`Read uncommitted`、读提交`read committed`、可重复读`repeatable read`和串行化`Serializable`
- **持久性**，`durability`
	事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失

事务的状态：
- **活动**`active`：事务对应的操作正在执行过程中
- **部分提交**`partially committed`：事务中最后一个操作执行完成，但由于操作都在内存中执行，所造成的影响并没有刷新到磁盘中
- **失败**`failed`：事务处于活动的或者部分提交的状态时，可能遇到错误或人为停止事务而无法继续执行，此时的状态为失败
- **中止**`aborted`：在事务处于失败状态后，需要对事务操作进行回滚，回滚后的状态成为中止
- **提交**`committed`：当事务操作都同步到磁盘上后，事务处于提交
<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\事务状态.jpg" style="width:700px;height:400px;" />

事务隔离级别：

| 隔离级别 | 定义 | 脏读`dirty read` | 不可重复读`nonrepeatable read` | 幻读`phantom read` |
| ----- | ----- | ----- | ----- | ----- |
| 未提交读`read uncommitted` | 在读未提交隔离级别下，事务A可以读取到事务B修改过但未提交的数据 | 可能 | 可能 | 可能 |
| 已提交读`read committed` | 在读已提交隔离级别下，事务B只能在事务A修改过并且已提交后才能读取到事务B修改的数据；读已提交隔离级别解决了脏读的问题，但可能发生不可重复读和幻读问题，一般很少使用此隔离级别 | 不可能 | 可能 | 可能 |
| 可重复读`repeatable read` | 在可重复读隔离级别下，事务B只能在事务A修改过数据并提交后，自己也提交事务后，才能读取到事务B修改的数据；可重复读隔离级别解决了脏读和不可重复读的问题，但可能发生幻读问题 | 不可能 | 不可能 | 可能 |
| 可串行化`serializable` | 各种问题（脏读、不可重复读、幻读）都不会发生，通过加锁实现（读锁和写锁） | 不可能 | 不可能 | 不可能 |

- 脏读
	指当一个事务正在访问数据，并且对数据进行了修改，而这种修改还没有提交到数据库中，这时，另外一个事务也访问这个数据，然后使用了这个数据
- 不可重复读
	指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的的数据可能是不一样的。这样就发生了在一个事务内两次读到的数据是不一样的
- 幻读
	第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象发生了幻觉一样

## 10. 锁
### 1. 行锁与表锁
#### 1. 行锁
行锁是作用在索引上的，由存储引擎实现
- 对于聚簇索引，会在B+树叶子节点的索引上加上锁，被加上锁的包括索引列以及数据行
- 对于辅助索引，因为需要回表，会先后在辅助索引以及聚簇索引上加上锁，即辅助索引B+树上叶子节点以及聚簇索引B+树上的叶子节点加上锁

行锁：加锁过程开销大，速度慢，会出现死锁；锁粒度小，锁冲突几率小，并发度高
- 最大程度的支持并发，同时也带来了最大的锁开销
- 在InnoDB中，除单个SQL组成的事务外，锁是逐步获得的，这就决定了在InnoDB中发生死锁是可能的
- 行级锁更适合于有大量按索引条件并发更新少量不同数据，同时又有并发查询的应用

#### 2. 表锁
当操作语句中不存在索引或者没走索引进行操作时，InnoDB会把涉及到的表整张锁住，即表锁，由Mysql服务层实现

表锁使用的是一次性锁技术，也就是说，在会话开始的地方使用`lock`命令将后续需要用到的表都加上锁，在表释放前，只能访问这些加锁的表，不能访问其他表，直到最后通过`unlock tables`释放所有表锁。而会话在执行事务前会或者执行`lock table`命令前，释放原先持有的所有表锁

表锁：加锁过程开销小，速度快，不会出现死锁；锁粒度大，锁冲突几率大，并发度低
- 一般在执行DDL语句时会对整个表进行加锁
- 如果对InnoDB的表使用行锁，被锁定字段不是主键，也没有针对它建立索引的话，那么将会锁整张表
- 表级锁更适合于以查询为主，并发用户少，只有少量按索引条件更新数据的应用

#### 3. MyISAM表锁（读写锁）
- 表共享读锁：不会阻塞其他线程对同一个表的读操作请求，但会阻塞其他线程的写操作请求，执行优先级比写锁低
- 表独占写锁：一旦表被加上独占写锁，那么无论其他线程是读操作还是写操作，都会被阻塞，但获取写锁的线程还能进行读操作，执行优先级比读锁高

因为MyISAM引擎只支持表锁，而表锁中的写锁执行队列永远比读锁执行队列优先级高，因此大量更新操作会持续获取写锁进行写操作，导致读操作被阻塞；而一些耗时过长的查询也会持续占用读锁，阻塞写操作获取写锁，上述情况均为锁冲突。因此使用表锁应当尽量避免大量写操作、长时间的读操作

MyISAM执行查询语句时，会自动给涉及到的所有表加读锁；在执行更新语句时，会自动给涉及到的所有表加写锁。无论是获取表的读锁还是写锁，获取表锁的过程都是原子性的，即不会出现死锁

```sql
lock tables table_name [read | write]
```

#### 4. InnoDB行锁与表锁
InnoDB中的行锁
- 共享锁：加了锁的记录，所有事务都能去读取但不能修改，同时阻止其他事务获得相同数据集的排他锁
- 排他锁：允许已经获得排他锁的事务去更新数据，阻止其他事务取得相同数据集的共享读锁和排他写锁

InnoDB中的表锁，意向锁：由于表锁与行锁之间会互相冲突，因此引入意向锁检测和避免这种冲突。意向锁分为**读意向锁IS**与**写意向锁IX**。**当事务中存在向表中记录添加行锁时，会首先在表上添加意向锁，意向锁之间不会冲突也不与行锁冲突，只会阻塞表读写锁**

表锁与意向锁的冲突与兼容情况

|      | 表写锁 | 写意向锁 | 表读锁 | 读意向锁 |
| ----- | ----- | ----- | ----- | ----- |
| 表写锁 | 冲突 | 冲突 | 冲突 | 冲突 |
| 写意向锁 | 冲突 | 兼容 | 冲突 | 兼容 |
| 表读锁 | 冲突 | 冲突 | 兼容 | 兼容 |
| 读意向锁 | 冲突 | 兼容 | 兼容 | 兼容 |

- InnoDB的意向锁是自动加的，不需要用户干涉
- 针对`update`、`delete`、`insert`语句，如果语句走的索引则会自动给相应数据行记录集加上行写锁；若语句不走索引，则会试图一次性为语句涉及到的所有表加上表写锁
- 对于普通的`select`语句，InnoDB不会加任何锁，但可以通过特殊的`select`语句为记录集的索引上添加行写锁或者行读锁：
	- 行读锁：`select * from table_name where ... lock in share mode`
	- 行写锁：`select * from table_name where ... for update`

InnoDB中的行锁类型：
- 记录锁
- 间隙锁
- 临键锁
- 插入意向锁

### 2. 乐观锁与悲观锁
悲观锁：对于数据的处理持悲观态度，总认为会发生并发冲突，获取和修改数据时，别人会修改数据。所以在整个数据处理过程中，需要将数据锁定，通常依靠数据库提供的锁机制实现

乐观锁：对数据的处理持乐观态度，乐观的认为数据一般情况下不会发生冲突，只有提交数据更新时，才会对数据是否冲突进行检测，实现不依靠数据库提供的锁机制，需要自已实现，实现方式一般是记录数据版本，一种是通过版本号，一种是通过时间戳

### 3. MVCC
在事务并发对行记录进行修改操作时，为了实现事务间的隔离与相应隔离级别的事务间数据可见度，InnoDB使用了**版本链**记录每次修改的事务ID与日志信息

<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\简易行格式.png" style="width:700px;height:180px;" />

行格式中使用`roll_pointer`连接行记录的各个版本、`trx_id`则为生成该版本的事务ID
（对于读写事务而言，只有在它第一次对某个表进行**增删改**操作时，才会为这个事务分配一个事务id，否则不会分配；如果不分配事务id，事务id的值默认为0）

<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\版本链.png" style="width:700px;height:500px;"/>

每对记录做一次修改操作，都要记录一条修改之前的日志，并且该日志还保存了当前事务的id，和行格式类似，这条日志也有一个`roll_pointer`节点
当对同一条记录更新的次数多了，所有的这些日志会被`roll_pointer`属性连接成一个单链表，这个链表就是版本链，而版本链的头节点就是当前记录的最新值

ReadView数据结构：
- `m_ids`：生成`ReadView`时，当前系统中活跃的读写事务id列表
- `min_trx_id`：生成`ReadView`时，当前系统中活跃的读写事务中最小的事务id，也就是`m_ids`中的最小值
- `max_trx_id`：生成`ReadView`时，待分配给下一个事务的id号
- `creator_trx_id`：生成当前`ReadView`的事务的事务id

<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\ReadView数据结构.png" style="width:700px;height:250px;"/>

结合版本链与ReadView，事务判断数据可见性规则如下：
1. 从版本链中的最新版本开始判断
2. 如果被访问版本的`trx_id = creator_trx_id`，说明这个版本就是当前事务修改的，允许访问
3. 如果被访问版本的`trx_id < min_trx_id`（未提交事务的最小id），说明生成这个版本的事务在当前`ReadView`生成之前就已经提交了，允许访问
4. 如果被访问版本的`trx_id > max_trx_id`（待分配的事务id），说明生成这个版本的事务是在当前ReadView生成之后建立的，不允许访问
5. 如果被访问版本的`trx_id`在`min_trx_id`和`max_trx_id`之间，那就需要判断`trx_id`是否在`m_ids`之中，如果在，说明生成当前`ReadView`时，生成该版本的事务还是活跃的，因此不允许访问。否则，可以访问
6. 如果当前版本不可见，就沿着版本链找到下一个版本，重复上面的2～5步

而对于`read committed`与`repeatable read`这两种隔离级别，之间的区别就是：生成`ReadView`时机不同，`read committed`是事务开始后，每次读取数据之前都重新生成一个`ReadView`；`repeatable read`是在事务开始后，只在第一次读取数据时生成一个`ReadView`，之后`ReadView`不再改变

事务并发存在三种情况：
- 读-读
- 读-写/写-读，由于并发可能存在脏读、不可重复读、幻读
- 写-写
针对并发问题，使用串行化解决效率太低；使用加锁解决并发可行；但在读多写少的情况下，效率也会低下
因此，**针对读一致性问题，使用MVCC可以解决，实现了读-写/写-读的无锁并发控制（在隔离级别为read committed与repeatable read时，才存在MVCC机制，读不需要锁）；而写一致性可以通过锁机制（表读写锁、行读写锁）完成**

## 11. 日志
### 1. bin log
Mysql的binlog日志作用是用来记录mysql内部增删改查等对mysql数据库有更新的内容的记录（对数据库的改动），对数据库的查询select或show等不会被binlog日志记录；主要用于数据库的主从复制以及增量恢复

Mysql配置文件my.cnf
```
log-bin=/data/3306/mysql-bin
```

查看启用binlog日志与否
```
mysql> show variables like 'log_bin';
```

可以使用Mysqlbinlog把Mysql中的二进制binlog日志文件解析出SQL语句

| 参数 | 描述 |
| ----- | ----- |
| -d | 指定库的binlog |
| -r | 相当于重定向到指定文件 |
| --start-position --stop-position | 按照指定位置精确解析binlog日志（精确），如不接--stop-positiion则一直到binlog日志结尾 |
| --start-datetime --stop-datetime | 按照指定时间解析binlog日志（模糊，不准确），如不接--stop-datetime则一直到binlog日志结尾 |

Mysql binlog的三种工作模式：
- `Row Level`：日志中会记录每一行数据被修改的情况，然后在slave端对相同的数据进行修改
- `Statement Level`(默认)：每一条被修改数据的sql都会记录到master的bin-log中，slave在复制的时候sql进程会解析成和原来master端执行过的相同的sql再次执行
- `Mixed`：结合了Row level和Statement level的优点

```sql
-- 查看binlog模式 --
mysql>show global variables like` `"binlog%";
+-----------------------------------------+-----------+
| Variable_name                           | Value     |
+-----------------------------------------+-----------+
| binlog_cache_size                       | 1048576   |
| binlog_direct_non_transactional_updates | OFF       |
| binlog_format                           | STATEMENT |      -- 系统默认为STATEMENT模式 --
| binlog_stmt_cache_size                  | 32768     |
+-----------------------------------------+-----------+
4 rows in set (0.00 sec)
```

```sql
mysql> set global binlog_format='ROW'; -- 会话中设置binlog模式 --
```

```sql
-- 配置文件中设置binlog --
[mysqld]
binlog_format='ROW'
user=xxx
port=xxxx
socket=xxx
```

### 2. redo log
`redo log`重做日志是InnoDB独有的，拥有崩溃恢复能力

Mysql中的数据以页为基本单位在内存和磁盘之间加载与载入。通常数据页会从磁盘中载入内存，放入`buffer pool`，后续的增删改查都是在内存中的`buffer pool`中进行。若没有命中数据则再去磁盘中加载数据页。

**而事务中增删改的SQL操作同时会把“在哪个数据页上做了什么修改”记录到重做日志缓存`redo log buffer`中，随后被刷新到磁盘的`redo log`文件中；每条redo记录由“表空间号+数据页号+偏移量+修改数据长度+具体修改的数据”组成**

**`InnoDB`存储引擎还存在一个后台线程，每隔`1`秒，就会把`redo log buffer`中的内容写到文件系统缓存（`page cache`），然后调用`fsync`刷盘**

<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\redo log刷盘流程.png" style="width:700px;height:400px;" />

`InnoDB`存储引擎为`redo log`的刷盘时机策略提供了`innodb_flush_log_at_trx_commit`参数，它支持三种策略
- 设置为0的时候，表示每次事务提交时不进行刷盘操作（提交事务那一刻宕机而没有进行`redo log`刷盘，可能会造成1秒内数据丢失）
- 设置为1的时候(默认)，表示每次事务提交时都将进行刷盘操作（默认值）（不会有任何损失）
- 设置为2的时候，表示每次事务提交时都只把redo log buffer内容写入page cache（写入page cache但没有调用fsync刷盘，可能造成1秒的数据丢失）

磁盘上`redo log`日志文件不只一个，是以一组日志文件存在的，组内每个日志文件大小一致，读写与擦除都是以环形文件组结构的顺序进行的
<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\环形redo log文件组.png" style="width:700px;height:600px;" />

环形日志文件组中存在`write pos`与`checkpoint`指针：
- `write pos`：当前写记录的位置，一边写一边后移
- `checkpoint`：当前擦除的位置，一边擦除一边后移
当`write pos`追上`checkpoint`时，表示`redo log`日志文件组满了，不能再写入新内容，需要清空一些记录让`checkpoint`往后移

### 3. undo log
回顾InnoDB的表空间存储结构，在InnoDB表空间中，存在`数据段`、`索引段`、`回滚段`，而`undo log`存放在表空间的回滚段中，用于保证事务的原子性与多版本MVCC并发控制，主要记录对数据库表增删改的撤销日志
一个事务中可能有多个增删改SQL，一个增删改SQL可能会产生多条`undo log`，一个事务中的`undo log`会从0开始被编号

<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\行记录格式隐藏列.png" style="width:700px;height:60px;" />
在InnoDB的行记录格式中，存在三个隐藏列：
- `DB_ROW_ID`：如果表中没有定义非空主键以及非空唯一索引，InnoDB会自动为表结构添加一个`row_id`隐藏列作为主键
- `DB_TRX_ID`：对某条记录作增删改操作，会将最后操作的事务ID写入该隐藏列
- `DB_ROLL_PTR`：回滚指针，指向`undo log`指针

<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\undo log页结构.png" style="width:700px;height:400px;" />

<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\undo page header结构.png" style="width:700px;height:500px;" />


<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\事务之间、普通临时表之间、insertupdate undo之间的页链表结构.png" style="width:700px;height:350px;" />

#### 1. insert undo
插入一条数据，与之对应的`undo`操作是根据主键删除该条数据，`insert`产生的`undo log`类型为`TRX_UNDO_INSERT_REC`，产生的行记录中`roll pointer`指向`undo log`日志

<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\insert undo的日志结构.png" style="width:700px;height:450px;" />

#### 2. delete undo
删除一条记录包括两个步骤：
1. 将行记录格式中的`delete_mask`标识位置为1，其他不做修改
2. 当删除语句所在的事务提交后，有专门的线程把记录从记录链表中移除，加入到空闲链表中

如果想实现事务提交前回滚，只需要针对删除记录中的第一个步骤作回滚即可，即将行记录格式中的`delete_mask`置为0

<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\delete undo的日志结构.png" style="width:700px;height:650px;" />

在对一条记录进行`delete mark`操作前，需要把该记录的旧`trx_id`和`roll_pointer`隐藏列的值都记录到对应的`undo日志`中，就是上图中显示的`old trx_id`和`old roll_pointer`属性。这样就可以通过`undo日志`的`old roll_pointer`找到记录在修改之前对应的`undo`日志
通过`roll_pointer`串成的日志链称为**MVCC中的版本链**

#### 3. update undo
1. 不更新主键的情况下
	- 就地更新：
		- 更新记录时，对于被更新的每个列来说，**如果更新后的列和更新前的列占用的存储空间都一样**，那么就可以进行就地更新，也就是直接在原记录的基础上修改对应列的值
	- 删除旧纪录并插入新纪录：
		- 在不更新主键的情况下，如果有任何一个被更新的列更新前和更新后占用的存储空间大小不一致，那么就需要先把这条旧的记录从聚簇索引页面中删除掉，然后再根据更新后列的值创建一条新的记录插入到页面中
		- **注意**：如果新创建的记录占用的存储空间大小不超过旧记录占用的空间，那么可以直接重用被加入到垃圾链表中的旧记录所占用的存储空间，否则的话需要在页面中新申请一段空间以供新记录使用，如果本页面内已经没有可用的空间的话，那就需要进行页面分裂操作，然后再插入新记录
2. 更新主键的情况下，分为两步
	- 将旧纪录进行`delete_mark`操作
	- 根据更新后各列的值创建一条新纪录，并把它插入到聚簇索引中

在`update`不更新主键情况下，对应`undo`日志结构：
<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\update undo的日志结构.png" style="width:700px;height:700px;" />

在`update`更新主键情况下，对记录进行`delete_mark`操作前，会产生一条`TRX_UNDO_DEL_MARK_REC`的`undo`日志；之后插入新纪录，又会产生一条`TRX_UNDO_INSERT_REC`的`undo`日志

## 12. 主从复制
随着业务的增长，一台数据服务器已经满足不了需求了，负载过重。这个时候就需要**减压**了，实现负载均衡读写分离，一主一丛、一主多从、多主一从、双主复制、级联复制
对于主服务器（Master）来说，主要负责写，从服务器（Slave）主要负责读，这样的话，就会大大减轻压力，从而提高效率

MySQL 主从复制是基于主服务器在二进制日志跟踪所有对数据库的更改。因此，要进行复制，必须在主服务器上启用二进制日志
每个从服务器从主服务器接收已经记录到日志的数据。当一个从服务器连接到主服务器时，它通知主服务器从服务器日志中读取最后一个更新成功的位置
从服务器接收从那时发生起的任何更新，并在主机上执行相同的更新。然后封锁等待主服务器通知的更新
从服务器执行备份不会干扰主服务器，在备份过程中主服务器可以继续处理更新

主从复制工作过程：
1. 从库生成**两个线程**，一个 I/O 线程，一个 SQL 线程；
2. I/O 线程去请求主库的 binlog，并将得到的 binlog 日志**写到** relay log(中继日志) 文件中；
3. 主库会**生成**一个 log dump 线程，用来给从库 I/O 线程传 binlog；
4. SQL 线程会读取 relay log 文件中的日志，并**解析**成具体操作，来实现主从的操作一致，而最终数据一致；
<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\主从复制过程.png" style="width:700px;height:350px;" />

主从复制可以分为：
- 主从同步：当主库执行完一个事务，然后所有的从库都复制了该事务并**成功执行完**才返回成功信息给客户端
- 主从异步：主节点**不会主动推送数据**到从节点，主库在执行完客户端提交的事务后会立即将结果返回给客户端，并不关心从库是否已经接收并处理
- 主从半同步：在异步复制的基础上，确保任何一个主库上的事物在提交之前**至少有一个从库**已经收到该事务并把日志记录下来

主从配置：
```
主数据库配置
server-id=1
log_bin=mysql-bin 打开日志
#binlog-do-db=czc 主机可以被同步的数据库 ，czc是一个数据库，自行先创建，不加则同步所有库
binlog-ignore-db=mysql 不给从机同步的数据库
binlog-ignore-db=performance_schema 不给从机同步的数据库
binlog-ignore-db=information_schema 不给从机同步的数据库
expire_logs_days=2 自动清理两天前的log文件
执行show master status，可以查看到file以及position的值，这两个值随后的从服务器会使用到
还需要在主机上创建用户并授权从机，让从机可以使用该用户复制binlog
create user 'slave1'@'%' identified by '123456';
grant replication slave on *.* to 'slave1'@'%';
alter user 'slave1'@'%' identified with mysql_native_password by '123456';
flush privileges;
执行show master status查看主机状态


从数据库配置方式1
server-id=2
master-host=192.168.1.1  主数据库的ip
master-user=slave1       主机中创建授权账号的用户名
master-password=123456   主机中创建授权账号的密码
master-port=3306
master-connect-retry=60
#replicate-do-db=czc    从机要同步的数据库,要同步多个数据库，就多加几个replicate-db-db=数据库名，不加则同步所有库
从数据库配置方式2
当mysql版本小于5.5，不能使用修改配置文件的方式直接配置，只能使用命令行的方式配置
配置文件中添加 server-id=3
执行以下change命令
change master to master_host='118.25.2437.342',master_port=3306,master_user='slave1',master_password='123456',master_log_file='mysql-bin.000015',master_log_pos=606;
注意，change master to这条命令中的master_log_file与master_log_pos可在主机的show mater status查看
change master to option='new value'
最后从mysql服务器执行start slave

执行show slave status查看从机状态，可以查看到slave_io_running与slave_sql_running，都为yes则算是配置成功
```

## 13. 备份与恢复
数据库备份恢复场景：
- 数据丢失应用场景
	1. 人为操作失误造成某些数据被误操作
	2. 软件BUG造成部分数据或全部数据丢失
	3. 硬件故障造成数据库部分或全部数据丢失
	4. 安全漏洞被入侵数据恶意破坏
- 非数据丢失应用场景
	1. 特殊应用场景下基于时间点的数据恢复
	2. 开发测试环境数据库搭建
	3. 相同数据库的新环境搭建
	4. 数据库或者数据迁移

<img src="D:\Project\IT-notes\框架or中间件\Mysql\img\备份方案.png" style="width:700px;height:300px;" />

|  | 备份方法1 | 备份方法2 | 备份方法3 |
| ----- | ----- | ----- | ----- |
| 根据是否需要数据库离线分类 | **热备**：在线备份，可以在数据库运行期间直接备份，不对读写操作产生影响<br/>**热备还可以根据备份后文件内容分为逻辑备份、裸文件备份**<br/>**逻辑备份**：备份出来的文件内容可读，内容可以为SQL语句或者表内实际数据<br/>**裸文件备份**：直接复制数据库的物理文件 | **冷备**：离线备份，必须在数据库停止情况下进行备份，不能进行读写操作 | **温备**：可以在数据库运行中进行备份，但对当前数据库的操作产生影响，备份时仅支持读操作，不支持写操作 |
| 根据备份范围分类 | **完全备份**：备份整个数据库 | **部分备份**：备份部分数据库，如备份一个表<br/>**部分备份可分为增量备份、差异备份**<br/>**增量备份**：在上一次完全备份的基础上，对变化的数据进行备份<br/>**差异备份**：备份自第一次完全备份以来变化的数据 |  |

Mysql中进行不同方式的备份还要考虑存储引擎是否支持
- `MyISAM`不支持热备份，支持温备份与冷备份
- `InnoDB`支持热备份、温备份、冷备份

需要备份的数据分为以下几种：
1. 表数据
2. 二进制日志、InnoDB事务日志
3. 代码（存储过程、存储函数、触发器、事件调度器）
4. 服务器配置文件

常用的备份工具
1. `mysqldump`：逻辑备份工具，适用于所有的存储引擎，支持温备、完全备份、部分备份、对于 InnoDB 存储引擎支持热备
2. `cp、tar`：归档复制工具、物理备份工具，适用于所有的存储引擎、冷备、完全备份、部分备份
3. `lvm2 snapshot`：借助文件系统管理工具进行备份
4. `mysqlhotcopy`：名不副实的一个工具，仅支持 MyISAM 存储引擎
5. `xtrabackup`：InnoDB/XtraDB 热备工具，支持完全备份、增量备份

| 备份工具 | 备份速度 | 恢复速度 | 便捷性 | 使用存储引擎 | 支持的备份类型 | 功能 |
| ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| cp、tar等（物理） | 快 | 快 | 一般 | 所有 | 冷备、全量、差异、增量 | 很弱 |
| lvm2快照（物理） | 快 | 快 | 一般 | 所有 | 支持几乎热备（即差不多是热备），是借助文件系统管理工具进行的备份 | 一般 |
| xtrabackup（物理） | 较快 | 较快 | 是一款非常强大的热备工具 | 由`percona`提供，只支持InnoDB/XtraDB | 热备、全量、差异、增量 | 强大 |
| mysqldump（逻辑） | 慢 | 慢 | 一般 | 所有 | 支持温备、完全备份、部分备份、对于**InnoDB**存储引擎支持热备 | 一般 |

备份策略与应用场景：
1. 如果数据量较小，可以直接`cp、tar`复制数据库文件，直接复制数据库文件
2. 如果数据量还行，可以`mysqldump+复制BIN LOGS`，先使用`mysqldump`对数据库进行完全备份，然后定期备份`BINARY LOG`达到增量备份的效果
3. 如果数据量一般，而又不过分影响业务运行，可以`lvm2快照+复制BIN LOGS`，使用`lvm2`的快照对数据文件进行备份，而后定期备份`BINARY LOG`达到增量备份的效果
4. 如果数据量很大，而又不过分影响业务运行，可以`xtrabackup+复制BIN LOGS`，使用`xtrabackup`进行完全备份后，定期使用`xtrabackup`进行增量备份或差异备份