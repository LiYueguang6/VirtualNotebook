# innodb_io_capacity

## 源码大致的调用关系

``` text
buf_flush_batch 调用 buf_flush_try_neighbors （尝试刷写neighbor page）
buf_flush_try_neighbors 调用 buf_flush_page （刷写单个page）
buf_flush_page调用buf_flush_write_block_low （实际刷写单个page）
buf_flush_write_block_low调用buf_flush_post_to_doublewrite_buf （将page放到double write buffer中，并准备刷写）
buf_flush_post_to_doublewrite_buf 调用 fil_io （ 文件IO的封装）
fil_io 调用 os_aio （aio相关操作）
os_aio 调用 os_file_write （实际写文件操作）
```

其中buf_flush_batch 只有两种刷写方式： BUF_FLUSH_LIST 和 BUF_FLUSH_LRU 两种方式的方式和触发时机简介如下：  

BUF_FLUSH_LIST: innodb master线程中 1_second / 10 second 循环中都会调用。触发条件较多（下文会分析）
BUF_FLUSH_LRU: 当Buffer Pool无空闲page且old list中没有足够的clean page时，调用。刷写脏页后可以空出一定的free page，供BP使用。
从触发频率可以看到 10 second 循环中对于 buf_flush_batch( BUF_FLUSH_LIST ) 的调用是10秒一次IO高负载的元凶所在。

我们再来看10秒循环中flush的逻辑：
通过比较过去10秒的IO次数和常量的大小，以及pending的IO次数，来判断IO是否空闲，如果空闲则buf_flush_batch( BUF_FLUSH_LIST,PCT_IO(100) );
如果脏页比例超过70，则 buf_flush_batch( BUF_FLUSH_LIST,PCT_IO(100) );
否则  buf_flush_batch( BUF_FLUSH_LIST,PCT_IO(10) );

可以看到由于SSD对于随机写的请求响应速度非常快，导致IO几乎没有堆积。也就让innodb误认为IO空闲，并决定全力刷写。
其中PCT_IO(N)  = innodb_io_capacity *N% ，单位是页。因此也就意味着每10秒，innodb都至少刷10000个page或者刷完当前所有脏页。

## 结论

innodb_io_capacity 增大后会影响一次刷盘的脏页page数量
当CPU认为io空闲的时候或者脏页超过70%的时候，会刷新100%*innodb_io_capacity 的脏页
其他情况刷新10%*innodb_io_capacity 的脏页
刷新这个脏页的时候，数据库将停止所有工作，全力刷写，单次刷新太多脏页会引起数据库波动，博客都是在**降低**innodb_io_capacity **提高**稳定性，但是无法判断这个参数对吞吐量的影响
个人理解在
