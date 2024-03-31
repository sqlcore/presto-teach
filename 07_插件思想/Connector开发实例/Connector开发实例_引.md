# Connector开发实例_引

[返回首页](../README.md)

---

Presto & Trino 得益于优秀的插件思想设计，所以在开发新的数据源 Connector 时会很方便。

学习 Connector 开发前，我们可以参考一下官方文档。

- https://prestodb.io/docs/0.285/develop/connectors.html
- https://trino.io/docs/current/develop/connectors.html

早期 Trino 与 Presto 的 Connector 开发还是比较像的，Trino 在后期的版本已经进行了多次迭代，开发方式与 Presto 稍微有点区别了。不会为了了解 Connector 的开发思想我是建议可以先从 Presto 学起。

后续的讲解我们先从 Presto 开始，然后再讲解 Trino。


## Presto Connector 的几个基础概念

1.  **ConnectorSplit**：这是Presto查询执行的基本单位。一个查询被分解成多个任务，每个任务处理一个或多个splits。每个`ConnectorSplit`代表了数据源中的一部分数据，它包含了足够的信息来定位和处理这部分数据。例如，在处理HDFS数据时，一个split可能代表一个文件块。
    
2.  **ConnectorFactory**：这是一个工厂类，用于创建Connector实例。Presto使用`ConnectorFactory`来实例化具体的Connector，这个过程通常在Presto启动时完成。每个Connector都需要提供一个`ConnectorFactory`来允许Presto服务器动态加载和配置该Connector。
    
3.  **ConnectorMetadata**：这个接口负责处理所有与元数据相关的操作。这包括获取表格、列、数据类型等信息。`ConnectorMetadata`是Presto与数据源之间的主要接口，用于获取关于数据源结构的信息。
    
4.  **ConnectorSplitManager**：此组件负责将Presto查询分解成多个splits。`ConnectorSplitManager`根据查询的需求和数据源的特性，决定如何最有效地分割数据。这个过程对于查询性能至关重要。
    
5.  **ConnectorRecordSetProvider**：这个接口用于提供对特定split中数据的访问。它负责创建`RecordSet`实例，后者封装了对数据的实际访问。`RecordSet`提供了一种迭代访问split中行数据的方式。