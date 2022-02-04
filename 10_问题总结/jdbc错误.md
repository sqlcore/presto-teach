# jdbc错误

[返回目录](../README.md)

---


## 错误信息

EXTERNAL JDBC_ERROR (67108864)

Attempted reconnect 3 times

```
com.facebook.presto.spi.PrestoException: Could not create connection to database server. Attempted reconnect 3 times. Giving up.
	at com.facebook.presto.plugin.jdbc.JdbcRecordCursor.handleSqlException(JdbcRecordCursor.java:236)
	at com.facebook.presto.plugin.jdbc.JdbcRecordCursor.<init>(JdbcRecordCursor.java:95)
	at com.facebook.presto.plugin.jdbc.JdbcRecordSet.cursor(JdbcRecordSet.java:59)
	at com.facebook.presto.spi.RecordPageSource.<init>(RecordPageSource.java:37)
	at com.facebook.presto.split.RecordPageSourceProvider.createPageSource(RecordPageSourceProvider.java:42)
	at com.facebook.presto.split.PageSourceManager.createPageSource(PageSourceManager.java:56)
	at com.facebook.presto.operator.ScanFilterAndProjectOperator.getOutput(ScanFilterAndProjectOperator.java:216)
	at com.facebook.presto.operator.Driver.processInternal(Driver.java:373)
	at com.facebook.presto.operator.Driver.lambda$processFor$8(Driver.java:282)
	at com.facebook.presto.operator.Driver.tryWithLock(Driver.java:672)
	at com.facebook.presto.operator.Driver.processFor(Driver.java:276)
	at com.facebook.presto.execution.SqlTaskExecution$DriverSplitRunner.processFor(SqlTaskExecution.java:973)
	at com.facebook.presto.execution.executor.PrioritizedSplitRunner.process(PrioritizedSplitRunner.java:162)
	at com.facebook.presto.execution.executor.TaskExecutor$TaskRunner.run(TaskExecutor.java:492)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)
Caused by: com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException: Could not create connection to database server. Attempted reconnect 3 times. Giving up.
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
```


## 解决方案

jdbc连接有问题，请检查连接字符串，账号，密码，网络。