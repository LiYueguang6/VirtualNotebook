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

