# SQL

[TOC]

### 使用的两种方式

+ 自含式：独立的交互式命令行语言
+ 嵌入式：嵌入到高级语言中使用

### SQL数据定义

+ 基表创建

  ```sql
  CREATE TABLE tablename (
  colname datatype [ NOT NULL ]
  { ,colname datatype [ NOT NULL ] }
  );
  
  eg:
  create table Course(
  id int not null primary key,
  title char(20) not null,
  dept_name char(2),
  credit int
  );
  ```

+ 基表修改

```sql
ALTER TABLE <基表名> ADD  <列名> <数据类型>;
ALTER TABLE <基表名> DROP <列名>;
ALTER TABLE <基表名> MODIFY  <列名> <数据类型>;

eg:
alter table Student add age smallint;
```

+ 基表删除

```sql
DROP TABLE <基表名>;
```

### SQL数据操纵

+ 数据查询

  + 映像语句：

    <img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20191227173850731.png" alt="image-20191227173850731" style="zoom:65%;" />

    注：**GROUP BY字句会去除重复项**
    
  + 目标字句：**SELECT * | colname { , colname ... }**
    + 可以用`AS`给属性/表重命名: `x AS y`
    + 可以用`distinct`消除重复元组
    
  + 范围子句：**FROM tablename { , tablename ... }**
  
    + tablename可以用`AS`定义一个别名
  
  + 条件字句：**WHERE search_condition**
  
    + 查询条件，自然连接
  
  + 常用谓词
  
    <img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20191227193346581.png" alt="image-20191227193346581" style="zoom:60%;" />
  
    + like谓词：column [**NOT**] **LIKE** val1 [ **ESCAPE** val2 ]
  
      + _：匹配任意一个字符
  
      + %：匹配任意一个字符串（包括空串）
  
      + 其他字符匹配自身
  
      + 紧跟在**ESCAPE**之后的字符的"_"或"%"不再是通配符，是其自身
  
        + eg：查询课程名中含有百分号的课程号
  
          ```
          SELECT cno
          FROM C
          WHERE cn LIKE ‘%A%%’ ESCAPE ‘A’;
          ```
  
  + 结果排序：**ORDER BY <列名> [ ASC | DESC ] { , … }**
  
  
  
+ 子查询与集合谓词

  + 子查询中不可以使用Order By，除非指定top n

  + 集合谓词

    + IN：一个标量是否在一个集合内

    + ANY/ALL：子查询结果中的任何一个值/所有值

    + EXIST：强调exist包围的子句是否为真（是否有返回集）

      + 子句一般跟的是Select * ...

      ```
      SELECT S.sn
      FROM S
      WHERE EXISTS (
          SELECT *
          FROM SC
          WHERE S.sno = SC.sno AND
              SC.cno = ‘C1’);
      //强调的是字句若为真则产生结果
      https://blog.csdn.net/zhangsify/article/details/71937745
      ```

      

  + 独立子查询：子查询只执行一次，利用子查询结果来计算外层查询语句（内→外）一般用**属性**去查，eg：select ... where ID in (select ...)

  + 相关子查询：子查询中调用外层查询中的元组变量，随着外层元组变量的改变，每次重新执行子查询（外→内）一般用属性**值**去查，eg：select ... where 'p1' in (select ...)

+ tips
  + as换名
  + like通配
  + 自连接要记得换名
  + **写完select子句以后记得检查group by里的属性在不在select里！**
  
+ select语句间的运算

  + ```
    ∪： <子查询1>  UNION	  [ ALL ] <子查询2>
    ∩：	<子查询1> 	INTERSECT  [ ALL ] <子查询2>
    —：	<子查询1> 	EXCEPT	   [ ALL ] <子查询2>
    ```

  + ALL表示不去除冗余重复，**默认去除冗余重复**

+ 统计
  + where字句中不能用统计函数（max(), count()等）

+ 分类
  + 分组查询子句：**GROUP BY colname { , colname ... }**
    + 根据colname将满足WHERE子句的元组划分成不同的集合
    + 出现在select语句中**未被聚集的属性**只能是**出现在GROUP BY子句**中的那些属性
    + **注：如果要让select能使用这些属性，必须在group by子句中也把这些属性列出来，详见ppt database_03_2 p103**
  + 分组查询子句：**HAVING group_condition**
    + 是在Group By**执行后**进行
    + group_condition通常作用于**某统计结果**
+ 映像语句的处理顺序
  + 合并FROM子句中的表
  + 根据WHERE子句中的条件进行元组选择
  + 根据GROUP BY子句对剩下的元组进行分组
  + 根据HAVING子句中的条件对分组后的元组进行选择
  + 根据SELECT子句进行统计计算
  + 根据ORDER BY子句进行

### 数据更新

+ 删除元组

  ```
  DELETE FROM table_name [ WHERE search_condition ];
  ```

+ 插入元组

  ```
  INSERT INTO [(colname {,colname … })] 
  VALUES (expr | NULL{,expr | NULL … }) 
  or
  INSERT INTO [(colname {,colname … })] | subquery;
  ```

+ 修改元组

  ```
  UPDATE table_name
  SET colname = expr | NULL | subquery, ......
  [ WHERE search_condition ] ;
  ```

### 视图

+ 基础 & 与基表的区别

  + 视图（view），又名 导出表（derived table），虚表（virtual table）
  + 并不实际存在，仅保留其构造信息
  + 用户执行视图上的访问操作时，会被转换为对应基表上的访问操作

+ 定义视图

  + **定义视图的子查询中不可以使用Order By**

  ```
  CREATE VIEW <视图名> [(<列名> {,<列名 > … })]
  AS <映像语句> [ WITH CHECK OPTION ]
  ```

  另：与CREATE TABLE的对比

  ```
  CREATE TABLE tablename (
  colname datatype [ NOT NULL ]
  { ,colname datatype [ NOT NULL ] });
  ```

  1. 列名不用给出类型
  2. 甚至不用给出列，直接用select的属性名就行
  3. 必须要给出select语句
  4. with check option：检查对视图做增删改操作是否满足视图的定义条件（即逻辑是否在视图对应的基表中是正确的）

+ 删除视图

  ```
  DROP VIEW <视图名>
  ```

+ 视图上的查询

  + 会被改写为在基表上的操作

+ 视图上的修改

  + 一般不允许，除非视图的每一行和列对应基表的唯一一行和唯一一列

+ **视图的优点**

  + 提高了数据的独立性
  + 简化用户观点
  + 提供自动的安全保护功能