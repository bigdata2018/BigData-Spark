# Spark快速度入门

<nav>
<a href="#一作业提交">一、增加Scala插件</a><br/>
<a href="#二Local模式">二、增加依赖关系</a><br/>
<a href="#三Standalone模式">三、WordCount案例</a><br/>
<a href="#三Spark-on-Yarn模式">四、异常处理</a><br/>
</nav>




## 一、创建工程

**我们使用的Spark版本为2.4.5，默认采用的Scala版本为2.12**

1、创建 IDEA 工程

![spark-1-06](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-1-06.png)

2、增加 Scala 支持

![spark-1-07](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-1-07.png)



## 二、增加依赖关系

修改Maven项目中的POM文件，增加Spark框架的依赖关系。本课件基于Spark2.4.5版本，使用时请注意对应版本。

```
<dependencies>
    <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-core_2.12</artifactId>
        <version>2.4.5</version>
    </dependency>
</dependencies>
<build>
    <plugins>
        <!-- 该插件用于将Scala代码编译成class文件 -->
        <plugin>
            <groupId>net.alchim31.maven</groupId>
            <artifactId>scala-maven-plugin</artifactId>
            <version>3.2.2</version>
            <executions>
                <execution>
                    <!-- 声明绑定到maven的compile阶段 -->
                    <goals>
                        <goal>testCompile</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>3.0.0</version>
            <configuration>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
            </configuration>
            <executions>
                <execution>
                    <id>make-assembly</id>
                    <phase>package</phase>
                    <goals>
                        <goal>single</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```



## 三、WordCount案例

流程分析：

![spark-1-08](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-1-08.png)

**创建目录 `src/main/scala`** 

项目下建input目录，目录下建：

1.txt 里添加

```
hello scala
hello spark
```

2.txt 里添加

```
hello spark
hello flink
```

scala下建立 cn.nogc.bigdata.spark.core包

```
package cn.nogc.bigdata.spark.core

import org.apache.spark.rdd.RDD
import org.apache.spark.{SparkConf, SparkContext}

/**
 * description: Spark01_WordCount 
 * date: 2020/5/29 17:41 
 * author: nogc
 * version: 1.0 
 */
object Spark01_WordCount {
  def main(args: Array[String]): Unit = {
    //1.1 准备Spark环境
    val conf: SparkConf = new SparkConf().setAppName("WordCount").setMaster("local[*]")
    //1.2 获取上下文环境对象
    val sc = new SparkContext(conf)

    //2.1 读文件中数据
    val fileRDD: RDD[String] = sc.textFile("input")
    //2.2 分词/扁平化
    val wordRDD: RDD[String] = fileRDD.flatMap(_.split(" "))
    //2.3 拆分后的数据结构改变
    val word2OneRDD: RDD[(String, Int)] = wordRDD.map((_, 1))
  /*  //2.4 分组
    val word2IterRdd: RDD[(String, Iterable[(String, Int)])] = word2OneRDD.groupBy(_._1)
    //2.5 聚合
    val word2CountRDD: RDD[(String, Int)] = word2IterRdd.map {
      case (word, list) => {
        (word, list.size)
      }
    }*/

    val value: RDD[(String, Int)] = word2OneRDD.reduceByKey(_ + _)
    //2.6 展示
    val result: Array[(String, Int)] = value.collect()
    result.foreach(println)

    //3释放连接
    sc.stop()
  }
}
```

执行过程中，会产生大量的执行日志，如果为了能够看好的查看程序的执行结果，可以在项目的resources目录中创建log4j.properties文件，并添加日志配置信息：

```
log4j.rootCategory=ERROR, console
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.target=System.err
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%d{yy/MM/dd HH:mm:ss} %p %c{1}: %m%n

# Set the default spark-shell log level to ERROR. When running the spark-shell, the
# log level for this class is used to overwrite the root logger's log level, so that
# the user can have different defaults for the shell and regular Spark apps.
log4j.logger.org.apache.spark.repl.Main=ERROR

# Settings to quiet third party logs that are too verbose
log4j.logger.org.spark_project.jetty=ERROR
log4j.logger.org.spark_project.jetty.util.component.AbstractLifeCycle=ERROR
log4j.logger.org.apache.spark.repl.SparkIMain$exprTyper=ERROR
log4j.logger.org.apache.spark.repl.SparkILoop$SparkILoopInterpreter=ERROR
log4j.logger.org.apache.parquet=ERROR
log4j.logger.parquet=ERROR

# SPARK-9183: Settings to avoid annoying messages when looking up nonexistent UDFs in SparkSQL with Hive support
log4j.logger.org.apache.hadoop.hive.metastore.RetryingHMSHandler=FATAL
log4j.logger.org.apache.hadoop.hive.ql.exec.FunctionRegistry=ERROR
```



## 四、异常处理

1、如果本机操作系统是Windows，在程序中使用了Hadoop相关的东西，比如写入文件到HDFS，则会遇到如下异常：

![spark-1-02](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-1-02.png)

2、出现这个问题的原因，并不是程序的错误，而是windows系统用到了hadoop相关的服务，解决办法是通过配置关联到windows的系统依赖就可以了：

![spark-1-03](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-1-03.png)

3、在IDEA中配置Run Configuration，添加HADOOP_HOME变量![spark-1-04](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-1-04.png)

![spark-1-05](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-1-05.png)
