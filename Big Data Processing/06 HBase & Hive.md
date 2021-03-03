# HBase & Hive

[toc]

## HBase基本工作原理

+ HBase数据模型
  + 逻辑模型：分布式多维表，表中数据通过
    + 一个行关键字（唯一）、一个列关键字、一个时间戳 来进行索引和查询定位
    + 按列存储，空列不存

+ 基本架构
  + 1个MasterServer+1组子表数据区服务器RegionServer
  + 底层数据存在HDFS

+ HBase shell 操作
  + 建表Create：`create '表名','RowKey','列族1'，...,'列族n'` 
  + 插入put：`put 'students','001','Description:Name','Li Lei'` 
  + 显示描述表信息Describe：`describe students`
  + 扫描数据（显示表内容） `scan students,{COLUMNS='Courses:'}` 
  + Disable：禁用写，比如修改表的样式的时候可以用（对应enable）
    + disable：在zookeeper中做记录，并将表的全部region下线



## Hive 基本工作原理

SQL-生成->MapReduce代码

+ 组成模块
  + HiveQL：Hive的数据查询语言，与SQL非常类似
  + Driver：执行驱动程序。用于将各个部分组成一个执行系统，包括会话处理，查询获取，执行驱动等
  + Compiler：将HiveQL编译成中间表示
  + Execution Engine：执行引擎。在Drive的驱动下，具体完成执行操作，包括执行MapReduce，HDFS，元数据操作
  + Metastore：用以存储元数据。一系列的表格组成一个命名空间，关于这个命名的描述信息会保存在Metastore的空间中
+ 数据模型
  + tabels：Hive的数据模型由数据表组成
  + partitions：数据表可以按照一定的规则进行划分Partition
  + Buckets：数据存储的桶。用于hash划分

