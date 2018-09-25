# Apache Spark简介

原文链接：https://www.baeldung.com/apache-spark

作者：[baeldung](https://www.baeldung.com/author/baeldung/)

译者：[jackzhaiyh](https://github.com/jackzhaiyh)

## 1、简介
_Apache Spark是一个开源的集群计算框架_。 它为Scala，Java，Python和R提供了优雅的开发API，允许开发人员跨多种数据源执行各种数据密集型工作负载，包括HDFS，Cassandra，HBase，S3等。

从历史上看，Hadoop的MapReduce对于某些迭代和交互式计算工作而言效率低下，最终导致了Spark的发展。 _使用Spark，我们可以比内存中的Hadoop运行逻辑快两个数量级，或者在磁盘上运行速度提高一个数量级。_

## 2、Spark架构
Spark应用程序在集群上作为独立的进程集运行，如[下图](https://spark.apache.org/docs/latest/cluster-overview.html)所示：

![Spark Architecture](https://www.baeldung.com/wp-content/uploads/2017/10/cluster-overview.png)

这些进程集由主程序中的SparkContext对象（称为驱动程序）协调。 SparkContext连接到几种类型的集群管理器（Spark自己的独立集群管理器，Mesos或YARN），它们跨应用程序分配资源。

连接后，Spark会在集群中的节点上获取执行程序，这些节点是为您的应用程序运行计算和存储数据的进程。

接下来，它将您的应用程序代码（由传递给SparkContext的JAR或Python文件定义）发送给执行程序。 最后，_SparkContext将任务发送给执行程序以运行。_

## 3、核心组件

下图给出了Spark的不同组件的清晰图片：

![Spark Architecture](https://www.baeldung.com/wp-content/uploads/2017/10/Components-of-Spark.jpg)

### 3.1、Spark Core
Spark Core组件负责所有基本I/O功能，调度和监控spark群集上的作业，任务调度，使用不同存储系统的网络，故障恢复和高效的内存管理。

与Hadoop不同，Spark通过使用称为RDD（弹性分布式数据集）的特殊数据结构，避免将共享数据存储在Amazon S3或HDFS等中间存储中。

_弹性分布式数据集是不可变的，是一个可以并行操作的分区记录集合，允许 - 容错的“内存中”计算。_

RDD支持两种操作：

* 转换 - Spark RDD转换是一种从现有RDD生成新RDD的函数。变换器将RDD作为输入，并产生一个或多个RDD作为输出。转换本质上是懒惰的，即当我们调用动作时它们会被执行
* 操作 - 转换会相互创建RDD，但是当我们想要使用实际数据集时，就会执行操作。因此，Actions是Spark RDD操作，它提供非RDD值。操作值存储在驱动程序或外部存储系统中
操作是从Executor向驱动程序发送数据的方法之一。执行程序是负责执行任务的代理程序。驱动程序是一个JVM进程，用于协调工作和执行任务。 Spark的一些操作是计数和收集。

执行程序是负责执行任务的代理程序。驱动程序是一个JVM进程，用于协调工作和执行任务。 Spark的一些操作是计数和收集。
### 3.2、Spark SQL
Spark SQL是用于结构化数据处理的Spark模块。 它主要用于执行SQL查询。 DataFrame构成了Spark SQL的主要抽象。 订购到命名列的分布式数据集合称为Spark中的DataFrame。

Spark SQL支持从Hive，Avro，Parquet，ORC，JSON和JDBC等不同来源获取数据。 它还可以使用Spark引擎扩展到数千个节点和多小时查询 - 它提供完整的中间查询容错。

### 3.3、Spark Streaming
Spark Streaming是核心Spark API的扩展，可实现实时数据流的可扩展，高吞吐量，容错流处理。 可以从许多来源获取数据，例如Kafka，Flume，Kinesis或TCP套接字。

最后，处理后的数据可以推送到文件系统，数据库和实时仪表板。

### 3.4、Spark Mlib
MLlib是Spark的机器学习（ML）库。 其目标是使实用的机器学习可扩展且简单。 从较高的层面来说，它提供了以下工具：

- ML算法 - 常见的学习算法，如分类，回归，聚类和协同过滤
- 特征化 - 特征提取，转换，降维和选择
- 管道 - 用于构建，评估和调整ML管道的工具
- 持久性 - 保存和加载算法，模型和管道
- 实用程序 - 线性代数，统计，数据处理等

### 3.5、Spark GraphX
GraphX是图形和图形并行计算的组件。 在较高的层次上，GraphX通过引入一个新的Graph抽象来扩展Spark RDD：一个定向的多图，其属性附加到每个顶点和边。

为了支持图形计算，GraphX公开了一组基本运算符（例如，子图，joinVertices和aggregateMessages）。

此外，GraphX包含越来越多的图算法和构建器，以简化图形分析任务。

## 4、Hello World

现在我们了解了核心组件，我们可以继续使用简单的基于Maven的Spark项目 - 用于计算字数。

我们将演示Spark在本地模式下运行，其中所有组件都在同一台机器上本地运行，其中它是主节点，执行器节点或Spark的独立集群管理器。

### 4.1、 Maven Setup
让我们在pom.xml文件中设置一个与Spark相关的依赖项的Java Maven项目：
```xml
<dependencies>
    <dependency>
        <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_2.10</artifactId>
    <version>1.6.0</version>
    </dependency>
</dependencies>
```

4.2、计数-Spark作业
现在让我们编写Spark作业来处理包含句子的文件，并在文件中输出不同的单词及其计数：
```java
public static void main(String[] args) throws Exception {
    if (args.length < 1) {
        System.err.println("Usage: JavaWordCount <file>");
        System.exit(1);
    }
    SparkConf sparkConf = new SparkConf().setAppName("JavaWordCount");
    JavaSparkContext ctx = new JavaSparkContext(sparkConf);
    JavaRDD<String> lines = ctx.textFile(args[0], 1);

    JavaRDD<String> words
      = lines.flatMap(s -> Arrays.asList(SPACE.split(s)).iterator());
    JavaPairRDD<String, Integer> ones
      = words.mapToPair(word -> new Tuple2<>(word, 1));
    JavaPairRDD<String, Integer> counts
      = ones.reduceByKey((Integer i1, Integer i2) -> i1 + i2);

    List<Tuple2<String, Integer>> output = counts.collect();
    for (Tuple2<?, ?> tuple : output) {
        System.out.println(tuple._1() + ": " + tuple._2());
    }
    ctx.stop();
}
```
请注意，我们将本地文本文件的路径作为参数传递给Spark作业。

SparkContext对象是Spark的主要入口点，表示与已经运行的Spark集群的连接。 它使用SparkConf对象来描述应用程序配置。 SparkContext用于将内存中的文本文件读取为JavaRDD对象。

接下来，我们使用flatmap方法将JavaRDD对象行转换为单词JavaRDD对象，首先将每行转换为空格分隔的单词，然后展平每行处理的输出。

我们再次应用变换操作mapToPair，它基本上将每个单词的出现映射到单词的元组和1的计数。

然后，我们应用reduceByKey操作将具有计数1的任何单词的多次出现分组到单词元组并将计数总结。

最后，我们执行收集RDD操作以获得最终结果。

4.3、执行-Spark作业
现在让我们使用Maven构建项目，在目标文件夹中生成apache-spark-1.0-SNAPSHOT.jar。

接下来，我们需要将此WordCount作业提交给Spark：
>${spark-install-dir}/bin/spark-submit --class com.baeldung.WordCount
  --master local ${WordCount-MavenProject}/target/apache-spark-1.0-SNAPSHOT.jar
  ${WordCount-MavenProject}/src/main/resources/spark_example.txt

在运行上述命令之前，需要更新Spark安装目录和WordCount Maven项目目录。

提交后，几个步骤发生在幕后：

1. 从驱动程序代码，SparkContext连接到集群管理器（在我们的例子中，本地运行的spark独立集群管理器）
2. Cluster Manager在其他应用程序之间分配资源
3. Spark在集群中的节点上获取执行程序。 在这里，我们的字数应用程序将获得自己的执行程序进程
4. 应用程序代码（jar文件）被发送给执行程序
5. 任务由SparkContext发送给执行程序。
最后，spark作业的结果返回给驱动程序，我们将看到文件中的单词数作为输出：

>Hello 1
from 2
Baledung 2
Keep 1
Learning 1
Spark 1
Bye 1

## 5、总结
在本文中，我们讨论了Apache Spark的体系结构和不同组件。 我们还演示了一个Spark作业的工作示例，该作业从文件中提供单词计数。

与往常一样，[GitHub上提供了完整的源代码](https://github.com/eugenp/tutorials/tree/master/apache-spark)。

