# Spark运行部署模式

<nav>
<a href="#一作业提交">一、增加Scala插件</a><br/>
<a href="#二Local模式">二、增加依赖关系</a><br/>
<a href="#三Standalone模式">三、WordCount案例</a><br/>
<a href="#三Spark-on-Yarn模式">四、异常处理</a><br/>
</nav>




Spark作为一个数据处理框架和计算引擎，被设计在所有常见的集群环境中运行, 在国内工作中主流的环境为Yarn，不过逐渐容器式环境也慢慢流行起来。接下来，我们就分别看看不同环境下Spark的运行

![spark-3-01](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-01.png)



## 一、Local模式

所谓的Local模式，就是不需要其他任何节点资源就可以在本地执行Spark代码的环境，一般用于教学，调试，演示等，之前在IDEA中运行代码的环境我们称之为开发环境，不太一样。

**1、解压缩文件**

将spark-2.4.5-bin-without-hadoop-scala-2.12.tgz文件上传到/opt/software目录下（没有此目录得自己建，mkdir -p /opt/software），在software下并解压缩，放置在指定位置，路径中不要包含中文或空格，课件后续如果涉及到解压缩操作，不再强调。并用mv命令重命名为spark-local

```
tar -zxvf spark-2.4.5-bin-without-hadoop-scala-2.12.tgz -C /opt/module
cd /opt/module 
mv /opt/module/spark-2.4.5-bin-without-hadoop-scala-2.12 /opt/module/spark-local
```

**spark2.4.5默认不支持Hadoop3，可以采用多种不同的方式关联Hadoop3**

（1）修改spark-local/conf/spark-env.sh文件，增加如下内容

```
SPARK_DIST_CLASSPATH=$(/opt/module/hadoop-3.1.3/bin/hadoop classpath)
```

（2）方式二（建议）：除了修改配置文件外，也可以直接引入对应的Jar包

![spark-3-09](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-09.png)

**2、启动Local环境**

（1）进入解压缩后的路径，执行指令

```
[atguigu@hadoop102 spark-local]$ bin/spark-shell --master local[*]
```

![spark-3-02](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-02.png)

（2）启动成功后，可以输入网址进行Web UI监控页面访问

```
http://虚拟机地址:4040
```

![spark-3-03](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-03.png)

**3、命令行工具**

```
cd /opt/module/spark-local/data/
```

在解压缩文件夹下的data目录中，添加word.txt文件。

```
[atguigu@hadoop102 data]$ vim word.txt
```

添加：

```
hello spark
hello scala
hello flink
spark
```

在命令行工具中执行如下代码指令（和IDEA中代码简化版一致）

```
scala> sc.textFile("data/word.txt").flatMap(_.split(" ")).map((_,1)).reduceByKey(_+_).collect
```

**4、退出本地模式**

按键Ctrl+C或输入Scala指令

```
:quit
```

**5、提交应用**

```
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master local[2] \
./examples/jars/spark-examples_2.12-2.4.5.jar \
10
```

1)    --class表示要执行程序的主类

2)    --master local[2] 部署模式，默认为本地模式，数字表示分配的虚拟CPU核数量

3)    spark-examples_2.12-2.4.5.jar 运行的应用类所在的jar包

4)    数字10表示程序的入口参数，用于设定当前应用的任务数量![spark-3-04](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-04.png)



## 二、Standalone模式

local本地模式毕竟只是用来进行练习演示的，真实工作中还是要将应用提交到对应的集群中去执行，这里我们来看看只使用Spark自身节点运行的集群模式，也就是我们所谓的独立部署（Standalone）模式。Spark的Standalone模式体现了经典的master-slave模式。

集群规划:

|       | Linux1         | Linux2 | Linux3 |
| ----- | -------------- | ------ | ------ |
| Spark | Worker  Master | Worker | Worker |

**1、解压缩文件**

将spark-2.4.5-bin-without-hadoop-scala-2.12.tgz文件上传到Linux并解压缩在指定位置

```
tar -zxvf spark-2.4.5-bin-without-hadoop-scala-2.12.tgz -C /opt/module
cd /opt/module 
mv spark-2.4.5-bin-without-hadoop-scala-2.12 spark-standalone
```

**spark2.4.5默认不支持Hadoop3，可以采用多种不同的方式关联Hadoop3**

（1）方式一：修改spark-standalone/conf/spark-env.sh文件，增加如下内容

```
SPARK_DIST_CLASSPATH=$(/opt/module/hadoop-3.1.3/bin/hadoop classpath)
```

要先启动Hadoop才能用Spark

（2）方式二（建议）：除了修改配置文件外，也可以直接引入对应的Jar包

![spark-3-09](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-09.png)

**2、修改配置文件**

（1）进入解压缩后路径的conf目录，修改slaves.template文件名为slaves

```
mv slaves.template slaves
```

（2）修改slaves文件，添加work节点

```
[atguigu@hadoop102 conf]$ vim slaves
```

```
hadoop102
hadoop103
hadoop104
```

（3）修改spark-env.sh.template文件名为spark-env.sh

```
mv spark-env.sh.template spark-env.sh
```

（4）修改spark-env.sh文件，添加JAVA_HOME环境变量和集群对应的master节点

```
[atguigu@hadoop102 conf]$ vim spark-env.sh 
```

```
export JAVA_HOME=/opt/module/jdk1.8.0_212
SPARK_MASTER_HOST=hadoop102
SPARK_MASTER_PORT=7077
```

注意：7077端口，相当于hadoop3内部通信的8020端口

（5）分发spark-standalone目录

```
[atguigu@hadoop102 module]$ xsync spark-standalone
```

**3、启动集群**

（1）执行脚本命令

```
sbin/start-all.sh
```

（2）查看三台服务器运行进程

```
================ hadoop102 ===============
3685 Jps
3558 Worker
3389 Master
================ hadoop103 ===============
1671 Jps
1577 Worker
================ hadoop104 ===============
1740 Jps
1645 Worker
```

（3）查看Master资源监控Web UI界面: http://hadoop102:8080/

![spark-3-05](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-05.png)

```
8080端口=>资源监控页面--Master
4040端口=>计算监控页面
```

**4、提交应用**

```
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://hadoop102:7077 \
./examples/jars/spark-examples_2.12-2.4.5.jar \
10
```

1)    --class表示要执行程序的主类

2)    --master spark://hadoop102:7077 独立部署模式，连接到Spark集群

3)    spark-examples_2.12-2.4.5.jar 运行类所在的jar包

4)    数字10表示程序的入口参数，用于设定当前应用的任务数量![spark-3-06](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-06.png)

执行任务时，会产生多个Java进程

![spark-3-07](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-07.png)

执行任务时，默认采用服务器集群节点的总核数，每个节点内存1024M。![spark-3-08](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-08.png)

**5、提交参数说明**

在提交应用中，一般会同时一些提交参数

```
bin/spark-submit \
--class <main-class>
--master <master-url> \
... # other options
<application-jar> \
[application-arguments]
```

| 参数                      | 解释                                                         | 可选值举例                                      |
| ------------------------- | ------------------------------------------------------------ | ----------------------------------------------- |
| --class                   | Spark程序中包含主函数的类                                    |                                                 |
| --master                  | Spark程序运行的模式                                          | 本地模式：local[*]、spark://linux1:7077、  Yarn |
| --executor-memory  1G     | 指定每个executor可用内存为1G                                 | 符合集群内存配置即可，具体情况具体分析。        |
| --total-executor-cores  2 | 指定所有executor使用的cpu核数为2个                           |                                                 |
| --executor-cores          | 指定每个executor使用的cpu核数                                |                                                 |
| application-jar           | 打包好的应用jar，包含依赖。这个URL在集群中全局可见。 比如hdfs:// 共享存储系统，如果是file://  path，那么所有的节点的path都包含同样的jar |                                                 |
| application-arguments     | 传给main()方法的参数                                         |                                                 |

##### **6、配置历史服务**

由于spark-shell停止掉后，集群监控hadoop102:4040页面就看不到历史任务的运行情况，所以开发时都配置历史服务器记录任务运行情况。

```
**注意：需要启动hadoop集群，HDFS上的directory目录需要提前存在。**
退出hadoop安全模式
hdfs dfsadmin -safemode leave
sbin/start-dfs.sh
[atguigu@hadoop102 conf]$ hdfs dfs -mkdir /directory
```

(1)    修改spark-defaults.conf.template文件名为spark-defaults.conf

```
mv spark-defaults.conf.template spark-defaults.conf
```

（2）修改spark-default.conf文件，配置日志存储路径

```
spark.eventLog.enabled          true
spark.eventLog.dir              hdfs://hadoop102:8020/directory
```

（3）修改spark-env.sh文件, 添加日志配置

```
export SPARK_HISTORY_OPTS="
-Dspark.history.ui.port=18080
-Dspark.history.fs.logDirectory=hdfs://hadoop102:8020/directory
-Dspark.history.retainedApplications=30"
```

- 参数1含义：WEBUI访问的端口号为18080

- 参数2含义：指定历史服务器日志存储路径

- 参数3含义：指定保存Application历史记录的个数，如果超过这个值，旧的应用程序信息将被删除，这个是内存中的应用数，而不是页面上显示的应用数。

（4）分发配置文件

```
[atguigu@hadoop102 spark-standalone]$ xsync conf
```

（5）重新启动集群和历史服务

```
sbin/start-all.sh
sbin/start-history-server.sh
```

（6）重新执行任务

```
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://hadoop102:7077 \
./examples/jars/spark-examples_2.12-2.4.5.jar \
10
```

（7）查看历史服务：http://hadoop102:18080/

![spark-3-11](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-11.png)

**7、配置高可用（HA）**

所谓的高可用是因为当前集群中的Master节点只有一个，所以会存在单点故障问题。所以为了解决单点故障问题，需要在集群中配置多个Master节点，一旦处于活动状态的Master发生故障时，由备用Master提供服务，保证作业可以继续执行。这里的高可用一般采用Zookeeper设置

|       | Linux1                    | Linux2                    | Linux3            |
| ----- | ------------------------- | ------------------------- | ----------------- |
| Spark | Master  Zookeeper  Worker | Master  Zookeeper  Worker | Zookeeper  Worker |

（1）停止集群

```
sbin/stop-all.sh 
```

（2)    启动Zookeeper

```
[atguigu@hadoop102 bin]$ zk.sh start
```

zk.sh脚本：

```
#!/bin/bash
case $1 in
    start)
        for i in hadoop102 hadoop103 hadoop104;do
        echo "================ $i ==============="
        ssh $i "/opt/module/zookeeper-3.5.7/bin/zkServer.sh start"
        done
    ;;

    stop)
        for i in hadoop102 hadoop103 hadoop104;do
        echo "================ $i ==============="
        ssh $i "/opt/module/zookeeper-3.5.7/bin/zkServer.sh stop"
        done
    ;;

   status)
        for i in hadoop102 hadoop103 hadoop104;do
        echo "================ $i ==============="
        ssh $i "/opt/module/zookeeper-3.5.7/bin/zkServer.sh status"
        done
    ;;
esac
```

（3）修改spark-env.sh文件添加如下配置

```
注释如下内容：
#SPARK_MASTER_HOST=hadoop102
#SPARK_MASTER_PORT=7077
SPARK_MASTER_WEBUI_PORT=8989

添加如下内容:
export SPARK_DAEMON_JAVA_OPTS="
-Dspark.deploy.recoveryMode=ZOOKEEPER 
-Dspark.deploy.zookeeper.url=hadoop102,hadoop103,hadoop104 
-Dspark.deploy.zookeeper.dir=/spark"

```

（4）分发配置文件

```
xsync conf/ 
```

（5）启动集群

```
sbin/start-all.sh 
```

（6）查看web页面：hadoop102:8989

![spark-3-09](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-21.png)

（7）启动hadoop102的单独Master节点，此时hadoop102节点Master状态处于备用状态

```
[atguigu@hadoop103 spark-standalone]$  sbin/start-master.sh
```

（8）查看web页面：hadoop103:8989

![spark-3-09](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-22.png)

（9）提交应用到高可用集群

```
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master spark://hadoop102:7077,hadoop103:7077 \
--deploy-mode cluster \
./examples/jars/spark-examples_2.12-2.4.5.jar \
10
```

（10）停止hadoop102的Master资源监控进程

![spark-3-09](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-23.png)

（11）查看hadoop103的Master 资源监控Web UI，稍等一段时间后hadoop103节点的Master状态提升为活动状态

![spark-3-09](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-24.png)



## 三、Yarn模式

独立部署（Standalone）模式由Spark自身提供计算资源，无需其他框架提供资源。这种方式降低了和其他第三方资源框架的耦合性，独立性非常强。但是你也要记住，Spark主要是计算框架，而不是资源调度框架，所以本身提供的资源调度并不是它的强项，所以还是和其他专业的资源调度框架集成会更靠谱一些。所以接下来我们来学习在强大的Yarn环境下Spark是如何工作的（其实是因为在国内工作中，Yarn使用的非常多）。

**1、解压缩文件**

将spark-2.4.5-bin-without-hadoop-scala-2.12.tgz文件上传到Linux并解压缩在指定位置

```
tar -zxvf spark-2.4.5-bin-without-hadoop-scala-2.12.tgz -C /opt/module
cd /opt/module 
mv spark-2.4.5-bin-without-hadoop-scala-2.12 spark-yarn
```

**spark2.4.5默认不支持Hadoop3，可以采用多种不同的方式关联Hadoop3**

（1）方式一：修改spark-standalone/conf/spark-env.sh文件，增加如下内容

```
SPARK_DIST_CLASSPATH=$(/opt/module/hadoop-3.1.3/bin/hadoop classpath)
```

要先启动Hadoop才能用Spark

（2）方式二（建议）：除了修改配置文件外，也可以直接引入对应的Jar包

![spark-3-09](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-09.png)

**2、修改配置文件**

（1） 修改hadoop配置文件/opt/module/hadoop-3.1.3/etc/hadoop/yarn-site.xml, 并分发

```
<!--是否启动一个线程检查每个任务正使用的物理内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
<property>
     <name>yarn.nodemanager.pmem-check-enabled</name>
     <value>false</value>
</property>

<!--是否启动一个线程检查每个任务正使用的虚拟内存量，如果任务超出分配值，则直接将其杀掉，默认是true -->
<property>
     <name>yarn.nodemanager.vmem-check-enabled</name>
     <value>false</value>
</property>
```

（2） 修改conf/spark-env.sh，添加JAVA_HOME和YARN_CONF_DIR配置

```
mv spark-env.sh.template spark-env.sh
。。。
export JAVA_HOME=/opt/module/jdk1.8.0_212
YARN_CONF_DIR=/opt/module/hadoop-3.1.3/etc/hadoop
```

**3、启动HDFS以及YARN集群**

```
#102上
[atguigu@hadoop102 hadoop-3.1.3]$ sbin/start-dfs.sh 
```

```
#103上
[atguigu@hadoop103 hadoop-3.1.3]$ sbin/start-yarn.sh 
```

**4、提交应用**

```
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \
./examples/jars/spark-examples_2.12-2.4.5.jar \
10
```

```
如果出现如下错误：
Caused by: org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.hdfs.server.namenode.SafeModeException): Cannot create directory /user/atguigu/.sparkStaging/application_1591070627301_0001. Name node is in safe mode.
得退出hadoop安全模式
```

```
 hdfs dfsadmin -safemode leave
```

然后再提交应用

最后显示结果为

![spark-3-09](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-12.png)

查看http://192.168.117.103:8088/页面，点击History，查看历史页面

![spark-3-09](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-13.png)

**5、配置历史服务器**



```
**注意：需要启动hadoop集群，HDFS上的directory目录需要提前存在。**
退出hadoop安全模式
hdfs dfsadmin -safemode leave
sbin/start-dfs.sh
[atguigu@hadoop102 conf]$ hdfs dfs -mkdir /directory
```

(1)    修改spark-defaults.conf.template文件名为spark-defaults.conf

```
mv spark-defaults.conf.template spark-defaults.conf
```

（2）修改spark-default.conf文件，配置日志存储路径

```
spark.eventLog.enabled          true
spark.eventLog.dir              hdfs://hadoop102:8020/directory
```

（3）修改spark-env.sh文件, 添加日志配置

```
export SPARK_HISTORY_OPTS="
-Dspark.history.ui.port=18080
-Dspark.history.fs.logDirectory=hdfs://hadoop102:8020/directory
-Dspark.history.retainedApplications=30"
```

- 参数1含义：WEBUI访问的端口号为18080

- 参数2含义：指定历史服务器日志存储路径

- 参数3含义：指定保存Application历史记录的个数，如果超过这个值，旧的应用程序信息将被删除，这个是内存中的应用数，而不是页面上显示的应用数。

（4） 修改spark-defaults.conf

```
spark.yarn.historyServer.address=hadoop102:18080
spark.history.ui.port=18080
```

（5）重新启动集群和历史服务

```
sbin/start-history-server.sh 
```

（6）重新执行任务

```
bin/spark-submit \
--class org.apache.spark.examples.SparkPi \
--master yarn \
./examples/jars/spark-examples_2.12-2.4.5.jar \
10
```

![spark-3-09](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-14.png)

（7）Web页面查看日志：http://hadoop102:18080/

![spark-3-09](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-15.png)

 Web页面查看日志：http://hadoop103:8088

![spark-3-09](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-16.png)



## 四、Windows模式

每次都需要启动虚拟机，启动集群，这是一个比较繁琐的过程，并且会占大量的系统资源，导致系统执行变慢，不仅仅影响学习效果，也影响学习进度，Spark非常暖心地提供了可以在windows系统下启动本地集群的方式，这样，在不使用虚拟机的情况下，也能学习Spark的基本使用

**1、解压缩文件**

将文件spark-2.4.5-bin-without-hadoop-scala-2.12.tgz解压缩到无中文无空格的路径中，将hadoop3依赖jar包拷贝到jars目录中。

![spark-3-09](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-17.png)

引入jar包

![spark-3-09](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-09.png)

**2、启动本地环境**

（1）执行解压缩文件路径下bin目录中的spark-shell.cmd文件，启动Spark本地环境

![spark-3-09](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-18.png)

（2）  在bin目录中创建input目录，并添加word.txt文件, 在命令行中输入脚本代码

```
sc.textFile("input/word.txt").flatMap(_.split(",")).map((_,1)).reduceByKey(_+_).collect
```

![spark-3-09](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-19.png)

**3、命令行提交应用**

在F:\spark-2.4.5\bin 目录下cmd 在打开cmd 执行如下命令：

```
spark-submit --class org.apache.spark.examples.SparkPi --master local[2] ../examples/jars/spark-examples_2.12-2.4.5.jar 10
```

![spark-3-09](https://github.com/bigdata2018/BigData-Spark/blob/master/picture/spark-3-20.png)

