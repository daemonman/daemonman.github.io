### 命令行查看hbase配置  
+ hbase shell中执行`@shell.hbase.configuration.get("hbase.local.dir")`,其中hbase.local.dir为需要获取的变量
+ RowPreFixFilter  
`scan 'tablename',{ROWPREFIXFILTER=>'xxxxx',LIMIT=>1}`
+ SingleColumnValueFilter  
`scan 'tablename',{FILTER=>"SingleColumnValueFilter('column','qualify',=,'binary:111',true,true)",LIMIT=>10}`  
参数说明：总共6个参数，列族名、列名、比较器、比较值、filterIfColumnMissing（true表示没有包含的列直接过滤，默认false无法过滤不包含这个字段的列）、setLatestVersionOnly（true表示只查看最新的版本）


