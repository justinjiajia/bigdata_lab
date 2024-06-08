

###  Settings

- 1 primary instance; type: `m4.large`

- 4 core instances; type: `m4.large`
  
    <img width="300" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/1644cc8c-d79b-4c48-a194-f5c49478d126">

- EMR release: 7.1.0

- Software configurations
```json
[
    {
        "classification":"core-site",
        "properties": {
            "hadoop.http.staticuser.user": "hadoop"
        }
    },
    {
        "classification": "hdfs-site",
        "properties": {
            "dfs.replication": "3"
        }
    }
]
```
- Run a .sh file at `s3://ust-bigdata-class/install_python_libraries.sh` as a bootstrap action

> Use AWS CLI: https://github.com/justinjiajia/bigdata_lab/blob/main/AWS_CLI_command.md


```shell
echo 'sc.getConf.get("spark.submit.deployMode")' | spark-shell --master local
[hadoop@ip-172-31-57-98 ~]$ echo 'sc.getConf.get("spark.submit.deployMode")' | spark-shell --master local
Jun 08, 2024 2:57:54 PM org.apache.spark.launcher.Log4jHotPatchOption staticJavaAgentOption
WARNING: spark.log4jHotPatch.enabled is set to true, but /usr/share/log4j-cve-2021-44228-hotpatch/jdk17/Log4jHotPatchFat.jar does not exist at the configured location

Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
Spark context Web UI available at http://ip-172-31-57-98.ec2.internal:4040
Spark context available as 'sc' (master = local, app id = local-1717858688375).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 3.5.0-amzn-1
      /_/
         
Using Scala version 2.12.17 (OpenJDK 64-Bit Server VM, Java 17.0.11)
Type in expressions to have them evaluated.
Type :help for more information.

scala> sc.getConf.get("spark.submit.deployMode")
res0: String = client

scala> :quit
[hadoop@ip-172-31-57-98 ~]$ echo 'sc.getConf.get("spark.submit.deployMode")' | spark-shell --master yarn
Jun 08, 2024 2:58:35 PM org.apache.spark.launcher.Log4jHotPatchOption staticJavaAgentOption
WARNING: spark.log4jHotPatch.enabled is set to true, but /usr/share/log4j-cve-2021-44228-hotpatch/jdk17/Log4jHotPatchFat.jar does not exist at the configured location

Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
24/06/08 14:58:52 WARN Client: Neither spark.yarn.jars nor spark.yarn.archive is set, falling back to uploading libraries under SPARK_HOME.
24/06/08 14:59:06 WARN YarnSchedulerBackend$YarnSchedulerEndpoint: Attempted to request executors before the AM has registered!
Spark context Web UI available at http://ip-172-31-57-98.ec2.internal:4040
Spark context available as 'sc' (master = yarn, app id = application_1717855434172_0005).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 3.5.0-amzn-1
      /_/
         
Using Scala version 2.12.17 (OpenJDK 64-Bit Server VM, Java 17.0.11)
Type in expressions to have them evaluated.
Type :help for more information.

scala> sc.getConf.get("spark.submit.deployMode")
res0: String = client

scala> :quit
```

Spark shells only run in client mode.


### Example 1

```shell
pyspark --master yarn --conf spark.dynamicAllocation.executorIdleTimeout=10m
```

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/ccb137d0-4ed1-4553-a330-0a87f8f530d8">


```python
>>> import numpy as np
>>> parallel_rdd = sc.parallelize(np.arange(1000000)**2)
>>> parallel_rdd.cache()
ParallelCollectionRDD[0] at readRDDFromFile at PythonRDD.scala:289
>>> parallel_rdd.count()
24/06/07 13:04:05 WARN TaskSetManager: Stage 0 contains a task of very large size (1173 KiB). The maximum recommended task size is 1000 KiB.
1000000                                                                         
>>> print(parallel_rdd.toDebugString().decode())
(16) ParallelCollectionRDD[0] at readRDDFromFile at PythonRDD.scala:289 [Memory Serialized 1x Replicated]
 |        CachedPartitions: 16; MemorySize: 5.9 MiB; DiskSize: 0.0 B

>>> parallel_rdd.getStorageLevel()
StorageLevel(False, True, False, False, 1)
>>> print(parallel_rdd.getStorageLevel())
Memory Serialized 1x Replicated
>>> parallel_rdd.getStorageLevel().MEMORY_ONLY
StorageLevel(False, True, False, False, 1)

>>> parallel_rdd.unpersist()

```

The `StorageLevel` class: https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.StorageLevel.html

<img width="300" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/b5f917ec-b73d-4941-a45d-9723b91cc1d7">

One stage only; 16 tasks (4 executors * 4 cores / executor) are created at approximately the same time for this stage:

<img width="400" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/1fd99eac-a5c8-4d09-9bb5-4eda2577ee9a">



<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/2cf9ba67-cc22-4f17-be89-46e0cc596535">
<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/f2cac384-54f2-4be8-88aa-239e7013f6e1">


The 4 exectuors stay alive even after 10 minutes elapses due to the data they cache.

but once you run:
```
>>> parallel_rdd.unpersist()
ParallelCollectionRDD[0] at readRDDFromFile at PythonRDD.scala:289
```
The 4 executors will be immediately removed.


### Example 2

```shell
wget https://raw.githubusercontent.com/justinjiajia/datafiles/main/soc-LiveJournal1Adj.txt

hadoop fs -mkdir /input

hadoop fs -put soc-LiveJournal1Adj.txt /input

pyspark --master yarn --executor-memory 2g --conf spark.dynamicAllocation.executorIdleTimeout=20m

```

<img width="938" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/34186597-49b3-414b-b38a-01a79bfec899">

```python
lines = sc.textFile("hdfs:///input/soc-LiveJournal1Adj.txt")
friend_lists = lines.map(lambda x: x.strip().split("\t")).filter(lambda x: len(x) == 2).mapValues(lambda x: x.split(","))

already_friend_pairs = friend_lists.flatMap(lambda x: [(int(x[0]), int(item)) for item in x[1]]) \
                                   .map(lambda x: x if x[0] <= x[1] else (x[1], x[0])).distinct()

already_friend_pairs.cache()

from itertools import combinations
potential_pairs = friend_lists.flatMap(lambda x: combinations(x[1], 2)).map(lambda x: (int(x[0]), int(x[1])))
rec_pairs = potential_pairs.subtract(already_friend_pairs)
output_pairs = rec_pairs.map(lambda x: (x, 1)).reduceByKey(lambda a,b: a+b)
output_pairs.saveAsTextFile("hdfs:///rec_pairs_output")
```



   
  
<img width="600" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/5b599410-283f-4e43-abce-79b2017a98c6">

Ongoing
<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/69cb158c-7d5d-455b-98b3-ab2b3e64ff58">

Completed
<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/2f178dbd-7658-4a54-9a5f-9809f7710ea0">


<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/056b0c61-ea4a-4996-b1e5-68c93d694981">



#### Stage 0

<img width="400" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/32377e25-91b1-4cd6-b626-a6f38b40ce0e">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/b5923053-d566-4543-bb4a-946b1e8b8e1c">

#### Stage 1

<img width="600" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/829aa613-8ee9-401f-b5bd-bac44b5a0416">
<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/93b0bf73-38ca-4e8e-bcc5-d54a43b37712">

#### Stage 2

<img width="400" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/9e32409c-dc74-4ee7-ab36-0f20046ce35f">
<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/618029df-dbad-4bb0-aa59-ac7026b1f5a4">

#### Stage 3

<img width="400" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/820df47a-daef-4dee-96e1-31443de94d32">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/ab1745df-9760-4d13-aa14-dd71dde87853">


#### Output files on HDFS
<img width="900" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/7bf4dc0d-8b18-469f-bfd5-9af0f07e5849">



#### Print debug strings and interpret the outputs

The Spark Driver keeps track of every RDD's lineage — that is, the series of transformations performed to yield an RDD or a partition thereof. This enables every RDD at every stage to be reevaluated in the event of a failure, which provides the resiliency in RDDs.

When you call `.toDebugString()` on an RDD, it provides a detailed description of the lineage of that RDD, showing how it was derived from other RDDs.

```python
>>> lines = sc.textFile("hdfs:///input/soc-LiveJournal1Adj.txt")
>>> print(lines.toDebugString().decode())
(2) hdfs:///input/soc-LiveJournal1Adj.txt MapPartitionsRDD[1] at textFile at NativeMethodAccessorImpl.java:0 []
 |  hdfs:///input/soc-LiveJournal1Adj.txt HadoopRDD[0] at textFile at NativeMethodAccessorImpl.java:0 []
```


- The number in parentheses `(2)` indicates the number of partitions in the MapPartitionsRDD.
- Each RDD created in a Spark application is assigned a unique ID when it is created. The numbers in square brackets `[1]` and `[0]` are the unique identifiers of the two RDDs.
- HadoopRDD is an RDD that represents the raw data source on a Hadoop-compatible file system, such as HDFS. The HadoopRDD doesn't immediately read the data; it defines how the data should be read. This definition includes information about the file path, input format, and partitioning (how the file is divided into partitions).
- After defining the `HadoopRDD`, Spark applies a map transformation to read the raw data from the file. Specifically,Spark applies a function to each partition of the HadoopRDD to convert the raw input records into lines of text. This explains why the signature of the resulting RDD is `MapPartitionsRDD`
- `at textFile` indicates that the RDD was created by a call to the `textFile()` method.
- `textFile` interacts with the Hadoop API, which is primarily written in Java. Spark also runs on the Java Virtual Machine (JVM). When certain methods are called, especially those related to I/O and file operations, the JVM might use native methods to perform these operations (e.g., file system libraries written in C/C++). This can sometimes lead to source locations being reported as `NativeMethodAccessorImpl.java:0`, where `0` indicates the lack of the precise line information, as the exact source line is not available.

[`.textFile()`](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/SparkContext.scala#L1094) uses the  [`org.apache.hadoop.fs.FileSystem`](https://github.com/apache/hadoop/blob/master/hadoop-common-project/hadoop-common/src/main/java/org/apache/hadoop/fs/FileSystem.java)

This class uses a utility class [`org.apache.hadoop.util.ReflectionUtils`](https://github.com/apache/hadoop/blob/master/hadoop-common-project/hadoop-common/src/main/java/org/apache/hadoop/util/ReflectionUtils.java)
that provides methods for dynamically creating objects and invoking methods using Java reflection.

When a method is invoked via reflection (e.g. using [`java.lang.reflect.Method.invoke()`](https://github.com/apache/hadoop/blob/master/hadoop-common-project/hadoop-common/src/main/java/org/apache/hadoop/util/ReflectionUtils.java#L109)), the JVM may use `NativeMethodAccessorImpl` internally to handle method invocations. If the method being invoked via reflection is a native method, the `NativeMethodAccessorImpl` handles the transition from the Java code to the native code.

Native Methods: These are methods declared in Java but implemented in another programming language, typically C or C++. Native methods are used to perform low-level operations that are platform-specific or require direct interaction with the operating system.

https://en.wikipedia.org/wiki/Java_Native_Interface

```python
>>> friend_lists = lines.map(lambda x: x.strip().split("\t")).filter(lambda x: len(x) == 2).mapValues(lambda x: x.split(","))
>>> print(friend_lists.toDebugString().decode())
(2) PythonRDD[2] at RDD at PythonRDD.scala:53 []
 |  hdfs:///input/soc-LiveJournal1Adj.txt MapPartitionsRDD[1] at textFile at NativeMethodAccessorImpl.java:0 []
 |  hdfs:///input/soc-LiveJournal1Adj.txt HadoopRDD[0] at textFile at NativeMethodAccessorImpl.java:0 []
```


- "PythonRDD" indicates that the RDD is now being processed using Python functions or lambda expressions. This change in signature reflects the type of RDD resulting from the applied transformations.

- "at PythonRDD.scala:53" indicates the location of the code file (PythonRDD.scala) and the line number (53) where the RDD is defined.



On an Amazon EMR cluster, the source files like `PythonRDD.scala` are not typically included in the pre-built binaries that come with the EMR installation. 

How to find Spark's installation directory?

```shell
[hadoop@ip-172-31-56-41 ~]$ echo 'sc.getConf.get("spark.home")' | spark-shell --master local
Jun 08, 2024 2:54:49 PM org.apache.spark.launcher.Log4jHotPatchOption staticJavaAgentOption
WARNING: spark.log4jHotPatch.enabled is set to true, but /usr/share/log4j-cve-2021-44228-hotpatch/jdk17/Log4jHotPatchFat.jar does not exist at the configured location

Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
Spark context Web UI available at http://ip-172-31-57-98.ec2.internal:4040
Spark context available as 'sc' (master = local, app id = local-1717858504144).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 3.5.0-amzn-1
      /_/
         
Using Scala version 2.12.17 (OpenJDK 64-Bit Server VM, Java 17.0.11)
Type in expressions to have them evaluated.
Type :help for more information.

scala> sc.getConf.get("spark.home")
res0: String = /usr/lib/spark

scala> :quit
[hadoop@ip-172-31-56-41 ~]$ cd /usr/lib/spark
[hadoop@ip-172-31-56-41 spark]$ pwd
/usr/lib/spark
[hadoop@ip-172-31-56-41 spark]$ find . -name PythonRDD.scala
[hadoop@ip-172-31-56-41 spark]$ jar tf jars/spark-core_*.jar | grep PythonRDD
org/apache/spark/api/python/PythonRDD.class
org/apache/spark/api/python/PythonRDD$.class
org/apache/spark/api/python/PythonRDDServer.class
```

The source code of a particular Spark version can be found at https://archive.apache.org/dist/spark/
For example, the source code for Spark 5.1.0 is in spark-3.5.0.tgz at https://archive.apache.org/dist/spark/spark-3.5.0/.    

https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/api/python/



```python
>>> already_friend_pairs = friend_lists.flatMap(lambda x: [(int(x[0]), int(item)) for item in x[1]]) \
... .map(lambda x: x if x[0] <= x[1] else (x[1], x[0])).distinct()
>>> print(already_friend_pairs.toDebugString().decode())
(2) PythonRDD[7] at RDD at PythonRDD.scala:53 []
 |  MapPartitionsRDD[6] at mapPartitions at PythonRDD.scala:160 []
 |  ShuffledRDD[5] at partitionBy at NativeMethodAccessorImpl.java:0 []
 +-(2) PairwiseRDD[4] at distinct at <stdin>:1 []
    |  PythonRDD[3] at distinct at <stdin>:1 []
    |  hdfs:///input/soc-LiveJournal1Adj.txt MapPartitionsRDD[1] at textFile at NativeMethodAccessorImpl.java:0 []
    |  hdfs:///input/soc-LiveJournal1Adj.txt HadoopRDD[0] at textFile at NativeMethodAccessorImpl.java:0 []
```


```python
>>> already_friend_pairs.cache()
PythonRDD[7] at RDD at PythonRDD.scala:53
>>> print(already_friend_pairs.toDebugString().decode())
(2) PythonRDD[7] at RDD at PythonRDD.scala:53 [Memory Serialized 1x Replicated]
 |  MapPartitionsRDD[6] at mapPartitions at PythonRDD.scala:160 [Memory Serialized 1x Replicated]
 |  ShuffledRDD[5] at partitionBy at NativeMethodAccessorImpl.java:0 [Memory Serialized 1x Replicated]
 +-(2) PairwiseRDD[4] at distinct at <stdin>:1 [Memory Serialized 1x Replicated]
    |  PythonRDD[3] at distinct at <stdin>:1 [Memory Serialized 1x Replicated]
    |  hdfs:///input/soc-LiveJournal1Adj.txt MapPartitionsRDD[1] at textFile at NativeMethodAccessorImpl.java:0 [Memory Serialized 1x Replicated]
    |  hdfs:///input/soc-LiveJournal1Adj.txt HadoopRDD[0] at textFile at NativeMethodAccessorImpl.java:0 [Memory Serialized 1x Replicated]
```

```python
>>> from itertools import combinations
>>> potential_pairs = friend_lists.flatMap(lambda x: combinations(x[1], 2)).map(lambda x: (int(x[0]), int(x[1])))
>>> print(potential_pairs.toDebugString().decode())
(2) PythonRDD[8] at RDD at PythonRDD.scala:53 []
 |  hdfs:///input/soc-LiveJournal1Adj.txt MapPartitionsRDD[1] at textFile at NativeMethodAccessorImpl.java:0 []
 |  hdfs:///input/soc-LiveJournal1Adj.txt HadoopRDD[0] at textFile at NativeMethodAccessorImpl.java:0 []
```



```python
>>> rec_pairs = potential_pairs.subtract(already_friend_pairs)
>>> print(rec_pairs.toDebugString().decode())
(4) PythonRDD[16] at RDD at PythonRDD.scala:53 []
 |  MapPartitionsRDD[15] at mapPartitions at PythonRDD.scala:160 []
 |  ShuffledRDD[14] at partitionBy at NativeMethodAccessorImpl.java:0 []
 +-(4) PairwiseRDD[13] at subtract at <stdin>:1 []
    |  PythonRDD[12] at subtract at <stdin>:1 []
    |  UnionRDD[11] at union at NativeMethodAccessorImpl.java:0 []
    |  PythonRDD[9] at RDD at PythonRDD.scala:53 []
    |  hdfs:///input/soc-LiveJournal1Adj.txt MapPartitionsRDD[1] at textFile at NativeMethodAccessorImpl.java:0 []
    |  hdfs:///input/soc-LiveJournal1Adj.txt HadoopRDD[0] at textFile at NativeMethodAccessorImpl.java:0 []
    |  PythonRDD[10] at RDD at PythonRDD.scala:53 []
    |  PythonRDD[7] at RDD at PythonRDD.scala:53 []
    |  MapPartitionsRDD[6] at mapPartitions at PythonRDD.scala:160 []
    |  ShuffledRDD[5] at partitionBy at NativeMethodAccessorImpl.java:0 []
    +-(2) PairwiseRDD[4] at distinct at <stdin>:1 []
       |  PythonRDD[3] at distinct at <stdin>:1 []
       |  hdfs:///input/soc-LiveJournal1Adj.txt MapPartitionsRDD[1] at textFile at NativeMethodAccessorImpl.java:0 []
       |  hdfs:///input/soc-LiveJournal1Adj.txt HadoopRDD[0] at textFile at NativeMethodAccessorImpl.java:0 []
```

```python
>>> output_pairs = rec_pairs.map(lambda x: (x, 1)).reduceByKey(lambda a,b: a+b)
>>> print(output_pairs.toDebugString().decode())
(4) PythonRDD[21] at RDD at PythonRDD.scala:53 []
 |  MapPartitionsRDD[20] at mapPartitions at PythonRDD.scala:160 []
 |  ShuffledRDD[19] at partitionBy at NativeMethodAccessorImpl.java:0 []
 +-(4) PairwiseRDD[18] at reduceByKey at <stdin>:1 []
    |  PythonRDD[17] at reduceByKey at <stdin>:1 []
    |  MapPartitionsRDD[15] at mapPartitions at PythonRDD.scala:160 []
    |  ShuffledRDD[14] at partitionBy at NativeMethodAccessorImpl.java:0 []
    +-(4) PairwiseRDD[13] at subtract at <stdin>:1 []
       |  PythonRDD[12] at subtract at <stdin>:1 []
       |  UnionRDD[11] at union at NativeMethodAccessorImpl.java:0 []
       |  PythonRDD[9] at RDD at PythonRDD.scala:53 []
       |  hdfs:///input/soc-LiveJournal1Adj.txt MapPartitionsRDD[1] at textFile at NativeMethodAccessorImpl.java:0 []
       |  hdfs:///input/soc-LiveJournal1Adj.txt HadoopRDD[0] at textFile at NativeMethodAccessorImpl.java:0 []
       |  PythonRDD[10] at RDD at PythonRDD.scala:53 []
       |  PythonRDD[7] at RDD at PythonRDD.scala:53 []
       |  MapPartitionsRDD[6] at mapPartitions at PythonRDD.scala:160 []
       |  ShuffledRDD[5] at partitionBy at NativeMethodAccessorImpl.java:0 []
       +-(2) PairwiseRDD[4] at distinct at <stdin>:1 []
          |  PythonRDD[3] at distinct at <stdin>:1 []
          |  hdfs:///input/soc-LiveJournal1Adj.txt MapPartitionsRDD[1] at textFile at NativeMethodAccessorImpl.java:0 []
          |  hdfs:///input/soc-LiveJournal1Adj.txt HadoopRDD[0] at textFile at NativeMethodAccessorImpl.java:0 []
```
