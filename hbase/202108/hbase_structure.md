# Hbase 初识
## hbase特点
BigTable论文中称BigTable为"sparse，distributed，persistent multidimensional sorted map"。本质上是一个map，具备稀疏、分布式、持久化、多维、有序等特点。map都是有key和value构成
hbase中map的key是有rowkey、column famili、qualifier、type、timestamp。value即为cell的值。

+ 稀疏  
  &emsp;空值不占用存储空间
+ 分布式  
  &emsp;集群部署
+ 持久化  
+ 多维  
  &emsp;key由多个元素构成
+ 有序  
  &emsp;三维有序：rowkey有序；rowkey相同，column familiy:qulifier有序；column相同，timestamp有序。
+ 列存  
## hbase架构
## hbase应用场景
## 参考文献

