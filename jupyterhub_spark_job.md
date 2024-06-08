### Settings

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



Note: Installing JupyterHub only won't download and install Python libraries such as NumPy. We still need to run a bootstrap action to downloand and install needed Python libraries.



<img width="1409" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/74a924d5-2c13-4fc9-b82a-7e9fe85a20b8">

No containers are allocated until we run some Python code in a notebook cell.

<br>




### Experiment 1

How to modify a spark application with custom configurations: https://repost.aws/knowledge-center/modify-spark-configuration-emr-notebook

In a Jupyter notebook cell, run the `%%configure` command with desired configurations:

```python
%%configure -f
{"conf": {
    "spark.dynamicAllocation.executorIdleTimeout": "5m"}  
}
```
For example, we may want to increase executors' idle timeout. Otherwise, executors will be automatically removed after 1 minute.

<img width="800" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/fde09276-dc92-45cf-9cad-ac957890cb52">

Running any code will start a new application on YARN with the custom configurations.


<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/e05f75f2-880c-4199-98e9-4952cb57ba02">


<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/4bbb92ac-67d7-4912-90f8-597c054786f3">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/d066c0d1-a265-4228-8129-acf97887b71d">


| Instance ID | Instance Type | Software Entities | No. of Containers |
| ------------- |-------------| ------------- | ------------- |
| ip-xxxx-63-62  | core | driver (0 core; 1G mem) | 1 (1 vCore; 2.38G mem) |
| ip-xxxx-54-228  | core | executor 1  (4 cores; 2G mem)| 1 (1 vCore; 4.97G mem)|
| ip-xxxx-48-235  | core |  executor 2 (4 cores; 2G mem) | 1 (1 vCore; 4.97G mem)|
| ip-xxxx-58-45  | core |  executor 3 (4 cores; 2G mem)| 1 (1 vCore; 4.97G mem)|


Note that the driver process now is started on a core instance. And there's no application master displayed on this page (no **Miscellaneous process** section).

> A Jupyter notebook uses the Sparkmagic kernel as a client for interactively working with Spark in a remote EMR cluster through an Apache Livy server.  https://repost.aws/knowledge-center/modify-spark-configuration-emr-notebook


Later, running the configuration cell every time will launch a new application with new configurations.

<br>

### Experiment 2


```python
%%configure -f
{"conf": {
    "spark.executor.cores": "2", 
    "spark.dynamicAllocation.executorIdleTimeout": "5m"} 
}
```


<img width="800" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/eb020ee4-7978-48b7-9d78-0d85ec297894">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/da36ada8-af96-4d27-bb69-d41530c33f20">
<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/1d384489-6d97-43a2-80c5-23838923f4c7">

| Instance ID | Instance Type | Software Entities | No. of Containers |
| ------------- |-------------| ------------- | ------------- |
| ip-xxxx-58-45  | core | driver (0 core; 1G mem) | 1 (1 vCore; 2.38G mem)|
| ip-xxxx-48-235  | core | executor 1  (2 cores; 2G mem)| 1 (1 vCore; 4.97G mem)|
| ip-xxxx-54-228  | core |  executor 2 (2 cores; 2G mem) | 1 (1 vCore; 4.97G mem)|
| ip-xxxx-63-62  | core |  executor 3 (2 cores; 2G mem)| 1 (1 vCore; 4.97G mem)|


<br>

### Experiment 3


```python
%%configure -f
{"conf": {
    "spark.executor.cores": "3", 
    "spark.executor.memory": "2g",
    "spark.dynamicAllocation.executorIdleTimeout": "5m"}
}
```

<img width="874" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/d8d6f227-1995-4538-9274-60571fcb8076">

<img width="1404" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/e59d0ecf-c538-4259-b534-f57c4eb578ac">

<img width="1405" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/68367571-16ef-409b-920c-f7c877342be7">

<img width="1429" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/9a2ebe4b-4c10-4c54-b0c8-03897ecf4a32">

| Instance ID | Instance Type | Software Entities | No. of Containers |
| ------------- |-------------| ------------- | ------------- |
| ip-xxxx-58-45  | core | driver (0 core; 1G mem) & executor 3 (3 cores, 912M mem) | 2 |
| ip-xxxx-48-235  | core | executors 4 & 5 (3 cores, 912M mem each)| 2  |
| ip-xxxx-54-228  | core |  executors 1 & 2 (3 cores, 912M mem each) | 2  |
| ip-xxxx-63-62  | core |  executors 6 & 7 (3 cores, 912M mem each)| 2  |

After all executors die out, it can be verified that the memory allocated to the container for driver is still 2.38G.


<br>

### Observations

To summarize:
```
%%configure -f
{"conf": {
    "spark.executor.instances": "6",     # does't take effect
    "spark.executor.cores": "3",         # take effect
    "spark.executor.memory": "2g",       # take effect; can affect the actual no. of executors
    "spark.dynamicAllocation.executorIdleTimeout": "5m"}  # take effect
}
```

It seems that a spark application created in this way can only run in cluster mode.

`'spark.submit.deployMode'` defaults to `'cluster'` (`sc.getConf().get('spark.submit.deployMode')`) and seems to be unmodifiable?




<img width="800" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/45703223-2ec8-4117-a9f5-d7d5ff38e135">

<img width="800" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/3cfb6080-5e9b-4b7e-a71b-2051e5db9f5a">

This also explains why the driver process runs on a worker machine. 

https://spark.apache.org/docs/latest/configuration.html


<br>

### Analyze livejournal data


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
```

```python
print(output_pairs.toDebugString().decode())
(4) PythonRDD[20] at RDD at PythonRDD.scala:53 []
 |  MapPartitionsRDD[19] at mapPartitions at PythonRDD.scala:160 []
 |  ShuffledRDD[18] at partitionBy at NativeMethodAccessorImpl.java:0 []
 +-(4) PairwiseRDD[17] at reduceByKey at <stdin>:12 []
    |  PythonRDD[16] at reduceByKey at <stdin>:12 []
    |  MapPartitionsRDD[15] at mapPartitions at PythonRDD.scala:160 []
    |  ShuffledRDD[14] at partitionBy at NativeMethodAccessorImpl.java:0 []
    +-(4) PairwiseRDD[13] at subtract at <stdin>:11 []
       |  PythonRDD[12] at subtract at <stdin>:11 []
       |  UnionRDD[11] at union at NativeMethodAccessorImpl.java:0 []
       |  PythonRDD[9] at RDD at PythonRDD.scala:53 []
       |  hdfs:///input/soc-LiveJournal1Adj.txt MapPartitionsRDD[3] at textFile at NativeMethodAccessorImpl.java:0 []
       |  hdfs:///input/soc-LiveJournal1Adj.txt HadoopRDD[2] at textFile at NativeMethodAccessorImpl.java:0 []
       |  PythonRDD[10] at RDD at PythonRDD.scala:53 []
       |  PythonRDD[8] at RDD at PythonRDD.scala:53 []
       |  MapPartitionsRDD[7] at mapPartitions at PythonRDD.scala:160 []
       |  ShuffledRDD[6] at partitionBy at NativeMethodAccessorImpl.java:0 []
       +-(2) PairwiseRDD[5] at distinct at <stdin>:4 []
          |  PythonRDD[4] at distinct at <stdin>:4 []
          |  hdfs:///input/soc-LiveJournal1Adj.txt MapPartitionsRDD[3] at textFile at NativeMethodAccessorImpl.java:0 []
          |  hdfs:///input/soc-LiveJournal1Adj.txt HadoopRDD[2] at textFile at NativeMethodAccessorImpl.java:0 []
```

```python
output_pairs.saveAsTextFile("hdfs:///rec_pairs_output")
```

<img width="675" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/e7ad4ec8-81ab-411f-946b-91a633853910">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/401b034c-49d0-4940-b1e4-4bbb0edcd2db">


<img width="370" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/56cbeb8e-3601-4ba1-82dc-20fbcc83f425">


<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/5c489836-fce2-4bad-b1e4-de505ea5a0fa">

<img width="511" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/f77f87d3-705f-4ac8-a468-013c35ee3b4f">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/2de052c9-38b6-4358-a249-eb611b408c06">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/6db91dff-4b89-4f97-bf09-ce3bbc36397e">


<img width="375" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/4d71f8d5-c7dd-4ee7-a494-bd5a4b5618ff">
<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/f39923fc-4558-4c76-a882-8083fc40b9cf">

<img width="402" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/29e425af-a80f-4c53-ad70-c1f03bc88488">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/9f729d62-f674-4cc0-b556-104204f68364">


```python
print(output_pairs.toDebugString().decode())
(4) PythonRDD[20] at RDD at PythonRDD.scala:53 []
 |  MapPartitionsRDD[19] at mapPartitions at PythonRDD.scala:160 []
 |  ShuffledRDD[18] at partitionBy at NativeMethodAccessorImpl.java:0 []
 +-(4) PairwiseRDD[17] at reduceByKey at <stdin>:12 []
    |  PythonRDD[16] at reduceByKey at <stdin>:12 []
    |  MapPartitionsRDD[15] at mapPartitions at PythonRDD.scala:160 []
    |  ShuffledRDD[14] at partitionBy at NativeMethodAccessorImpl.java:0 []
    +-(4) PairwiseRDD[13] at subtract at <stdin>:11 []
       |  PythonRDD[12] at subtract at <stdin>:11 []
       |  UnionRDD[11] at union at NativeMethodAccessorImpl.java:0 []
       |  PythonRDD[9] at RDD at PythonRDD.scala:53 []
       |  hdfs:///input/soc-LiveJournal1Adj.txt MapPartitionsRDD[3] at textFile at NativeMethodAccessorImpl.java:0 []
       |  hdfs:///input/soc-LiveJournal1Adj.txt HadoopRDD[2] at textFile at NativeMethodAccessorImpl.java:0 []
       |  PythonRDD[10] at RDD at PythonRDD.scala:53 []
       |  PythonRDD[8] at RDD at PythonRDD.scala:53 []
       |      CachedPartitions: 2; MemorySize: 2.5 MiB; DiskSize: 0.0 B
       |  MapPartitionsRDD[7] at mapPartitions at PythonRDD.scala:160 []
       |  ShuffledRDD[6] at partitionBy at NativeMethodAccessorImpl.java:0 []
       +-(2) PairwiseRDD[5] at distinct at <stdin>:4 []
          |  PythonRDD[4] at distinct at <stdin>:4 []
          |  hdfs:///input/soc-LiveJournal1Adj.txt MapPartitionsRDD[3] at textFile at NativeMethodAccessorImpl.java:0 []
          |  hdfs:///input/soc-LiveJournal1Adj.txt HadoopRDD[2] at textFile at NativeMethodAccessorImpl.java:0 []
```

```python
output_pairs.saveAsTextFile("hdfs:///rec_pairs_output_1")
```
<img width="704" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/cfda56ff-f121-4d71-b472-d914a2f35ceb">
<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/1b11255e-d2e5-4b5c-afcd-1703a6d27f58">

<img width="404" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/061cda36-9eaa-4671-af3a-e32caf5d93cd">
<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/b09bfebc-478d-4b40-b1e5-12bffc842535">


```python
print(already_friend_pairs.toDebugString().decode())
(2) PythonRDD[8] at RDD at PythonRDD.scala:53 [Memory Serialized 1x Replicated]
 |       CachedPartitions: 2; MemorySize: 2.5 MiB; DiskSize: 0.0 B
 |  MapPartitionsRDD[7] at mapPartitions at PythonRDD.scala:160 [Memory Serialized 1x Replicated]
 |  ShuffledRDD[6] at partitionBy at NativeMethodAccessorImpl.java:0 [Memory Serialized 1x Replicated]
 +-(2) PairwiseRDD[5] at distinct at <stdin>:4 [Memory Serialized 1x Replicated]
    |  PythonRDD[4] at distinct at <stdin>:4 [Memory Serialized 1x Replicated]
    |  hdfs:///input/soc-LiveJournal1Adj.txt MapPartitionsRDD[3] at textFile at NativeMethodAccessorImpl.java:0 [Memory Serialized 1x Replicated]
    |  hdfs:///input/soc-LiveJournal1Adj.txt HadoopRDD[2] at textFile at NativeMethodAccessorImpl.java:0 [Memory Serialized 1x Replicated]
```

```python
already_friend_pairs.saveAsTextFile("hdfs:///pairs_output")
```

<img width="338" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/59845d39-4911-49bb-a6e6-b3abdf002e94">
<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/0d7c2e0d-18be-407c-97b6-102ff005ea84">

<img width="337" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/50202e37-6ab6-43c6-a12f-f430568bcbb2">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/e8ce7803-ea24-4820-9311-6b79c4984493">



