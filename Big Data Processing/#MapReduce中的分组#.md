# MapReduce中的分组

**MapReduce中的数据流动**

map-combine(本地reduce)-partition-reduce

![img](https://img-blog.csdnimg.cn/20190903173936954.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3BhdWwyNTA2NzA=,size_16,color_FFFFFF,t_70)

**Partition 分区**

决定key-value分发到哪个reducer。默认用Hashing的方式将key-value分发到reducer。

***getPartition() 输入：Map的key-value结果+Reducer数量 输出：分配的Reducer编号***

在Reduce过程中，可以根据实际需求，把Map完的数据Reduce到不同的文件中。可通过setNumReduceTasks来设置分区的个数。
默认MapReduce使用HashPartitioner来进行分区。但有时，会造成数据倾斜，那么我们可以自定义分区算法。

~~~
//将小于等于3的key放在一个分区
//等于6的放到一个分区
//剩下其它的放到一个分区
public class MyPartitioner extends Partitioner<IntWritable, IntWritable> {
    @Override
    public int getPartition(IntWritable intWritable, IntWritable intWritable2, int i) {
        int key = intWritable.get();
        if (key <= 3) {
            return  0;
        } else if (key == 6) {
            return  1;
        } else {
            return  2;
        }
    }
}

job.setNumReduceTasks(3);//设置分区的个数
job.setPartitionerClass(MyPartitioner.class);//使用自定义分区算法
~~~



**分组**

分区的目的是根据Key值决定Mapper的输出记录被送到哪一个Reducer上去处理。分组就是与记录的Key相关。在同一个分区里面，具有相同Key值的记录是属于同一个分组的。

~~~
//将left相等的key放在同一个组
public class MyGroup extends WritableComparator {
    public MyGroup() {
        super(MySortWritable.class, true);
    }
    @Override
    public int compare(WritableComparable a, WritableComparable b) {
        MySortWritable n1 = (MySortWritable)a;
        MySortWritable n2 = (MySortWritable)b;
        return n1.getLeft() - n2.getLeft();
    }
}
job.setGroupingComparatorClass(MyGroup.class);//使用自定义分组算法

//对于组内的排序，可以利用setSortComparatorClass来实现
~~~





![image-20200706142306608](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200706142306608.png)

