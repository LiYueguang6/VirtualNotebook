# SQL

## SQL执行过程

1.解析sql语句，生成查询计划

2.检查是否存在已经生成的查询计划（避免多次编译）

3.检查缓冲区的数据是否满足需求

4.没有查询计划则检查语法语义

5.加锁

6.检查权限

7.优化SQL

8.保存到SQL计划缓存中

9.执行SQL

## SQL执行顺序

from on join where group rollup having select distinct orderby limit

## 连接

1. 左连接 left join

	左边的表无限制

2. 右链接 right join

	右边的表无限制

3. 内连接 inner join/join

	两个表符合条件的才会查询到。（和等值连接相同）

4. 自然连接 natural join

	不用写条件on，自动取相同名称的属性列进行连接，基本和内连接类似。

5. 全外连接 full join（mysql不存在）

  两张表都不加限制

## 查询步骤

1. 查询分析：词法分析、语法分析（查询语句是否符合sql语法规则）
2. 查询检查：语义分析（表、属性名是否有效），符号名转换，安全性检查（权限），完整性初步检查。
3. （查询树）生成关系代数表达式，一般用查询树来表示关系代数表达式
4. 查询优化：代数优化（对关系代数进行**等价变换**，使查询更加高效）、物理优化（对数据存储和底层操作算法进行选择，实际要考虑规则、代价和语义的优化器）
5. （查询执行计划）
6. 查询执行：代码生成（代码生成器生成查询计划的代码，执行并返回结果）
7. （查询计划的执行代码）



## SQL优化

#### explain详解

id:选择标识符

select_type:表示查询的类型。

table:输出结果集的表

partitions:匹配的分区

**type**:表示表的连接类型

- ALL：Full Table Scan， MySQL将遍历全表以找到匹配的行
- index: Full Index Scan，index与ALL区别为index类型只遍历索引树
- range:只检索给定范围的行，使用一个索引来选择行
- ref: 表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值
- eq_ref: 类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用primary key或者 unique key作为关联条件
- const、system: 当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。如将主键置于where列表中，MySQL就能将该查询转换为一个常量，system是const类型的特例，当查询的表只有一行的情况下，使用system
- NULL: MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成。

**possible_keys**:表示查询时，可能使用的索引

**key**:表示实际使用的索引

**key_len**:索引字段的长度，索引使用的字节数

**ref**:列与索引的比较

**rows**:扫描出的行数(估算的行数)

filtered:按表条件过滤的行百分比

Extra:执行情况的描述和说明



## 数据库优化面试实例

1. Q：一张大数据表上存在卖家和买家两个id，如何对该表进行优化

	A：将每一个数据存储两份，分为买家库和卖家库。在卖家库上使用卖家id进行hash分表，买家库上使用买家id进行hash分表。

2. Q：select * from Order order by money desc limit 1000,100;如何优化。

	A：对id列建索引。select * from Order where money <= (selct money from Order order by money desc limit 1000,1) limit 100 ;这样可以先对id列进行覆盖索引的查询，再去主键进行io读取数据。否则limit会将1000条数据都读出来，这部分消耗大。

