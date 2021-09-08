# RegionServer
## regionserver 构成
&emsp;&emsp;RegionServer是hbase最核心的部分，负责用户数据的读写。主要由Hlog、BlockCache、Region、Store、MemCache、Hfile。  
<center><B>Regionserver结构图</B></center> 
![图片](./hbase_regionserver.jpg)