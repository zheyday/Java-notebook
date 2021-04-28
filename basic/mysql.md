# Mybatis

## 工作原理

是一个半ORM（对象关系映射）框架，封装了JDBC

基本工作原理：

先封装SQL，接着调用JDBC操作数据库，最后把返回的结果封装成Java类

### 优点

1. 基于sql编程，支持动态sql，比较灵活，sql写在xml中，和程序代码解耦；
2. 与JDBC相比，减少了代码量
3. 数据库兼容，JDBC支持的Mybatis都支持

### 缺点

1. sql语句编写工作量大
2. sql语句依赖数据库，移植性差

## 与Hibernate比较

1. Mybatis是一个半ORM框架，因为sql语句需要编写；使用Hibernate查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取，是全自动的
2. Mybatis直接编写sql，灵活度高，适合对关系数据模型要求不高的场景
3. Hibernate对象/关系映射能力强，**数据库**无关性好，适用对关系模型要求高的场景，可以节省代码，提高效率

## #{}和${}的区别

#{}是预编译处理，会被替换为?，调用PreparedStatement的set方法来赋值

${}是变量占位符，会被替换成变量的值

使用#{}可以有效防止SQL注入

## 预编译

**过程**

1. 执行预编译语句
2. 设置变量
3. 执行语句

如果需要再次执行，那么就不需要第一步了

**好处**

- 避免SQL注入，因为SQL已经编译完成，结构已经固定
- 提高SQL执行效率。当发送一条SQL语句给服务器时，服务器首先校验SQL的语法是否正确，然后编译成可执行的函数，最后才执行。其中前两步的时间可能比执行时间还要长。如果需要多次执行insert语句，每次只是参数不同，使用预编译的话只对SQL语句进行一次语法校验和编译，所以效率高



## 7、Dao接口的工作原理

Dao接口即Mapper接口

接口的全限名就是映射文件中的namespace；接口的方法名就是映射文件中的id值

当调用接口方法时，接口全限名+方法名可唯一定位一个MapperSatement，所以接口的方法不能重载

Mybatis运行时使用**JDK动态代理**为Mapper接口生成代理对象proxy，proxy拦截接口方法，执行MapperStatement所代表的sql，然后返回执行结果

## 8、一级、二级缓存

### 一级缓存

默认开启，其存储作用域为SqlSession。MyBatis开启一个数据库会话时，会创建一个新的SqlSession对象，SqlSession对象中有一个PerpetualCache对象(HashMap结构)；当执行CUD操作、Session flush或close之后，Session中的所有Cache都被清空

对于两次查询，如果以下条件都完全一样，那么就认为它们是完全相同的两次查询：

- 传入的statementId
- 传递的参数
- sql语句字符串
- 查询时要求的结果范围

### 二级缓存

存储作用域为Mapper，多个SqlSession类的实例对象可以共用二级缓存，并且可自定义存储源。默认不打开，使用二级缓存类需要实现Serializable序列化接口

# MySQL

## 数据类型

MySQL支持多种类型，大致可以分为三类：数值、日期/时间和字符串(字符)类型

**整数类型**

<img src="https://raw.githubusercontent.com/zheyday/BlogImg/master/img/image-20200710091254668.png" alt="image-20200710091254668" style="zoom:67%;" />

分为两种：

- 整数：tinyint(1B)、smallint(2B)、mediumint(3B)、int(4B)、bigint(8B)

- 实数：float、double、decimal

- 字符串：varchar（变长）、char（定长)、blob、text

  - char：插入时会在尾部填充空格，查询时会删除末尾空格
    适合场景：存储很短的字符串，因为不需要额外字节记录长度；所有值接近同一长度；对于经常变更的数据也不易产生碎片
  - varchar：存储可变长；需要额外字节记录长度（长度<=255用1个，大于用2个）。
    适合场景：列的最大长度比平均长度大很多；列更新少；使用了像UTF-8这样复杂的字符集
  - binary：定长二进制字符串，存储的是字节，尾部会加0x00，取出时保留所有字节
  - varbinary：变长二进制字符串，尾部不填充

  char和varchar类型在比较的时候忽略大小写和最后的空格，而binary和varbinary的字节比较时

- 日期和时间类型

  datetime：8个字节，默认为空，范围是1000-01-01 00:00:00 to 9999-12-31 23:59:59

  timestap：4个字节，不为空，默认为当前操作时间，范围是1970-01-01 00:00:01 to 2038-01-19 03:14:07，插入时会转换为UTC(世界标准时间)，查询时再转换为当前时区

## 连接

- INNER JOIN内连接：获取两个表中字段匹配的记录，可以直接用JOIN
- LEFT JOIN左连接：获取左表所有记录
- RIGHT JOIN右连接：获取右表所有记录

## 三大范式

 https://www.cnblogs.com/wsg25/p/9615100.html 

**第一范式（1NF）**

每个字段必须是不可拆分的最小单元，强调列的原子性

<img src="https://raw.githubusercontent.com/zheyday/BlogImg/master/img/image-20191129093015814.png" alt="image-20191129093015814" style="zoom:67%;" />

**第二范式（2NF）**

 https://blog.csdn.net/ljp812184246/article/details/50706596 

满足1NF后，所有字段都必须完全依赖主键，不能有任何一列与主键没有关系

假定选课关系表为SelectCourse(学号, 姓名, 年龄, 课程名称, 成绩, 学分)，关键字为组合关键字(学号, 课程名称)，因为存在如下决定关系： 
 		(学号, 课程名称) → (姓名, 年龄, 成绩, 学分) 

这个数据库表不满足第二范式，因为存在如下决定关系：

(课程名称) → (学分)

(学号) → (姓名, 年龄)

不符合2NF会存在以下问题：

1. 数据冗余
2. 更新异常：如果调整某课程的学分，那么所有行都要调整，否则会出现学分不同的情况
3. 插入异常：如果要开设新课程，暂时没有人选修，由于没有学号关键字，因此无法插入
4. 删除异常：如果这批学生已经结课，这些记录就应该删除，但是同时课程信息也被删除了

**第三范式（3NF）**

满足2NF后，所有字段只与主键**直接相关**而不是间接相关，不能包含依赖传递

 (学号) → (姓名, 年龄, 所在学院, 学院地点, 学院电话) 

这个数据库是符合2NF的，但是不符合3NF，因为存在如下决定关系：
	(学号) → (所在学院) → (学院地点, 学院电话)

即存在非关键字段"学院地点"、"学院电话"对关键字段"学号"的传递函数依赖。

优点：减少数据冗余

缺点：对于查询需要多个表进行关联，写效率降低，很难进行索引优化

## delete、truncate、drop

delete和truncate只删除数据，drop是删除整个表(结构和数据)

delete删除过程是一行一行删除，在日志保存删除记录，能够恢复，且保留标识计数值。

truncate是一次性将所有数据都删除，不能恢复，标识计数值重置

delete是DML，这个操作会被放到 rollback segment中，事务提交后才生效。如果有相应的 tigger，执行的时候将被触发。truncate和drop是DLL，立即执行

速度：drop> truncate > delete

```mysql
drop table test;
truncate table test;
delete from test [where id=1];
```

## 锁

https://www.cnblogs.com/leedaily/p/8378779.html

1. 表级锁

   直接锁定整张表，其他进程无法进行写操作。如果是写锁，则读操作也不允许。MyISAM采用

   两种模式：表共享读锁（Table Read Lock）和表独占写锁（Table Write Lock）。

   加锁：lock table study write/read [local];

   解锁：unlock tables;

   - 加了**“local”**选项，其作用就是在满足MyISAM表并发插入条件的情况下，允许其他用户在表尾并发插入记录;

   - 给表显式加表锁时，必须同时取得所有涉及到表的锁，并且MySQL不支持锁升级。也就是说，在执行LOCK TABLES后，只能访问显式加锁的这些表，不能访问未加锁的表
   - 如果加的是**读锁**，那么只能执行**查询**操作，而不能执行**更新**操作

2. 行级锁

   InnoDB支持行级锁和表级锁，默认是行级锁，只有通过索引才是行级锁

   - 共享锁（S）：又称**读锁**。若事务T对数据对象A加上S锁，则事务T可以读A但不能修改A，其他事务只能再对A加S锁，而不能加X锁，直到T释放A上的S锁。这保证了其他事务可以读A，但在T释放A上的S锁之前不能对A做任何修改。
   - 排他锁（X）：又称**写锁**。允许获取排他锁的事务更新数据，阻止其他事务取得相同的数据集共享读锁和排他写锁。若事务T对数据对象A加上X锁，事务T可以读A也可以修改A，其他事务不能再对A加任何锁，直到T释放A上的锁。

   **<u>注意</u>**：

   排他锁指的是一个事务在一行数据加上排他锁后，其他事务不能再在其上加其他的锁。InnoDB引擎默认的修改数据语句：**update,delete,insert**都会**自动**给涉及到的数据加上**排他锁**，**select**语句默认**不会加**任何锁类型（加排他锁可以使用select …for update语句，加共享锁可以使用select … for share语句）

3. 页级锁

   一次锁定相邻的一组记录

|          | 表级锁 | 行级锁 | 页级锁 |
| :------: | :----: | :----: | :----: |
|   开销   |   小   |   大   |  适中  |
| 加锁时间 |   快   |   慢   |  适中  |
| 锁定粒度 |  最大  |  最小  |  适中  |
|  并发度  |  最低  |  最高  |  适中  |

## 索引数据结构

### B树

又叫平衡多路查找树。一棵**m阶**的B树 (m叉树)的特性：

1. 如果整棵树不只有一个结点，则根结点**至少有2个**子结点

2. 中间结点有**k-1**个关键字和**k**个子结点，其中**ceil(m / 2)<= k <= m**（ceil(x)是一个取上限的函数）

3. 所有叶子结点都在同一层

4. 非叶子结点包含n个关键字信息 (P1，K1，P2，K2，P3，......，Kn，Pn+1),其中：

   a)   Ki (i=1...n)为关键字，且关键字按顺序升序排序K(i-1)< Ki。

   b)   Pi为指向子结点的接点，且指针P(i)指向的子树中的所有结点的关键字均小于Ki，但都大于K(i-1)。 

   c)   关键字的个数n必须满足： [ceil(m / 2)-1]<= n <= m-1。

B树相对于平衡二叉树来说，每个节点的关键字增多了，树的层数变少，那么查找次数就少了，搜索磁盘的效率变高了

### B+树

采用B+树，比B树更充分的利用了节点空间，查询速度更稳定

1. 每个结点最多有**m个**关键字（B树最多m-1），非叶子结点的子树指针与关键字个数**相同**
2. 所有叶子结点包含了全部的关键字信息，且按照关键字顺序相连
3. 所有数据仅存在叶子结点
4. 为所有叶子结点增加一个链指针
5. 非叶子结点的子树指针P[i]，指向关键字值属于[K[i], K[i+1])的子树（B-树是开区间）；

选用B+树的原因：

1. 由于B+树在非叶子节点上不包含数据信息，因此单个节点能够存放更多的key。 数据存放的更加紧密，具有更好的空间局部性。
2. B+树查询性能稳定，因为必须要搜寻到叶子结点
3. B+树的叶子结点都是相链的，因此对整棵树的遍历只需要一次线性遍历叶子结点即可。而且由于数据顺序排列并且相连，所以便于区间查找和搜索。而B树则需要进行每一层的递归遍历。相邻的元素可能在内存中不相邻，所以缓存命中性没有B+树好。

##  数据库索引

http://blog.codinglabs.org/articles/theory-of-mysql-index.html

**优缺点**

优点：

  1. 加快数据检索速度
   2. 使用索引可以在查询的过程中，使用优化隐藏器，提高系统性能

缺点：

  1. 创建和维护索引需要耗费时间，且随着数据量增加而增加
  2. 索引占用物理空间
  3. 当对表中数据进行insert、delete、update时候，索引也需要动态维护

### 索引失效

1. 条件中有or。如果想使索引生效，必须将or条件的每个列都加上索引
2. like查询以%开头
4. 列是字符串类型，必须要在条件中对数据使用引号，否则索引失效
4. 对索引列进行运算
5. 对于联合索引，如果不符合最左前缀匹配原则，索引失效
6. mysql估计使用全表扫描比使用索引快

**普通索引**

允许重复，加快查询速度

```mysql
创建索引
1、CREATE INDEX index_name ON table_name(col_name)
2、ALTER TABLE table_name ADD INDEX index_name(col_name)
3、CREATE TABLE mytable(  
ID INT NOT NULL,   
username VARCHAR(16) NOT NULL,  
INDEX indexName (username)  
); 

删除索引
DROP INDEX [indexName] ON table_name; 
```

如果是CHAR，VARCHAR类型，length可以小于字段实际长度；如果是BLOB和TEXT类型，必须指定 length。

**唯一索引**

它与前面的普通索引类似，不同的就是：索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。不可以被其他表引用为外键

```mysql
1、CREATE UNIQUE INDEX indexName ON table(col_name) 
2、ALTER TABLE mytable ADD UNIQUE index_name(col_name)
3、CREATE TABLE mytable(  
ID INT NOT NULL,   
usrname VARCHAR(16) NOT NULL,   
UNIQUE INDEX indexName (username)  
);  
```

**主键索引**

在数据库表关系图中自动为主键创建主键索引，它是唯一索引的特定类型，不能为空，可以被其他表引用为外键

**联合索引**

https://juejin.im/post/5e57ac99e51d45270e212534

<img src="https://raw.githubusercontent.com/zheyday/BlogImg/master/img/bVbmBDQ" alt="preview" style="zoom: 80%;" />

<img src="https://raw.githubusercontent.com/zheyday/BlogImg/master/img/170867cb6af0a72d" alt="img" style="zoom:80%;" />

**聚集索引(聚簇索引)**

数据行的物理顺序与键值的逻辑顺序相同。一个表只能包含一个聚集索引。

聚集索引的叶子节点存放的就是对应的数据，

如果某索引不是聚集索引，则表中行的物理顺序与键值的逻辑顺序不匹配。与非聚集索引相比，聚集索引通常提供更快的数据访问速度 

InnoDB使用聚集索引，MyISAM使用非聚集索引

<img src="https://raw.githubusercontent.com/zheyday/BlogImg/master/img/image-20200413214435993.png" alt="image-20200413214435993" style="zoom:67%;" />

## 引擎区别

1. MyISAM是非事务安全的，不支持外键，而InnoDB是事务安全的，支持外键

2. MyISAM效率上要优于InnoDB，小型应用可以考虑使用MyISAM。为什么快？
   因为在进行select的时候，InnoDB要维护的东西比较多
   1）InnoDB要缓存数据块，MyISAM只缓存索引块
   2）InnoDB寻址要映射到块，然后到行，而MyISAM记录的直接是文件的OFFSET，定位更快
   3）InnoDB要维护MVCC一致

3. MyISAM表保存成文件形式，跨平台使用更加方便 

4. MyISAM 只支持表级锁，InnoDB 支持行级锁和表级锁，默认是行级锁 ，行锁大幅度提高了多用户并发操作的性能 。InnoDB 比较适合于插入和更新操作比较多的情况，而 MyISAM则适合用于频繁查询的情况。 
   另外，InnoDB 表的行锁也不是绝对的，如果在执行一个 SQL 语句时，MySQL 不能确定要扫描的范围，InnoDB 表同样会锁全表，例如 update table set num=1 where name like “%aaa%”。 

5. MyISAM： select count(`*`) from table

   MyISAM 只要简单的读出保存好的行数。因为MyISAM 内置了一个计数器，count(`*`)时它直接从计数器中读。
   InnoDB：不保存表的具体行数，也就是说，执行 select count(*) from table 时，InnoDB要扫描一遍整个表来计算有多少行。 

## 最左前缀匹配



## 大表优化

### Explain命令

模拟优化器执行SQL查询语句，从而分析性能瓶颈

在(a,b,c)上创建了一个联合索引，d是主键索引

```mysql
mysql> explain select * from test where a<10\G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: test
   partitions: NULL
         type: range
possible_keys: index_abc
          key: index_abc
      key_len: 5
          ref: NULL
         rows: 9
     filtered: 100.00
        Extra: Using index condition
1 row in set, 1 warning (0.00 sec)
```

- select_type：查询类型，用于区别普通查询、联合查询、子查询等复杂查询

  - simple：简单查询
  - primary：复杂查询的最外层查询
  - subquery：在select或where中的子查询
  - derived：在from中的子查询
  - union：如果第二个select出现在union之后，则被标记为union；如果union包含在from子句的子查询中，外层select会被标记为derived
  - union result：从union表获取结果的select

- type：访问类型，NULL,system,const,eq_ref,ref,range,index,ALL（从左到右，性能从好到差）

  | 类型           | 说明                                                         |
  | -------------- | ------------------------------------------------------------ |
  | NULL           | mysql能够在优化阶段分解查询语句，在执行阶段不需要访问表或索引，比如min |
  | system / const | 针对主键或唯一索引的查询，system是特殊的const                |
  | eq_ref         | 连接查询中，主键或唯一索引都被使用，最多返回一条             |
  | ref            | 和eq_ref相比，不使用唯一索引，而使用普通索引或联合索引的部分前缀 |
  | range          | 只检索给定范围的行                                           |
  | index          | 扫描索引树                                                   |
  | all            | 全表扫描                                                     |

- key：实际使用的索引

- ref：显示哪些列或常量被用于查找索引列上的值

### 单表优化

一般以整型值为主的表在`千万级`以下，字符串为主的表在`五百万`以下是没有太大问题的

**字段**

- 尽量使用较小的整数类型，尽量不使用INT，非负加上UNSIGNED
- varchar长度只分配真正需要的空间
- 使用枚举或整数替代字符串
- 尽量使用`TIMESTAMP`而非`DATETIME`
- 使用整型存储IP

**索引**

- 在经常需要搜索、根据范围进行搜索的列上创建索引，因为索引已经排序，其指定的范围是连续的 
- 在`WHERE`和`ORDER BY`命令上涉及的列建立索引，可根据`EXPLAIN`来查看用了索引还是全表扫描
- 字符字段只建前缀索引，且最好不要做主键
- 尽量扩展索引而不是新建索引
- 对于只有**很少数据值**的列也不应该增加索引 。比如性别列
- 对于那些定义为text, image和bit数据类型的列不应该增加索引。这是因为，这些列的数据量要么相当大，要么取值很少。 
- 当修改性能远远大于检索性能时，不应该创建索引
- 不要让索引的默认值为NULL

**优化sql**

- 在where中**避免使用**

  - 使用or来连接条件。如：

    select id from t where num=10 or num=20
    可以这样查询：

    select id from t where num=10
    **union all**
    select id from t where num=20 

  - 避免对字段进行表达式操作
    num/2=1改为num=1*2

  这些会使引擎放弃使用索引而进行全表扫描

- 少用join

- 下面的查询也将导致全表扫描： 

  select id from t where name like '%abc%' 

  若要提高效率，可以考虑全文检索 

### 垂直拆分

根据业务相关性进行拆分

优点：行数据变小，一个数据块能存放更多的数据，减少磁盘I/O

缺点：主键冗余；会引起表的join操作；事务处理复杂

### 水平拆分

分表：解决单表大量数据的查询性能问题

分库：解决单台数据库的高并发问题。根据业务不同进行切分

分区：就是把一张表的数据分成多个区块，这些区块可以在一个磁盘上，也可以在不同的磁盘上，分区后，表面上还是一张表，但数据散列在多个位置，这样一来，多块硬盘同时处理不同的请求，从而提高磁盘 I/O 读写性能，实现比较简单。 包括水平分区和垂直分区。 

### 优化实战

- 表的大小没到1kw，主键使用medium_int
- name使用varchar

### 主从复制

https://blog.csdn.net/qq_30299977/article/details/100087045

原理：

​	在MySQL中有一种叫做bin的二进制日志，这个日志文件里面记录了关于此数据库的所有修改的sql语句（包括insert，update，delete，grant等等）。而主从复制就是利用这个二进制bin日志，在主库上创建一个用户，从数据库通过此用户去读取bin日志，然后再在从数据库上再执行一次。

复制方式：

1. 基于SQL复制 (statement-based replication, SBR)：binlog文件小，包含了所有数据库更改信息；可以用于实时还原；当复制需要进行全表扫描的UPDATE时，需要比RBR请求更多的行级锁；执行复杂的语句需要消耗更多的资源
2. 基于行复制 (row-based replication, RBR)：任何情况都可以被复制；binlog更大；主服务器执行UPDATE时，所有发送变化的记录都会写到binlog，而SBR只写一次
3. 混合模式复制 (mixed-based replication, MBR)

同步方式：

1. 同步复制

   主服务器在将更新的数据写入它的二进制日志（bin-log）文件中后，<u>**必须等待验**</u>证所有的从服务器的更新数据是否已经复制到其中，之后才可以自由处理其它进入的事务处理请求 

2. 异步复制

   主服务器在将更新的数据写入它的二进制日志（bin-log）文件中后，<u>**无需等待验证**</u>更新数据是否已经复制到从服务器中，就可以自由处理其它进入的事务处理请求。 

3. 半同步复制

   主服务器在将更新的数据写入它的二进制日志（bin-log）文件中后，只需**<u>等待验证其中一台</u>**从服务器的更新数据是否已经复制到其中，就可以自由处理其它进入的事务处理请求，其他的从服务器不用管 

实现：https://www.jianshu.com/p/5c6d6ffa98af

## SQL注入

举个例子：

后台java代码拼接sql如下：

public List getInfo(String title){

StringBuffer buf = new StringBuffer();

buf.append("select id,title from study where title=' ").append(title).append(" ' ");

}

前端页面有输入框如下：

主题：___________

如果输入为 ' or '1'='1

最后拼接结果如下：

select id,title from study where title = ' ' or '1'='1'

那么查询结果就是所有数据，这就是sql注入。

**防止措施**

1. PreparedStatement 

   采用预编译语句集，它内置了处理SQL注入的能力，只要使用它的setXXX方法传值即可。

2. 使用正则表达式过滤传入的参数

## MVCC

https://www.jianshu.com/p/fd51cb8dc03b

https://blog.csdn.net/Waves___/article/details/105295060

https://blog.csdn.net/sanyuesan0000/article/details/90235335

MVCC并非乐观锁，但是InnoDB存储引擎所实现的MVCC是乐观的

InnoDB默认隔离级别是RR，是通过MVCC解决了脏读、不可重复读。只解决了读操作的幻读（select使用的是快照，也就是多版本），对于修改操作依然存在幻读。解决幻读的方法是MVCC+next-key locks。next-key lock是行锁的一种，实现相当于record lock(行记录的锁，锁定一个记录上的索引，而不是记录本身) + gap lock(间隙锁，锁定索引之间的间隙，但是不包括数据本身)；其特点是不仅会锁住记录本身(record lock的功能)，还会锁定一个范围(gap lock的功能)

SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX\G

MVCC(Multiversion Concurrency Control)即多版本并发控制技术，它使得大部分支持行锁的事务引擎，不再单纯使用行锁来进行数据库的并发控制，而是把**数据库的行锁与行的多个版本结合**起来，只需要很小的开销,就可以实现非锁定读，读写不冲突，从而大大提高数据库系统的并发性能。

**具体实现**

不同引擎的实现不同，典型的有乐观并发控制和悲观并发控制。以InnoDB为例。下面是REPEATABLE READ隔离级别下MVCC的具体操作： 

InnoDB的MVCC,是通过在每行记录后面保存三个隐藏的列来实现的(不是两个，别被误导了)：

- 6字节的事务ID（DB_TRX_ID）：标记最近更新这条行记录的事务ID（更新包括delete和update，update实质是新插入一行，并且标记原有行为删除；delete操作，InnoDB认为是一个update操作，不过会更新一个另外的删除位，将行表示为deleted。并非真正删除。）
- 7字节的回滚指针（DB_ROLL_PTR）：指向undo log信息
- 6字节隐藏的ID（DB_ROW_ID）：行ID，当表没有主键或唯一索引时，InnoDB就会使用这个行ID自动产生聚簇索引，否则这列就没用

MVCC需要依赖undo log和read view

### undo log

<img src="https://raw.githubusercontent.com/zheyday/BlogImg/master/img/image-20200627220937599.png" alt="image-20200627220937599" style="zoom: 80%;" />

当更改行数据时，使用排他锁锁定该行，记录到redo log，将该行复制到undo log，然后修改当前行的值，填写事务编号，使回滚指针指向undo log中的数据

<img src="https://raw.githubusercontent.com/zheyday/BlogImg/master/img/image-20200627220952052.png" alt="image-20200627220952052" style="zoom: 67%;" />

### Read view

创建事务的时候，InnoDB会将当前系统中处于活跃的事务ID保存得到一个副本(read view)，其实保存的也就是系统中当前事务不应该看到的事务ID列表，如果没有其他事务，那么read view就不存在。

[Read View结构源码](https://github.com/facebook/mysql-5.6/blob/42a5444d52f264682c7805bf8117dd884095c476/storage/innobase/include/read0read.h#L125)

有几个重要属性：

- trx_ids：当前系统未提交事务ID的集合，不包括当前事务和内存中正commit的事务
- creator_trx_id：创建当前read view时的事务ID
- low_limit_id：目前出现过的最大事务ID+1
- up_limit_id：trx_ids中最小的事务ID，如果trx_ids为空，则等于low_limit_id

这两个属性看起来不太符合习惯，但是没有写反

RC下每条select都会重新生成read view

RR下执行第一条select生成

匹配条件

- 数据行事务ID<up_limit_id或者等于creator_trx_id（说明数据是事务自己生成的），则显示

- 数据行事务ID>=low_limit_id则不显示

- trx_ids为空则显示

- up_limit_id<=行事务ID<low_limit_id，则与trx_ids匹配

  情况1：如果行事务ID不存在于trx_ids集合中，说明read view产生的时候事务已经提交了，则可以显示
  情况2：如果行事务ID存在trx_ids中，就说明read view产生的时候数据没有commit，且不是自己生成的，则不可以显示
  
- 不满足read view的时候，根据数据的回滚指针从undo log获取数据的历史版本，根据历史数据版本号再重新和read view条件匹配，直到找到满足条件的，或者返回空

![](https://raw.githubusercontent.com/zheyday/BlogImg/master/img/20200403172924433.png)

## 事务的实现原理

https://www.cnblogs.com/kismetv/p/10331633.html

ACID是事务的四个特性

### ACID

**原子性Atomicity**

一个事务是最小工作单元，整个事务中的操作要么全部成功，要么全部失败回滚。

undo log是事务实现原子性和隔离性的基础。当事务对数据库进行修改时，InnoDB会生成相应的undo log，如果事务失败或者回滚，就可以利用undo log的信息将数据回滚

**一致性Consistency**

事务开始前和结束后，数据库的完整性没有被破坏

一致性是目的，通过其他三个特性来保证

**隔离性Isolation**

多个事务并发执行时，事务之间是互相隔离的，不能互相干扰。

锁+MVCC

**持久性Durability**

事务一旦提交，对数据库的改变是永久性的，不能回滚。

通过redo log保证，当修改数据时，redo log记录这次操作。当事务提交时，将redo log日志进行刷盘

## 事务的隔离级别

 - Read Uncommitted读未提交
 隔离级别最低。一个事务B会读到另一个事务A更新后但未提交的数据，如果A回滚，那么B读到的数据就是脏数据，这就是脏读。
    ![在这里插入图片描述](https://raw.githubusercontent.com/zheyday/BlogImg/master/img/20190708100609959.png)

 - Read Committed
        一个事务从开始直到提交之前，所做的任何修改对其他事务都是不可见的。
	
	​       在事务B内，多次读同一数据，在这个事务还没有结束时，如果事务A恰好修改了这个数据，那么，在第一个事务中，两次读取的数据就可能不一致，这就是**不可重复读**（Non Repeatable Read）的问题，不可重复读偏向update操作
	 	![在这里插入图片描述](https://raw.githubusercontent.com/zheyday/BlogImg/master/img/20190708101440567.png)
	
 - Repeatable Read（MySQL默认） set transaction isolation level read committed;
 解决了脏读和不可重复读的问题。
  问题： 在事务B中读取数据，另外一个事务A插入一条新记录，当B再次读取时，没有数据，试图更新这条不存在的记录时，竟然能成功，并且，再次读取同一条记录，它就神奇地出现了。这就是**幻读**（Phantom Read），幻读偏重insert操作
    ![在这里插入图片描述](https://raw.githubusercontent.com/zheyday/BlogImg/master/img/20190708101646305.png)

 - Serializable
 最严格的隔离级别，所有事务按照次序依次执行，因此，脏读、不可重复读、幻读都不会出现。
  但是，由于事务是串行执行，所以效率会大大下降，应用程序的性能会急剧降低。如果没有特别重要的情景，一般都不会使用Serializable隔离级别。

**默认隔离级别**
 在MySQL中，如果使用InnoDB，默认的隔离级别是Repeatable Read。

## 数据恢复

1、配置my.ini

`server_id = 1
log_bin = mysql-bin.log　　#设置log文件保存路径，默认为mysql的data目录下
max_binlog_size = 512M　　#最大log
binlog_format = row
binlog_row_image = full`

2、`show binary logs;`查看最新log

3、`show binlog events in 'mysql-bin.000106;`

可以看到228建立了一个test表，475删除了test

![在这里插入图片描述](https://raw.githubusercontent.com/zheyday/BlogImg/master/img/image-20191212104847047.png)

4、打开一个cmd，进入data目录，执行下列命令

终止位置要是402，因为创建test的命令是在402结束

![20191212105214496](https://raw.githubusercontent.com/zheyday/BlogImg/master/img/20191212105214496.png)

5、进入mysql，执行 `source e:u.sql;`

## 常用命令

创建数据库：create database people;

删除数据库：drop database people;

选择数据库：use people;

创建数据表：

```mysql
CREATE TABLE `crawl`.`people`  (
  `id` int UNSIGNED NOT NULL AUTO_INCREMENT,
  `name` varchar(10) NOT NULL,
  PRIMARY KEY (`id`)
);
```

删除表数据：delete from success_killed;

删除表：drop table seckill;

插入数据：insert into people (field1,field2) values (value1,value2);

更新数据：update people set filed1=value1;

插入或更新：insert into people (id,name) values (3,'user') on duplicate key update name='root';

插入或忽略：insert ignore into people (id,name) values (3,'user');

删除数据：delete from people where filed1=value1;







