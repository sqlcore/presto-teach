# hive元数据连接超时

[返回目录](../README.md)

---

## 错误信息

EXTERNAL HIVE_METASTORE_ERROR (16777216)

```
com.facebook.presto.spi.PrestoException: hive的元数据地址: java.net.SocketTimeoutException: Read timed out
	at com.facebook.presto.hive.metastore.thrift.ThriftHiveMetastore.getTable(ThriftHiveMetastore.java:221)
	at com.facebook.presto.hive.metastore.thrift.BridgingHiveMetastore.getTable(BridgingHiveMetastore.java:82)
	at com.facebook.presto.hive.metastore.CachingHiveMetastore.loadTable(CachingHiveMetastore.java:257)
	at com.google.common.cache.CacheLoader$FunctionToCacheLoader.load(CacheLoader.java:165)
	at com.google.common.cache.CacheLoader$1.load(CacheLoader.java:188)
	at com.google.common.cache.LocalCache$LoadingValueReference.loadFuture(LocalCache.java:3524)
	at com.google.common.cache.LocalCache$Segment.loadSync(LocalCache.java:2273)
	at com.google.common.cache.LocalCache$Segment.lockedGetOrLoad(LocalCache.java:2156)
	at com.google.common.cache.LocalCache$Segment.get(LocalCache.java:2046)
	at com.google.common.cache.LocalCache.get(LocalCache.java:3943)
	at com.google.common.cache.LocalCache.getOrLoad(LocalCache.java:3967)
	at com.google.common.cache.LocalCache$LocalLoadingCache.get(LocalCache.java:4952)
	at com.google.common.cache.LocalCache$LocalLoadingCache.getUnchecked(LocalCache.java:4958)
	at com.facebook.presto.hive.metastore.CachingHiveMetastore.get(CachingHiveMetastore.java:207)
	at com.facebook.presto.hive.metastore.CachingHiveMetastore.getTable(CachingHiveMetastore.java:252)
```

对应sql

```sql
SHOW COLUMNS FROM "
  里面是一个常规的查询语句
"
```

## 解决方案

表象是 hive 元数据连接超时。

实际应该是不支持这种sql。可是sql解析是过了，走到执行的逻辑了，进行相应执行的时候报错。

经过沟通发现，有一些业务为会为了长时间保持能够顺利快速的查询(因为presto在第一次建立连接查询时，速度较慢)会在本机配置一个自动任务，每隔一段时间来执行一个sql，然后很巧的是我们的业务大部分都是使用这样的sql，来保持与presto的长连接，以达到查询的尴尬提速。

这变相算是一个使用体验的问题把。
