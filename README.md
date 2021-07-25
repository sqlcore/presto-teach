# presto-teach

这是一个presto、trino的文档分享项目。

presto的开发文档，和源码阅读，问题调优等相关资料特别少。于是就想着把自己长时间总结的，搜集的，和实际工作中遇到的问题以及二次开发相关都分享出来。

## 参与贡献注意事项

- [注意事项](./注意事项.md)

## 项目架构

- 1_基础概念与全面了解
    - [论文阅读：Presto_SQL_on_Everything](./1_基础概念与全面了解/论文阅读：Presto_SQL_on_Everything.md)
    - 书籍阅读：Presto实战
    - 书籍阅读：Presto技术内幕
    - 基础框架Airlift
    - 概念设计与角色分布
    - [源码分布](./1_基础概念与全面了解/源码分布.md)
    - [源码开发与阅读环境搭建](./1_基础概念与全面了解/源码开发与阅读环境搭建.md)
- 2_通信流程
    - 内部通信方式与流程
    - 服务发现流程
    - 客户端与集群的连接通信流程
    - 任务提交流程
- 3_计算流程
    - 解析流程
        - 解析框架Antlr在Presto中的应用
    - 执行计划生成流程
    - 执行计划优化，分布式化
    - 调度流程
    - 队列控制流程
- 4_内存管理
    - 内存池的划分与调度
    - 配置中的内存配置与理解
- 4_容错与恢复流程
    - 长时间运行任务的容错实现与讨论 [链接1](https://github.com/trinodb/trino/issues/455) [链接2](https://github.com/prestodb/presto/issues/11241)
- 5_插件思想
    - 了解Presto的插件设计思想
    - 用户自定义函数插件
    - 事件监听器插件
    - 数据类型插件
    - 访问控制插件
    - 资源组插件
- 6_对接存储流程
    - Connector的设计与实现思路
    - Connector开发实例
- 7_运维与监控
    - 基础监控接口
    - 对接Prometheus生态
- 8_其他扩展
    - Presto On Spark [链接1](https://github.com/prestodb/presto/issues/13856) [链接2](https://prestodb.io/docs/current/installation/spark.html?highlight=spark)
    - Presto On TiKV [链接1](https://github.com/marsishandsome/presto/tree/feature/tidb-hackathon-2019) [链接2](https://github.com/tidb-incubator/TiBigData)
    - Presto On hetu [链接1](https://github.com/openlookeng/hetu-core)
- 9_问题总结
  - [where子句中在对char类型进行判断时需要手动补齐空格](./9_问题总结/where子句中在对char类型进行判断时需要手动补齐空格.md)
  - [hive元数据连接超时](./9_问题总结/hive元数据连接超时.md)
  - [jdbc错误](./9_问题总结/jdbc错误.md)
  - [mongodb连接异常MongoTimeoutException](./9_问题总结/mongodb连接异常MongoTimeoutException.md)
  - [worker资源问题](./9_问题总结/worker资源问题.md)
  - [时区问题](./9_问题总结/时区问题.md)
  - [最大查询内存问题](./9_问题总结/最大查询内存问题.md)
  - [资源文件加载错误](./9_问题总结/资源文件加载错误.md)
  - [调试代码的超时限制](./9_问题总结/调试代码的超时限制.md)
- 10_二次开发
  - 隐式转换
  - 基于event-listener实现用户行为日志
- 11_资料收集(排名不分先后)
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