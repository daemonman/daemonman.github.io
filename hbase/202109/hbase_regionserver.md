# RegionServer
## Regionserver总体构成
&emsp;&emsp;RegionServer是hbase最核心的部分，负责用户数据的读写。主要由Hlog、BlockCache、Region、Store、MemCache、Hfile构成。  
<center><B>Regionserver结构图</B></center> 
![图片](./hbase_regionserver.jpg)
如图：一个regionserver包含一个Hlog、一个blockcache、多个Region。一个Region由多个Store构成，一个Store包含一个memstore和多个hfile。hfile作为真实的数据存在Hdfs上。

##Hlog
hlog（wal）