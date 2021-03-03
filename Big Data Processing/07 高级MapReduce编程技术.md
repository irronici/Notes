# 高级MapReduce编程技术

[toc]

### 复合键值对的使用

+ 目的：要对（key, {value}）中的key及value进行排序
  + 将value中需要排序的部分加入key，定值Partitioner，让同一key值的键值对分发到同一个Reducer节点上

+ 目的：（计算问题会产生大量键值对）减少键值对的传输和排序开销
  + 将小的键值对合并为一些大的键值对（pairs(单键值)->stripes(多键值)）



### 用户自定义数据类型

需要实现Writable接口，若作为Key或者需要比较大小时则需要实现WritableComparable接口
+ 接口要实现`write(DataOutput dataOutput)`, `void readFields(DataInput dataInput)`, `compareTo()`（compareTo是在实现WritableComparable接口需要实现）



### 用户自定义Partitioner&Combiner

+ 定值Partitioner使用场景
  + 排序（默认情况是HashPartitioner，不是将小于某threshold的放到一个Reducer）
  + 复合键值对的处理
+ Partitioner作用：决定当前键值对发给哪个Reducer

+ Combiner：本地的Reducer



### 迭代MapReduce计算