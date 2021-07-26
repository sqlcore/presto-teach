# mongodb连接异常MongoTimeoutException

[返回目录](../README.md)

---

## 错误信息

com.mongodb.MongoTimeoutException

INTERNAL_ERROR GENERIC_INTERNAL_ERROR (65536)

```
com.mongodb.MongoTimeoutException: Timed out after 30000 ms while waiting to connect. Client view of cluster state is {type=UNKNOWN, servers=[{address=10.100.15.69:27017, type=UNKNOWN, state=CONNECTING, exception={com.mongodb.MongoSecurityException: Exception authenticating MongoCredential{mechanism=null, userName='bigdata', source='eagle_riskcon_third', password=<hidden>, mechanismProperties={}}}, caused by {com.mongodb.MongoCommandException: Command failed with error 18: 'Authentication failed.' on server 10.100.15.69:27017. The full response is { "ok" : 0.0, "errmsg" : "Authentication failed.", "code" : 18, "codeName" : "AuthenticationFailed" }}}]
	at com.mongodb.connection.BaseCluster.getDescription(BaseCluster.java:167)
	at com.mongodb.Mongo.getConnectedClusterDescription(Mongo.java:881)
	at com.mongodb.Mongo.createClientSession(Mongo.java:873)
	at com.mongodb.Mongo$3.getClientSession(Mongo.java:862)
	at com.mongodb.Mongo$3.execute(Mongo.java:819)
	at com.mongodb.MongoIterableImpl.execute(MongoIterableImpl.java:130)
	at com.mongodb.MongoIterableImpl.iterator(MongoIterableImpl.java:77)
	at com.mongodb.MappingIterable.iterator(MappingIterable.java:40)
	at com.mongodb.MappingIterable.iterator(MappingIterable.java:24)
	at com.google.common.collect.ImmutableList.copyOf(ImmutableList.java:234)
	at com.facebook.presto.mongodb.MongoSession.getAllSchemas(MongoSession.java:131)
	at com.facebook.presto.mongodb.MongoMetadata.listSchemaNames(MongoMetadata.java:72)
	at com.facebook.presto.mongodb.MongoMetadata.listSchemas(MongoMetadata.java:291)
	at com.facebook.presto.mongodb.MongoMetadata.listTables(MongoMetadata.java:101)
```

## 解决方案

检查mongodb连接的账号，密码，网络，能否正常连接