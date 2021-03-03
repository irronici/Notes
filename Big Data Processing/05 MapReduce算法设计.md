# 05 MapReduce算法设计

[toc]

## MapReduce处理流程



## MapReduce排序算法



## 单词同线矩阵算法

+ 输入联想，搜索推荐



## 文档倒排索引算法

+ Inverted Index（倒排索引）是一个数据结构（用于搜索引擎的全文检索）。基于索引结构可以通过给出一个词(term)，能取得含有这个term的文档列表

+ Web Search主要有三个部分：
  + crawling（爬取网页内容）
  + indexing（构造倒排索引）
  + retrieval（根据查询条件对文档进行排序）
    + crawling, indexing是离线的，retrieval是实时的，当用户提出请求时进行





改数据结构的原因

+ 内存不够
+ 可拓展性不够





## Q

+ 为什么是writable？
  + 可进行序列化（转化为字节串）、反序列化（从字节串转换为基础类型）

+ 带词频的文档倒排算法为什么要改数据结构？
  + 为了降低map&reduce的传输/存储开销，应用combiner在每次mapper结束后进行一次自动排序合并
  + 会产生新的问题：进行shuffle并传送给reducer时，同一个term的键值对可能会被放到不同的reducer，故要定值partitioner来解决这个问题

