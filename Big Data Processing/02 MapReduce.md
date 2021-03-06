## 02 MapReduce

[toc]

### 大数据处理：分而治之

+ 不可拆分的 /相互间有依赖关系的数据 的计算任务无法进行并行计算
+ 具有相同计算过程 且 数据块间不存在数据依赖关系的数据块，提高处理速度的最佳方法就是并行计算

+ 大数据并行化计算：将大数据计算任务划分为多个子任务

### 构建抽象模型：Map与Reduce

+ Map：对一组数据元素进行某种重复性处理（有多少键值对就调用多少次）
  + 键值对：(k1;v1) -> [(k2;v2)]
+ Reduce：对Map的中间结果进行进一步的整理
  + (k2;[v2]) ->  [(k3;v3)]
    + map输出的键值对[(k2;v2)]进行合并处理，将主键相同的不同数值合并到一个列表[v2]中，故reduce的输入为(k2;[v2])（不一定发送到同一reducer上的数据的key都相同）
+ <img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200306000641151.png" alt="image-20200306000641151" style="zoom:50%;" />

### 自动并行化与隐藏底层细节

+ MapReduce提供的主要功能*
  + 任务调度：划分map/reduce节点；监控节点状态；节点同步控制；计算性能优化处理
  + 数据/代码定位：基本原则：本地化数据处理（尽可能处理本地分布存储的数据）
  + 出错处理：检测出错节点，重新分发任务
  + 分布式数据存储与文件管理
  + Combiner & Partitioner：切分合并+发送给哪个reducer
    + 中间数据进入reduce节点前需要合并(combine)处理，map节点输出需要适当地划分(partitioner)处理，保证相关数据发送到同一个reduce节点
    + <img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200305232418886.png" alt="image-20200305232418886" style="zoom:80%;" />

### MapReduce的主要设计思想与特点*

+ 横向扩展，而非纵向扩展

+ 失效被认为是常态

+ 把处理向数据迁移：首先计算本地存储数据，再就近寻找可用计算节点，并将数据传送到该可用计算节点

+ 顺序处理数据、避免随机访问数据

  + 磁盘的顺序访问和随机访问（因为可能要进行数据移动）在性能上有巨大差异

+ 为应用开发者屏蔽系统层细节

+ 平滑的(Seamless)可拓展性

  + 数据扩展：软件算法随数据规模的扩大而持续有效，性能的下降程度与数据规模的扩大呈线性关系
  + 系统规模扩展：算法计算性能应能随节点数的增加保持线性程度的增长

  

