# Spark核心编程

大数据Spark学习指南（2020） — — 持续更新完善中

<nav>
<a href="#二、Hadoop相关安装">一、RDD</a><br/>
<a href="#三、Spark相关安装">二、RDD转换算子</a><br/>
</nav>


Spark计算框架为了能够进行高并发和高吞吐的数据处理，封装了三大数据结构，用于处理不同的应用场景。三大数据结构分别是：

- RDD : 弹性分布式数据集

- 累加器：分布式共享只写变量

- 广播变量：分布式共享只读变量

接下来我们一起看看这三大数据结构是如何在数据处理中使用的。



## [一、RDD](https://github.com/bigdata2018/BigData-Spark/blob/master/nodes/Spark%E6%A6%82%E5%BF%B5.md)

#### **1、RDD概念**

RDD（Resilient Distributed Dataset）叫做弹性分布式数据集，是Spark中最基本的数据处理模型。代码中是一个抽象类，它代表一个弹性的、不可变、可分区、里面的元素可并行计算的集合。

**弹性**

- 存储的弹性：内存与磁盘的自动切换；

- 容错的弹性：数据丢失可以自动恢复；

- 计算的弹性：计算出错重试机制；

- 分片的弹性：可根据需要重新分片。

**分布式：**数据存储在大数据集群不同节点上

**数据集：**RDD封装了计算逻辑，并不保存数据

**数据抽象：**RDD是一个抽象类，需要子类具体实现

**不可变：**RDD封装了计算逻辑，是不可以改变的，想要改变，只能产生新的RDD，在新的RDD里面封装计算逻辑

**可分区、并行计算**

```
RDD特点
1、RDD 是一个编程模型
（1）RDD 允许用户显式的指定数据存放在内存或者磁盘
（2）RDD 是分布式的, 用户可以控制 RDD 的分区

2、RDD 是一个编程模型
（1）RDD 提供了丰富的操作
（2）RDD 提供了 map, flatMap, filter 等操作符, 用以实现 Monad 模式
（3）RDD 提供了 reduceByKey, groupByKey 等操作符, 用以操作 Key-Value 型数据
（4）RDD 提供了 max, min, mean 等操作符, 用以操作数字型的数据

3、RDD 是混合型的编程模型, 可以支持迭代计算, 关系查询, MapReduce, 流计算

4、RDD 是只读的

5、RDD 之间有依赖关系, 根据执行操作的操作符的不同, 依赖关系可以分为宽依赖和窄依赖
```



#### **2、核心属性**

![spark-5-01](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-5-01.png)

（1）分区列表

![spark-5-02](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-5-02.png)

![spark-5-02](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-5-11.png)

```
例如上图中, 使用了一个 RDD 表示 HDFS 上的某一个文件, 这个文件在 HDFS 中是分三块, 那么 RDD 在读取的时候就也有三个分区, 每个 RDD 的分区对应了一个 HDFS 的分块

后续 RDD 在计算的时候, 可以更改分区, 也可以保持三个分区, 每个分区之间有依赖关系, 例如说 RDD2 的分区一依赖了 RDD1 的分区一
```

（2）分区计算函数

Spark在计算时，是使用分区函数对每一个分区进行计算

![spark-5-03](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-5-03.png)

```
RDD 是弹性分布式数据集
RDD 一个非常重要的前提和基础是 RDD 运行在分布式环境下, 其可以分区
```

（3）RDD之间的依赖关系

RDD是计算模型的封装，当需求中需要将多个计算模型进行组合时，就需要将多个RDD建立依赖关系

![spark-5-04](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-5-04.png)

（4）分区器（可选）

当数据为KV类型数据时，可以通过设定分区器自定义数据的分区

![spark-5-05](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-5-05.png)

（5）首选位置（可选）

计算数据时，可以根据计算节点的状态选择不同的节点位置进行计算

![spark-5-06](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-5-06.png)



#### **3、执行原理**

从计算的角度来讲，数据处理过程中需要计算资源（内存 & CPU）和计算模型（逻辑）。执行时，需要将计算资源和计算模型进行协调和整合。

Spark框架在执行时，先申请资源，然后将应用程序的数据处理逻辑分解成一个一个的计算任务。然后将任务发到已经分配资源的计算节点上, 按照指定的计算模型进行数据计算。最后得到计算结果。

RDD是Spark框架中用于数据处理的核心模型，接下来我们看看，在Yarn环境中，RDD的工作原理:

（1） 启动Yarn集群环境

![spark-5-07](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-5-07.png)

（2）Spark通过申请资源创建调度节点和计算节点

![spark-5-08](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-5-08.png)

（3）Spark框架根据需求将计算逻辑根据分区划分成不同的任务

![spark-5-09](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-5-09.png)

（4）调度节点将任务根据计算节点状态发送到对应的计算节点进行计算

![spark-5-10](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-5-10.png)

从以上流程可以看出RDD在整个流程中主要用于将逻辑进行封装，并生成Task发送给Executor节点执行计算，接下来我们就一起看看Spark框架中RDD是具体是如何进行数据处理的。



#### 4、创建RDD

在Spark中创建RDD的创建方式可以分为四种：

**（1）从集合（内存）中创建RDD**

从集合中创建RDD，Spark主要提供了两个方法：parallelize和makeRDD

```
val sparkConf =
    new SparkConf().setMaster("local[*]").setAppName("spark")
val sparkContext = new SparkContext(sparkConf)
val rdd1 = sparkContext.parallelize(
    List(1,2,3,4)
)
val rdd2 = sparkContext.makeRDD(
    List(1,2,3,4)
)
rdd1.collect().foreach(println)
rdd2.collect().foreach(println)
sparkContext.stop()
```

从底层代码实现来讲，makeRDD方法其实就是parallelize方法

```
def makeRDD[T: ClassTag](
    seq: Seq[T],
    numSlices: Int = defaultParallelism): RDD[T] = withScope {
  parallelize(seq, numSlices)
}
```

因为不是从外部直接读取数据集的, 所以没有外部的分区可以借鉴, 于是在这两个方法都都有两个参数, 第一个参数是本地集合, 第二个参数是分区数

**（2）  从外部存储（文件）创建RDD**

由外部存储系统的数据集创建RDD包括：本地的文件系统，所有Hadoop支持的数据集，比如HDFS、HBase等。

```
访问方式：

1、支持访问文件夹, 例如 sc.textFile("hdfs:///dataset")
2、支持访问压缩文件, 例如 
sc.textFile("hdfs:///dataset/words.gz")
3、支持通过通配符访问, 例如 sc.textFile("hdfs:///dataset/*.txt")
```

```
如果把 Spark 应用跑在集群上, 则 Worker 有可能在任何一个节点运行
所以如果使用 file:///…; 形式访问本地文件的话, 要确保所有的 Worker 中对应路径上有这个文件, 否则可能会报错无法找到文件
```

```
分区：

1、默认情况下读取 HDFS 中文件的时候, 每个 HDFS 的 block 对应一个 RDD 的 partition, block 的默认是128M
2、通过第二个参数, 可以指定分区数量, 例如 sc.textFile("hdfs://node01:8020/dataset/wordcount.txt", 20)
3、如果通过第二个参数指定了分区, 这个分区数量一定不能小于`block`数
```

通常每个 CPU core 对应 2 - 4 个分区是合理的值

```
支持的平台：

1、支持 Hadoop 的几乎所有数据格式, 支持 HDFS 的访问
2、通过第三方的支持, 可以访问AWS和阿里云中的文件, 详情查看对应平台的 API
```

```
val sparkConf =
    new SparkConf().setMaster("local[*]").setAppName("spark")
val sparkContext = new SparkContext(sparkConf)
val fileRDD: RDD[String] = sparkContext.textFile("input")
fileRDD.collect().foreach(println)
sparkContext.stop()
```

**（3）从其他RDD创建**

```
val conf = new SparkConf().setMaster("local[2]")
val sc = new SparkContext(conf)

val source: RDD[String] = sc.textFile("hdfs://node01:8020/dataset/wordcount.txt", 20)
val words = source.flatMap { line => line.split(" ") }
```

```
source 是通过读取 HDFS 中的文件所创建的
words 是通过 source 调用算子 map 生成的新 RDD
```

**（4） 直接创建RDD（new）**

使用new的方式直接构造RDD，一般由Spark框架自身使用。



#### 5、RDD并行度与分区

默认情况下，Spark可以将一个作业切分多个任务后，发送给Executor节点并行计算，而能够并行计算的任务数量我们称之为并行度。这个数量可以在构建RDD时指定。记住，这里的并行执行的任务数量，并不是指的切分任务的数量，不要混淆了。

```
val sparkConf =
    new SparkConf().setMaster("local[*]").setAppName("spark")
val sparkContext = new SparkContext(sparkConf)
val dataRDD: RDD[Int] =
    sparkContext.makeRDD(
        List(1,2,3,4),
        4)
val fileRDD: RDD[String] =
    sparkContext.textFile(
        "input",
        2)
fileRDD.collect().foreach(println)
sparkContext.stop()
```

读取内存数据时，数据可以按照并行度的设定进行数据的分区操作，数据分区规则的Spark核心源码如下：

```
def positions(length: Long, numSlices: Int): Iterator[(Int, Int)] = {
  (0 until numSlices).iterator.map { i =>
    val start = ((i * length) / numSlices).toInt
    val end = (((i + 1) * length) / numSlices).toInt
    (start, end)
  }
}
```

读取文件数据时，数据是按照Hadoop文件读取的规则进行切片分区，而切片规则和数据读取的规则有些差异，具体Spark核心源码如下

```
public InputSplit[] getSplits(JobConf job, int numSplits)
    throws IOException {

    long totalSize = 0;                           // compute total size
    for (FileStatus file: files) {                // check we have valid files
      if (file.isDirectory()) {
        throw new IOException("Not a file: "+ file.getPath());
      }
      totalSize += file.getLen();
    }

    long goalSize = totalSize / (numSplits == 0 ? 1 : numSplits);
    long minSize = Math.max(job.getLong(org.apache.hadoop.mapreduce.lib.input.
      FileInputFormat.SPLIT_MINSIZE, 1), minSplitSize);
      
    ...
    
    for (FileStatus file: files) {
    
        ...
    
    if (isSplitable(fs, path)) {
          long blockSize = file.getBlockSize();
          long splitSize = computeSplitSize(goalSize, minSize, blockSize);

          ...

  }
  protected long computeSplitSize(long goalSize, long minSize,
                                       long blockSize) {
    return Math.max(minSize, Math.min(goalSize, blockSize));
  }
```

```
基本上平均分，如果不能整除，会采用一个基本的算法实现分区
val start = ((i * length) / numSlices).toInt
val end = (((i + 1) * length) / numSlices).toInt
(start, end)

0->（0，1）-1
1->（1，3）-2，3
2->（3，5）-4，5
```



## 二、RDD转换算子
