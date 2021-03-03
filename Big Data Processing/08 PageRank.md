# PageRank

[toc]

## 基本设计思想

若一个网页：

+ 有很多网页链接到它
+ 有高质量网页链接到它

则该网页拥有高PR值。一个网页PR值越高，则网页越受欢迎。



## 简化模型

#### 模型

对任意网页$P_i$，其PageRank值为：$R\left(P_{i}\right)=\sum_{P_{j} \in B_{i}} \frac{R\left(P_{j}\right)}{L_{j}}$

+ 其中$B_i$为所有链接到网页i的网页集合，$L_j$为网页j对外的链接数

#### 矩阵表示

定义超链接矩阵 $\mathbf{H}_{i j}=\left\{\begin{array}{cc}1 / L_{j} & \text { if } P_{j} \in B_{i} \\ 0 & \text { otherwise }\end{array}\right.$，$\mathbf{R}=\left[R\left(P_{i}\right)\right]$

有$R=HR$，$R$为$H$的特征向量，特征值为1

#### 缺陷

+ Rank leak：一个独立的网页如果没有外出的链接就产生排名泄漏
  + 原因：独立网页A假设被B链接，A的PR值仅由B提供，B所在的连通分量不断将PR值分给A，最终B所在的连通分量PR值变为0，A的PR值也变为0
  + 解决方法：
    1. 将无出度的节点递归地从图中去掉，待其他节点计算完毕后再加上
    2. 对无出度的节点添加一条边，指向那些指向它的顶点

+ Rank sink：整个网页图 中点 若有网页没有入度链接，如节点A所示， 所示，其所产生的贡献会被由节点 其所产生的贡献会被由节点B 、C 、D构成的强联通分量“吞噬” 构成的强联通分量“吞噬”掉，就会产生排名下沉，节点 掉，就会产生排名下沉，节点A 的PR 值在迭代后会趋向于0

  ![image-20200603151908194](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200603151908194.png)

#### 随机浏览模型

+ 基础

  <img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200604005031967.png" alt="image-20200604005031967" style="zoom: 50%;" />

  + 网页间原本就有的链接关系跳转概率为$d$，按照概率$1-d$随机浏览
  + 矩阵表示：$H^{\prime}=d * H+(1-d) *[1 / N]_{N \times N}$，则$R=H'R$，且R存在唯一解
  + 用邻接表来表示网页之间的链接关系：$P R\left(p_{i}\right)=\frac{1-d}{N}+d \sum_{p_{j} \in M\left(p_{i}\right)} \frac{P R\left(p_{j}\right)}{L\left(p_{j}\right)}$
  + 优点：
    + 更符合用户行为
    + 一定程度上解决了Rank sink 和 Rank leak问题
    + 保证PageRank存在唯一值

+ MapReduce实现PageRank：

  原始数据集：网页名, 所有链出网页名

  1. GraphBuilder：简历网页间的超链接图

     + Map: 输出<URL, (PR_init, link_list)>		# PR_init为PR初始值，link_list为网页出度列表
     + Reduce: 不作任何处理，直接输出<URL, (PR_init, link_list)>

  2. RageRanklter：迭代计算各网站的PR值

     对上阶段的<URL, (cur_rank, link_list)>产生两种<key, value>

     + Map: 

       Type 1. for each u in link_list, 输出<u, cur_rank/|link_list|>

       Type 2. 传递每个网页的链接信息<URL, link_list>

     + Reduce: 

       + 对<URL, url_list>为当前URL的链出信息
       + <URL, val>为当前URL的链入网页对其贡献的PR值
       + 输出：<URL, (new_rank, url_list)>

     $\operatorname{PR}(A)=(1-d) / N+d\left(P R\left(T_{1}\right) / C\left(T_{1}\right)+\ldots+P R\left(T_{n}\right) / C\left(T_{n}\right)\right)$

  3. RankViewer：按PR值排序输出

     + Key: 网页名
     + Value: PR值

  

  终止条件：PR值不再改变/PR值排序不再改变/迭代固定次数



# ？

+ PR矩阵表示没有很清楚