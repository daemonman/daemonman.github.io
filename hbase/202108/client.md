# hbase客户端
## 编程语言支持  
  支持java、C++/C、Python、shell多语言。 非java语言推荐ThriftServer访问。
  
## 需要的配置文件
  hbase-site.xml、core-site.xml、hdfs-site.xml。
  
## Connection和table
  一个进程需要为一个集群建立一个Connection即可，不要连接池。一个Connection可以创建多个table对象（执行完关闭）。
  hbase 1.X以及以前的版本，table不是线程安全；hbase 2.X之后线程安全，放心使用。

## 元数据hbase:meta
  元数据表存储hbase表的元数据性信息，存储表的元数据信息。 只会在一个Region上 。
  
+ rowkey  
  由tablename+startRow+timestamp+EncodeName（前三个MD5Hex值）  
+ column    
  info:regioninfo由regionname、encodename、startkey、endkey 4部分构成    
 info:seqnumDuringOpen  Region打开时候的sequenceId  
 info：server 所在的regionServer  
 info:serverstartcode 在regionserver的启动时间  
 info:state region状态  
 info:sn regionserver、端口、时间戳  
 
+ 查询region所在位置，通过REVERSE SCAN
+ 缓存元素据信息,防止集群压力过大。MetaCache(ConcurrentNavigableMap)
   
## Scan的关键方法
   通过table.getScanner(scan),获取通过scanner.next().关键参数
   
   + .caching 操作放在cache队列最大数据量result  
   + .batch 最多获取cell个数
   + .allowPartial 是否允许拿到一行的部分cell
   + .maxResultSize 单次RPC最多拿到的字节数
   +  .limit 最大查询条数
   
## 重要配置
   + hbase.rpc.timeout: 单次rpc的超时时间，默认60000ms
   + hbase.client.retries.number: 最多失败的次数。 默认35次
   + hbase.client.pause 表示两次RPC之间重试的休眠时间。 默认100ms
   + hbase.client.operation.timeout:单次API的超时时间。默认12000ms
   
## Scan的Filter
   + PrefixFilter 以rowkey作为前缀，比较暴力。指定startrow，否则扫描范围过大。 即使如此建议使用rowprefixfilter
   + PageFilter 只能做region内有效，如果设置3000，但是一旦扫描了3个region可能导致9000数据返回。
   + SingleColmunValueFilter 。可以设置filterIfMissing ,不读取额外数据。 但是要注意不建议和其他过滤器一起组合使用。

## 数据写入的方式
   + table.put(put)   
   + table.put(List<put>) 减少rpc但是无法保证事务
   + bulkload 直接生成HFILE，使用loadHFile装载，load完成后会进行compaction
   
## 客户端延迟的可能原因
   + 客户端、服务端 Full GC
   + 获取了大量数据。MultiActionResultTooLarge,服务端限制请求Block总字节数。
   +  compaction   
    