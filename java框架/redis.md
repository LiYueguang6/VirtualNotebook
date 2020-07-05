# Redis

### Redis和MySQL同步数据

1. 更新数据时采取延时双删

	1）先删除缓存

	2）再写数据库

	3）休眠500毫秒

	4）再次删除缓存

	休眠时间之后删除缓存

2. 通过binlog，redis读取binlog的改变，来更新数据。类似MySQL自己主从复制

### redis分布式锁（jedis函数）

常用函数

| redis                | jedis                     | 作用                                                         |
| -------------------- | ------------------------- | ------------------------------------------------------------ |
| setnx key value      | jedis.set(keys,args,"NX") | “set if not exits”，若该key-value不存在，则成功加入缓存并且返回1，否则返回0。 |
| get(key)             |                           | 获得key对应的value值，若不存在则返回nil。                    |
| getset(key, value)   |                           | 先获取key对应的value值，若不存在则返回nil，然后将旧的value更新为新的value。 |
| expire(key, seconds) |                           | 设置key-value的有效期为seconds秒。                           |

**自己实现**

一个进程先判断是否存在key，存在则等待，否则设置key和过期时间（防止进程失效，锁未删除）。处理完成后判断key是否存在，签名是否是自己（防止超时后其他进程加锁），是则删除，不然直接退出。

**更多使用lua脚本**

```java
String str = "if redis.call('get',KEYS[1]) == ARGV[1] then return   redis.call('del',KEYS[1])  else return 0 end";                         
 
// keys 即加锁key ，args 即 密钥
jedis.eval(str, Collections.singletonList(keys), Collections.singletonList(args));
// Collections.singletonList 特殊集合：单元素集合，内存占用优化
```

### redis如何解决高并发

##### redis集群

星型、链式

全量复制：主生成RDB快照复制给从结点，并且记录之后的日志，也发送给从结点

增量复制：主节点每写一条数据，并向从结点发送

##### 读写分离

主写从读

##### 哨兵模式Sentinel（高可用）

主要功能包括主节点存活检测、主从运行情况检测、自动故障转移（failover）、主从切换



### redis持久化

1. RDB快照

	隔一段时间将所有数据拷贝到硬盘

2. AOF日志

	记录读之外的所有操作到日志文件。可以每次都写，可以每秒写，也可以系统决定（一般是日志缓冲区满了再持久化）