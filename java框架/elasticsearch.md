# elasticsearch

ES6

## ES的数据存储结构

面向文档的，使用JSON格式进行数据存储

一个文档的格式：

```json
{
 "_index" :  "文档所在的索引",
 "_type" :  "文档所在类型名称",
 "_id" :   "唯一id",
 "_version" : 1,
 "found" :  true,
 "_source" : {
   "first_name" : "John",
   "last_name" :  "Smith",
   "age" :     25,
   "about" :    "I love to go rock climbing",
   "interests": [ "sports", "music" ]
 }
}
```

文档：一个文档就是一条数据，可以是包括商品信息的商品数据

_source：文档的数据，其中包含多个field



通过http协议访问，get，delete

## 原理

倒排索引

