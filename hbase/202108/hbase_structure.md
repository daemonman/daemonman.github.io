# Hbase 初识
## 特点
BigTable论文中称BigTable为"sparse，distributed，persistent multidimensional sorted map"。本质上是一个map，具备稀疏、分布式、持久化、多维、有序等特点。map都是有key和value构成
hbase中map的key是有rowkey、column family、qualifier、type、timestamp。value即为cell的值。

+ 稀疏  
  &emsp;空值不占用存储空间
+ 分布式  
  &emsp;集群部署
+ 持久化
+ &emsp;数据最终会持久化到hdfs上  
+ 多维  
  &emsp;key由多个元素构成
+ 有序  
  &emsp;三维有序：rowkey有序；rowkey相同，column family:qulifier有序；column相同，timestamp有序。
+ 列簇存储  
  &emsp;行存：在一行一行检索有优势，但如果查询结果只有其中部分列，会产生大量无效的IO。通常传统的关系型数据库采取行存  
  &emsp;列存：将一列数据存储在一起，常见kudu、parquet都是列存，列存在查询某些列时十分高效。但若获取一行数据，需要多次IO，不适合整行查询。    
  &emsp;列簇存储：hbase采用这种存储方式，可以说是列和行的中间态，列簇能保证在同一列簇的多列可高效检索。但建议实际运用中，列簇不宜设置过多。   
## 架构
+ 结构视图  
 &emsp; 一个完整的Hbase集群依赖关键组件：Zookeeper 、Hmaster、RegionServer、HDFS，用户通过使用Client访问Hbase。
  ![图1](./res/hbase-structure.jpg )
+ Client  
&emsp; 客户端可以通过Shell、Api、Thrift/Rest Api、Mapreduce等方式访问Hbase。支持DDL、DML等常用操作，通过Thrift/Rest 支持非java语言，通过Mapreduce批量读写。Client首先需要获取元数据，然后去对应的RegionServer进行相关数据读写。为提高性能，Client会缓存元数据信息，但如果发生过Region的迁移，Client需要重新获取元数据信息。
+ Zookeeper  
&emsp;从Google的chubby而来，主要用于协调管理分布式程序。在Hbase系统中，Zookeeper具备巨大的作用。  
 1. 实现Hmaster的高可用。监听Hmaster服务状态，确保Hmaster服务政策。
  2.  元数据管理。例如作为hbase:meta表的元数据存储。
  3. RegionServer故障恢复。监听RegionServer服务状态，确定RegionServer可用。
  4. 分布式锁。比如为了防止Hbase DDl并发操作，用作表级别的锁。     
+ Master
 1. 处理客户端的各类管理请求。表的增删改查，split，compaction等。
 2. RegionServer的管理。Region的负载均衡，宕机恢复，region迁移等。
 3. Hlog和Hfile定期清理。清理过期的Hlog、Hfile。  
+ RegionServer  
&emsp;Hbase最核心的模块，直接处理客户端的各类IO请求。主要有Hlog、BlockCache、多个Region构成。
 1. Hlog 主要用于实现Hbase高可用，Hbase基于LSM树（log-structured-merge-tree）。数据写入首先写入Hlog，然后写入Region的memStore，最后满足flush条件后flush到磁盘成hfile。在做hbase多集群数据同步的时候会依赖hlog。
 2. BlockCache，主要用于缓存数据Block（非hdfs的block），是指hbase的数据块64k（默认）大小，存储有序kv值。
 3. Region，Hbase的数据存放Region切分，一个Region由Store构成，Store按列族切分，Store内部有一个MemStore和多个Hfile构成。MemStore（默认128M）是Region的内存空间，缓存写入的数据，定期刷到hdfs形成Hfile。
+ HDFS  
&emsp;分布式文件系统，作为Hbase实际持久化的存储。

## 应用场景
+ 优点  
 1. 容量巨大：理论上支持千亿行、百万列存储。
 2. 良好的扩展性：扩容hdfs节点即可增大存储能力，扩容RegionServer即可增大hbase读写服务能力。
 3. 稀疏存储：空值不存储，节约存储资源。
 4. 高性能：不会随着数据量增大，性能下降。对于小范围扫描、随机单点查询有强悍的性能表现。
 5. 多版本：Hbase的kv数据可以同时存在多个版本，用户可以根据需要查询历史数据。
 6.  TTL:有数据老化的机制，不要动态清除过期数据。
 7. hadoop生态友好。
+  缺点
 1.  对于聚合操作（groupby、join）不支持。虽然可借助Phoneix、Spark实现，但在OLTP场景性能不如随机查询。
 2. 不支持二级索引。需要用户在设计上解决二级索引，或者借助Phoneix实现。
 3. 不支持跨行事务。只能保障Hbase单行事务，需要用户在业务场景上多做考虑。
 
## 参考文献

参考太多，例举不完
