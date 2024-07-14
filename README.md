# presto-teach

这是一个 Presto、Trino 的文档分享项目(排名不分先后)。

Presto 和 Trino 的开发文档，和源码阅读，问题调优等相关资料特别少。于是就想着把自己长时间总结的，搜集的，和实际工作中遇到的问题以及二次开发相关都分享出来。

## 参与贡献注意事项

- [注意事项](./注意事项.md)

## 项目架构

- 01_基础概念与全面了解
    - [论文阅读：Presto_SQL_on_Everything](./01_基础概念与全面了解/论文阅读：Presto_SQL_on_Everything.md)
    - 书籍阅读：Presto实战
    - 书籍阅读：Presto技术内幕
    - 基础框架Airlift
    - 概念设计与角色分布
    - [源码分布](./01_基础概念与全面了解/源码分布.md)
    - [源码开发与阅读环境搭建](./01_基础概念与全面了解/源码开发与阅读环境搭建.md)
    - [Presto类型系统浅入](./01_基础概念与全面了解/Presto类型系统浅入.md)
- 02_基础实践
    - 如何把Presto、Trino跑起来
    - [如何配置访问控制](./02_基础实践/如何配置访问控制.md)
    - 如何配置权限管理
    - 如何使用客户端连接
    - 如何查看WebUI
- 03_通信流程
    - 内部通信方式与流程
    - 服务发现流程
      - [服务发现-主要流程介绍篇](./03_通信流程/服务发现-主要流程介绍篇.md)
    - 客户端与集群的连接通信流程
    - 任务提交流程
- 04_计算流程
    - 解析流程
        - 解析框架Antlr在Presto中的应用
    - 执行计划生成流程
    - 执行计划优化，分布式化
    - 计算模型
    - 调度流程
    - 队列控制流程
- 05_内存管理
    - 内存池的划分与调度
    - 配置中的内存配置与理解
- 06_容错与恢复流程
    - 长时间运行任务的容错实现与讨论 [链接1](https://github.com/trinodb/trino/issues/455) [链接2](https://github.com/prestodb/presto/issues/11241)
- 07_插件思想
    - 了解Presto的插件设计思想
    - Connector的设计与实现思路
    - Connector开发实例
        - [Presto如何实现查询下推_初探](./07_插件思想/Connector开发实例/Presto如何实现查询下推_初探.md)
        - [Connector开发实例_引](./07_插件思想/Connector开发实例/Connector开发实例_引.md)
        - [Presto从零开始实现一个Connector_实现流程简述与测试](./07_插件思想/Connector开发实例/Presto从零开始实现一个Connector_实现流程简述与测试.md)
        - MySQL_Connector
            - [Presto查询Doris问题](./07_插件思想/Connector开发实例/MySQL_Connector/Presto查询Doris问题.md)
- 08_运维与监控
    - [基础监控接口](./08_运维与监控/基础监控接口.md)
    - [对接Prometheus生态](./08_运维与监控/对接Prometheus生态.md)
    - [如何发现当前Presto集群的性能瓶颈_初探](./08_运维与监控/如何发现当前Presto集群的性能瓶颈_初探.md)
    - [如何发现当前Presto集群的性能瓶颈_IO瓶颈](./08_运维与监控/如何发现当前Presto集群的性能瓶颈_IO瓶颈.md)
    - [执行计划Stage颜色介绍](./08_运维与监控/执行计划Stage颜色介绍.md)
    - [Presto的Worker为什么会挂掉](./08_运维与监控/Presto的Worker为什么会挂掉.md)
- 09_其他扩展
    - Presto On Spark [链接1](https://github.com/prestodb/presto/issues/13856) [链接2](https://prestodb.io/docs/current/installation/spark.html?highlight=spark)
    - Presto On TiKV [链接1](https://github.com/marsishandsome/presto/tree/feature/tidb-hackathon-2019) [链接2](https://github.com/tidb-incubator/TiBigData)
    - Presto On hetu [链接1](https://github.com/openlookeng/hetu-core)
- 10_问题总结
  - [where子句中在对char类型进行判断时需要手动补齐空格](./10_问题总结/where子句中在对char类型进行判断时需要手动补齐空格.md)
  - [hive元数据连接超时](./10_问题总结/hive元数据连接超时.md)
  - [jdbc错误](./10_问题总结/jdbc错误.md)
  - [mongodb连接异常MongoTimeoutException](./10_问题总结/mongodb连接异常MongoTimeoutException.md)
  - [worker资源问题](./10_问题总结/worker资源问题.md)
  - [时区问题](./10_问题总结/时区问题.md)
  - [最大查询内存问题](./10_问题总结/最大查询内存问题.md)
  - [资源文件加载错误](./10_问题总结/资源文件加载错误.md)
  - [调试代码的超时限制](./10_问题总结/调试代码的超时限制.md)
  - [Presto单元测试错误NoClassDefFoundError_QueryAnalyzer_引发的版本使用思考](./10_问题总结/Presto单元测试错误NoClassDefFoundError_QueryAnalyzer_引发的版本使用思考.md)
  - [Presto错误格式分区问题](./10_问题总结/Presto错误格式分区问题.md)
  - [Presto中文问题](./10_问题总结/Presto中文问题.md)
- 11_二次开发
  - 隐式转换
  - 基于event-listener实现用户行为日志
  - 实现自定义密码认证
  - udf开发
  - hadoop包兼容
- 12_资料参考(排名不分先后)
  - trino issues与pr 解读
  - trino slack 解读 
  - presto issues与pr 解读
  - [zhihu专栏：presto-cn](https://www.zhihu.com/column/presto-cn)
  - 易观博客
  - [zhihu用户：qw](https://www.zhihu.com/people/qw-qw-72/posts)
  - 分享实践
  - [zhihu专栏：深入浅出Presto：PB级OLAP引擎](https://www.zhihu.com/column/c_1294277883771940864)
  - [个人博客：若飞](http://armsword.com/)
  - [个人博客：马云雷](https://mayunlei.github.io/archives/)
  - [oppo博客](https://www.zhihu.com/org/oppohu-lian-wang-ji-zhu)
  - [learn-bigdata](https://learn-bigdata.incubator.edurt.io/docs/Presto)
  - [个人博客：あらびき日記](https://abicky.net/tag/presto/)