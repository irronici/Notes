## 03 Google & Hadoop MapReduce

[toc]

### Google MapReduce工作基本原理

 <img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200306002358200.png" alt="image-20200306002358200" style="zoom:80%;" />

1. 将待处理任务划分为大小相同的数据块（如64MB/128MB）
2. 用户提交任务给主节点
3. 主节点为任务程序寻找可用Map节点，并传送数据与程序
4. 主节点为任务程序寻找可用Reduce节点，并传送程序
5. 主节点启动每个Map节点执行程序（Map节点尽可能读取本地数据进行计算）
6. Map节点处理数据块并将中间结果存放在本地；通知主节点任务完成并告知结果数据存放位置
   + 之所以不直接丢给对应的Reduce节点是因为
     + Reduce节点可能在忙
     + 万一Reduce节点计算到一般故障了Map节点又要重新算一遍

7. Map节点全部计算完毕后启动Reduce节点，Reduce节点读取对应数据
8. Reduce将计算结果汇总并输出

### 错误处理（见ppt）



### MapReduce分布式文件系统GFS

+ Google GFS是一个基于分布式集群的大型分布式文件系统，为MapReduce提供底层数据存储和数据可靠性支持
+ GFS是一个构建在分布节点本地文件系统之上的一个逻辑上的文件系统。它将数据存储在物理上分布的每个节点上，但通过GFS将整个数据形成一个逻辑上整体的文件
  + 廉价本地磁盘分布存储。容量可随节点数的增加而增加
  + 多数据自动备份解决可靠性。用普通的磁盘，把磁盘数据出错看做常态。用自动多数据备份存储解决数据存储可靠性
  + 为上层MapReduce计算框架提供支持。上层框架不需要考虑低层数据存储/容错问题

### Google GFS的基本框架和工作原理

![img](http://oserror.com/images/gfs_architecture.png)

+ GFS Master 存储元数据（3种）：★
  + 命名空间，即整个分布式文件系统的目录结构
    + 可用于查找文件
  + Chunk与文件名的映射表 (文件被分成固定大小的Chunk来存储)
  + Chunk副本的位置信息，每一个Chunk默认有三个副本
    + 注：前两种元数据通过日志进行错误恢复；第三种元数据直接保存在ChunkServer上，Master启动或ChunkServer注册时自动完成其在Chunk Server上元数据的生成

+ ChunkServer

  用来保存大量数据的数据服务器

  + GFS中每个数据块（Chunk）划分缺省为64MB，每个数据块会在3（缺省情况）个不同的地方复制副本，对每个Chunk当3个副本都更新成功时，才认为数据保存成功
  + 每个本地Chunk副本会被再划分为64KB的子块，每个子块有32bits的校验和，读数据时会检查校验和

+ 数据访问过程

  > GFS架构中只有单个GFS master，这种架构的好处是设计和实现简单，例如，实现负载均衡时可以利用master上存储的全局的信息来做决策。

  1. 程序运行前，数据已经存储在GFS文件系统中。程序实行时 应用程序告诉 GFS Server 要访问的文件名/数据块索引是什么
  2. GFS Server根据文件名，在文件目录中查找并定位该文件/数据块，并查找其具体在哪些ChunkServer上
  3. GFS Server返回具体Chunk位置信息，应用程序**直接访问**相应的Chunk Server直接读取数据进行计算处理

  + **特点：**应用程序访问具体数据时不需要经过GFS Master，避免Master成为访问瓶颈；一个大数据会存储在ChunkServer中，应用程序可以实现并发访问

### 分布式结构化数据表BigTable

> 解释：为什么要有BigTable'
>
> GFS作为一个文件系统难以提供对结构化数据的存储和访问管理
> BigTable提供了一定粒度的结构化数据操作能力，主要解决一些大型媒体数据（Web文档，图片等）的结构化存储问题。

BigTable将表中数据一律视为字节串，具体数据结构的实现用户自行定义
BigTable数据模型，通过 **行关键字、列关键字、时间戳** 进行索引和查询定位

+ 行名/行关键字：反向URL（大小不超过64KB的任意字节串）。表中数据根据行关键字进行排序
  + URL倒排的好处：
    + 同一地址的网页会被存储在连续的位置
    + 便于压缩数据
+ 列族：列关键字组成的集合
  + 同一列族中数据属于同一类型
  + 列关键字表示为 `族名:列名`
+ 时间戳：表中可以包含一个数据项的不同版本，不同版本需要用时间戳来区分
  + BigTable提供两种设置以简化不同版本的数据管理
    + 保留最近n个版本数据
    + 保留xx时间内的所有版本数据

### BigTable基本框架

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200319194100858.png" alt="image-20200319194100858" style="zoom:67%;" />

主服务器：新子表空间分配、子表监控、负载均衡

子表服务器：存储&管理子表

+ 基本存储结构SSTable

  <img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200809162818809.png" alt="image-20200809162818809" style="zoom: 50%;" />

  + 所有SSTable文件存储在GFS上，对应GFS上的一个Chunk
  + SSTable中的数据进一步划分为64KB的子块，并使用一个索引来维护这1000个子块的位置信息

+ 子表数据格式：一个子表服务器上的子表会由很多SSTable组成，一个SSTable可以由不同的子表共享

+ 子表寻址：以三级B+数进行索引



### Hadoop MapReduce

InputFormat: 定义用什么方法读文件，默认是按行读

RecordReader(RR): 

Split分成很多：提高并发度

![image-20200319204312770](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200319204312770.png)

+ InputFormat：定义了数据文件如何进行分割和读取
  + 选择输入文件、定义InputSplit以将文件划分为不同任务
  + 文件输入格式：文本、键值对、二进制
+ InputSplit：输入数据到单个Map任务
+ RecordReader：将数据记录转换为键值对，并传给Mapper
+ Mapper：处理数据
+ Combiner：合并相同key的键值对，减少Partitioning的数据通信开销（可以看作一个**本地的Reducer**）
+ Partitioner & Shuffle：将处理后的数据根据键值对传到对应的Reducer上



### HDFS



