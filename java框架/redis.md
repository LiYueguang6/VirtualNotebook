# Redis

### 基本数据类型

String、List、Set（无序集合，不能重复）、Hash（不能重复）、Zset（有序name-score集合，name不能重复）

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

**redisson**

redisson已实现分布式锁

```java
Jedis jedis = redisUtil.getJedis();
RLock lock = redissonClient.getLock("lock");// 声明锁
lock.lock();//上锁
try {
	doSomething();
}finally {
	jedis.close();
	lock.unlock();// 解锁
}
```

#### redisson和jedis

redisson实现了分布式和可扩展的java数据结构。支持各种java数据类型，采用非阻塞I/O模型，和netty网络编程，线程安全。

jedis提供了比较全面的对redis命令的支持。仅支持基本数据类型，采用阻塞I/O模型，线程不安全。

### redis如何解决高并发

##### redis集群

星型、链式

全量复制：主生成RDB快照复制给从结点，并且记录之后的日志，也发送给从结点

增量复制：主节点每写一条数据，并向从结点发送

##### 读写分离

主写从读

##### 哨兵模式Sentinel（高可用）

主要功能包括主节点存活检测、主从运行情况检测、自动故障转移（failover）、主从切换

### redis解决大数据量的问题

通过对key的hash，分布数据到不同的redis集群中。

### redis持久化

1. RDB快照

	隔一段时间将所有数据拷贝到硬盘

2. AOF日志

	记录读之外的所有操作到日志文件。可以每次都写，可以每秒写，也可以系统决定（一般是日志缓冲区满了再持久化）

### 缓存穿透

恶意查询不存在的key，加重后端数据库的压力。
解决方法：对查询结果为空的情况进行缓存，存活时间可以设为久一点。把所有存在的key记录到一个布隆过滤器（redis支持bit的插入删除，天然优势。使用公式根据值的个数和误报率选取bit的位数和hash函数的个数）中，查询时过滤掉不存在的key，效率高但是存在误报。

### 缓存雪崩

多个缓存失效，后端数据库压力暴增
解决方法：搭建redis集群多级缓存；缓存失效后给后端数据库通过加锁限流，不同的key设置不同的过期时间，分散key失效的时间点

### 缓存击穿

热点缓存失效
解决方法：通过锁进行限流（分布式锁）
A.redis自带锁set  nx（只有不存在时才写入），因为锁是基于kv插入的，应用在具体业务上依然会产生并发问题，用redisson
B.redisson框架（可以理解为jedis+juc）结合apach压测

```java
Jedis jedis = redisUtil.getJedis();
RLock lock = redissonClient.getLock("lock");// 声明锁
lock.lock();//上锁
try {
	String v = jedis.get("k");
	if (StringUtils.isBlank(v)) {
		v = "1";
    }
    jedis.set("k", (Integer.parseInt(v) + 1) + "");
}finally {
	jedis.close();
	lock.unlock();// 解锁
}
```

### 缓存过期策略

**定时器**：消耗资源

**惰性**：浪费内存

**定期扫描**：消耗资源

redis同时使用了**惰性和定期扫描**

### 内存淘汰策略

noeviction：当内存不足以容纳新写入数据时，新写入操作会报错。

allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key。

allkeys-random：当内存不足以容纳新写入数据时，在键空间中，随机移除某个key。

volatile-lru：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的key。

volatile-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个key。

volatile-ttl：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的key优先移除。



