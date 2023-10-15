# 对接Prometheus生态

对接Prometheus生态，最重要的就是将监控数据以metrics的形式暴露出来。

而Presto&Trino的默认监控方式，暂时还没有方便的对接方式，比如打开一个开关就会有一个metrics接口之类。而是使用它的WebUI，或者去用WebUI中对象的接口去监控。

在Prometheus的exporter生态中，监控方式一般有两种，嵌入式与外挂式。简单介绍一下这两种模式



## jvm java agent

比较方便的就是嵌入式的java agent方式，而且presto与trino的配置基本类似。

trino 配置参考

```
-server
-Xmx16G
-XX:InitialRAMPercentage=80
-XX:MaxRAMPercentage=80
-XX:G1HeapRegionSize=32M
-XX:+ExplicitGCInvokesConcurrent
-XX:+ExitOnOutOfMemoryError
-XX:+HeapDumpOnOutOfMemoryError
-XX:-OmitStackTraceInFastThrow
-XX:ReservedCodeCacheSize=512M
-XX:PerMethodRecompilationCutoff=10000
-XX:PerBytecodeRecompilationCutoff=10000
-Djdk.attach.allowAttachSelf=true
-Djdk.nio.maxCachedBufferSize=2000000
-XX:+UnlockDiagnosticVMOptions
-XX:+UseAESCTRIntrinsics
# Disable Preventive GC for performance reasons (JDK-8293861)
-XX:-G1UsePreventiveGC
-javaagent:/usr/lib/trino/jmx/jmx_prometheus_javaagent-0.16.1.jar=监控暴露端口:/usr/lib/trino/jmx/trino_exporter.yaml
```

presto 配置参考

```
-server
-Xmx16G
-XX:+UseG1GC
-XX:G1HeapRegionSize=32M
-XX:+UseGCOverheadLimit
-XX:+ExplicitGCInvokesConcurrent
-XX:+HeapDumpOnOutOfMemoryError
-XX:+ExitOnOutOfMemoryError
-javaagent:/usr/lib/presto/jmx/jmx_prometheus_javaagent-0.16.1.jar=监控暴露端口:/usr/lib/presto/jmx/presto_exporter.yaml
```

***_exporter.yaml 配置参考

```

```

配置完成启动后，会在进程中看到相关监控的信息。

trino

```
root      6650     1  1 Apr21 ?        00:25:01 java -cp /data8/trino-server-414/lib/* -server -Xmx64G -XX:-UseBiasedLocking -XX:+UseG1GC -XX:G1HeapRegionSize=32M -XX:+ExplicitGCInvokesConcurrent -XX:+ExitOnOutOfMemoryError -XX:+HeapDumpOnOutOfMemoryError -XX:-OmitStackTraceInFastThrow -XX:ReservedCodeCacheSize=512M -XX:PerMethodRecompilationCutoff=10000 -XX:PerBytecodeRecompilationCutoff=10000 -Djdk.attach.allowAttachSelf=true -Djdk.nio.maxCachedBufferSize=2000000 -Djava.library.path=/usr/lib/hadoop/lib/native:/usr/lib/hadoop/share/hadoop/common/lib:/data8/trino-server-414/hadoop-lzo -javaagent:/data8/bak/jmx_prometheus_javaagent-0.16.1.jar=20081:/data8/bak/trino_exporter.yaml -Dnode.environment=trino_414_test -Dnode.data-dir=/data8/trino-server-414/data -Dlog.levels-file=/data8/trino-server-414/etc/log.properties -Dlog.output-file=/data8/trino-server-414/data/var/log/server.log -Dlog.enable-console=false -Dconfig=/data8/trino-server-414/etc/config.properties io.trino.server.TrinoServer
```

presto

```
root     31049     1 99 10:29 ?        00:00:09 java -cp /data8/presto-server-0.279/lib/* -server -Xmx64G -XX:+UseG1GC -XX:G1HeapRegionSize=32M -XX:+UseGCOverheadLimit -XX:+ExplicitGCInvokesConcurrent -XX:+HeapDumpOnOutOfMemoryError -XX:+ExitOnOutOfMemoryError -Djava.library.path=/usr/lib/hadoop/lib/native:/usr/lib/hadoop/lzo/lib:/data8/presto-server-0.279/hadoop-lzo -javaagent:/data8/bak/jmx_prometheus_javaagent-0.16.1.jar=20082:/data8/bak/trino_exporter.yaml -Dnode.environment=test -Dnode.id=cdp_presto_test -Dnode.data-dir=/data8/presto-server-0.279/data -Dlog.levels-file=/data8/presto-server-0.279/etc/log.properties -Dlog.output-file=/data8/presto-server-0.279/data/var/log/server.log -Dlog.enable-console=false -Dconfig=/data8/presto-server-0.279/etc/config.properties com.facebook.presto.server.PrestoServer
```

此时我们再来请求一下监控暴露端口，即可得到metrics格式的监控数据。注意到已经有了5000~6000个监控指标，大部分的关键指标都涵盖了，能够满足很多监控场景了。

```
```

## 监控分层


## 指标解释

### 元数据缓存

trino

```
trino_plugin_hive_metastore_cache_CachingHiveMetastore_PartitionStats_MissRate{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_TablePrivilegesStats_RequestCount{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_TableWithParameterStats_MissRate{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_RolesStats_MissRate{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_PartitionStatisticsStats_MissRate{name="hive_bak",} 0.5
trino_plugin_hive_metastore_cache_CachingHiveMetastore_TablePrivilegesStats_MissRate{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_PartitionStatisticsStats_RequestCount{name="hive_bak",} 16.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_TableStats_HitRate{name="hive_bak",} 1.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_TableStats_MissRate{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_RoleGrantsStats_HitRate{name="hive_bak",} 1.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_PartitionStatisticsStats_HitRate{name="hive_bak",} 0.5
trino_plugin_hive_metastore_cache_CachingHiveMetastore_TableStatisticsStats_HitRate{name="hive_bak",} 0.4444444444444444
trino_plugin_hive_metastore_cache_CachingHiveMetastore_PartitionStats_RequestCount{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_DatabaseStats_MissRate{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_ConfigValuesStats_RequestCount{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_ViewNamesStats_RequestCount{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_RoleGrantsStats_RequestCount{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_ViewNamesStats_MissRate{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_DatabaseNamesStats_HitRate{name="hive_bak",} 1.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_TableNamesStats_HitRate{name="hive_bak",} 1.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_ViewNamesStats_HitRate{name="hive_bak",} 1.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_TableStatisticsStats_MissRate{name="hive_bak",} 0.5555555555555556
trino_plugin_hive_metastore_cache_CachingHiveMetastore_TableWithParameterStats_HitRate{name="hive_bak",} 1.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_DatabaseStats_HitRate{name="hive_bak",} 1.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_RolesStats_RequestCount{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_PartitionFilterStats_HitRate{name="hive_bak",} 1.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_TablePrivilegesStats_HitRate{name="hive_bak",} 1.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_TableNamesStats_MissRate{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_RolesStats_HitRate{name="hive_bak",} 1.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_ConfigValuesStats_HitRate{name="hive_bak",} 1.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_DatabaseNamesStats_MissRate{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_TableNamesStats_RequestCount{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_GrantedPrincipalsStats_MissRate{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_ConfigValuesStats_MissRate{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_PartitionFilterStats_RequestCount{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_PartitionFilterStats_MissRate{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_RoleGrantsStats_MissRate{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_TableWithParameterStats_RequestCount{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_DatabaseNamesStats_RequestCount{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_GrantedPrincipalsStats_RequestCount{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_DatabaseStats_RequestCount{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_GrantedPrincipalsStats_HitRate{name="hive_bak",} 1.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_TableStats_RequestCount{name="hive_bak",} 0.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_PartitionStats_HitRate{name="hive_bak",} 1.0
trino_plugin_hive_metastore_cache_CachingHiveMetastore_TableStatisticsStats_RequestCount{name="hive_bak",} 9.0
```

presto

```
com_facebook_presto_hive_metastore_MetastoreCacheStats_PartitionsWithColumnCountGreaterThanThreshold_FiveMinute_Count{name="hive",} 0.0
com_facebook_presto_hive_metastore_MetastoreCacheStats_PartitionCacheHit{name="hive",} 0.0
com_facebook_presto_hive_metastore_MetastoreCacheStats_PartitionsWithColumnCountGreaterThanThreshold_FiveMinute_Rate{name="hive",} 0.0
com_facebook_presto_hive_metastore_MetastoreCacheStats_PartitionsWithColumnCountGreaterThanThreshold_FifteenMinute_Count{name="hive",} 0.0
com_facebook_presto_hive_metastore_MetastoreCacheStats_PartitionsWithColumnCountGreaterThanThreshold_TotalCount{name="hive",} 0.0
com_facebook_presto_hive_metastore_MetastoreCacheStats_PartitionCacheMiss{name="hive",} 0.0
com_facebook_presto_hive_metastore_MetastoreCacheStats_PartitionsWithColumnCountGreaterThanThreshold_OneMinute_Rate{name="hive",} 0.0
com_facebook_presto_hive_metastore_MetastoreCacheStats_PartitionCacheSize{name="hive",} 0.0
com_facebook_presto_hive_metastore_MetastoreCacheStats_PartitionCacheEviction{name="hive",} 0.0
com_facebook_presto_hive_metastore_MetastoreCacheStats_PartitionsWithColumnCountGreaterThanThreshold_OneMinute_Count{name="hive",} 0.0
com_facebook_presto_hive_metastore_MetastoreCacheStats_PartitionsWithColumnCountGreaterThanThreshold_FifteenMinute_Rate{name="hive",} 0.0
```

## 绘图参考




