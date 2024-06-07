

## Dynamic allocation on EMR


###  Settings

- 1 primary instance; type: `m4.large`

- 4 core instances; type: `m4.large`
  
    <img width="300" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/1644cc8c-d79b-4c48-a194-f5c49478d126">

- EMR release: 7.1.0

- Run a .sh file at `s3://ust-bigdata-class/install_python_libraries.sh` as a bootstrap action

### AWS CLI

```shell
aws emr create-cluster \
 --name "My cluster" \
 --log-uri "s3://aws-logs-339712892718-us-east-1/elasticmapreduce" \
 --release-label "emr-7.1.0" \
 --service-role "arn:aws:iam::339712892718:role/EMR_DefaultRole" \
 --unhealthy-node-replacement \
 --ec2-attributes '{"InstanceProfile":"EMR_EC2_DefaultRole","EmrManagedMasterSecurityGroup":"sg-0e760c758acd676b7","EmrManagedSlaveSecurityGroup":"sg-091de8a5a48f351e5","KeyName":"vockey","AdditionalMasterSecurityGroups":[],"AdditionalSlaveSecurityGroups":[],"SubnetId":"subnet-00d31ecbf9ca1d2f3"}' \
 --applications Name=Hadoop Name=Hive Name=Spark \
 --configurations '[{"Classification":"core-site","Properties":{"hadoop.http.staticuser.user":"hadoop"}},{"Classification":"hdfs-site","Properties":{"dfs.replication":"3"}}]' \
 --instance-groups '[{"InstanceCount":4,"InstanceGroupType":"CORE","Name":"Core","InstanceType":"m4.large","EbsConfiguration":{"EbsBlockDeviceConfigs":[{"VolumeSpecification":{"VolumeType":"gp2","SizeInGB":32},"VolumesPerInstance":1}]}},{"InstanceCount":1,"InstanceGroupType":"MASTER","Name":"Primary","InstanceType":"m4.large","EbsConfiguration":{"EbsBlockDeviceConfigs":[{"VolumeSpecification":{"VolumeType":"gp2","SizeInGB":32},"VolumesPerInstance":1}]}}]' \
 --bootstrap-actions '[{"Args":[],"Name":"python_libraries","Path":"s3://ust-bigdata-class/install_python_libraries.sh"}]' \
 --scale-down-behavior "TERMINATE_AT_TASK_COMPLETION" \
 --auto-termination-policy '{"IdleTimeout":3600}' \
 --region "us-east-1"
```

--

YARN resource manager Web UI. It shows that we logged in as hadoop. This is because we set `hadoop.http.staticuser.user` to `hadoop` in the EMR launch wizard before the cluster is spin off. Otherwise, it will be shown as "logged in as: dr.who").

 <img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/acddf4d5-1bb4-407d-a0ff-3d3c5ac3060f">



 

The cluster metrics section shows that there are 24 GB memory and 16 vCores.
It seems that YARN sees 6 GB memory and 4 vCores per core instance. 

SSHing into the primary node of the same instance type to further verifies that there were 2 CPUs (1 core each) at work in each core instance.

<img width="883" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/eb78025d-bd72-4102-9c4b-aa9e9ce506cc">



This confusion seems to arise from the default configuration for the `yarn.nodemanager.resource.cpu-vcores` property. 

<img width="150" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/5d49060a-c976-4493-a809-b4a640b6f500">


`http://<primary-node-dns>:8088/conf`

```xml
<property>
<name>yarn.nodemanager.resource.cpu-vcores</name>
<value>4</value>
<final>false</final>
<source>yarn-site.xml</source>
```
 <img width="766" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/85e92972-c24f-4647-89ce-95acd5ca238f">

More on this confusion: https://repost.aws/questions/QUmbShfKT4ShOy1IX8T6Exng/difference-in-vcore-and-vcpu-ec2-and-emr


When launching a shell, make sure to set executors' idle timeout (`spark.dynamicAllocation.executorIdleTimeout`) to a longer time interval (e.g., 10 minutes).
The default timeout is 60s. If we were not to configure the property to a longer time interval, idle executors would be automatically removed after 1 minute.

We don't need to specify the `--deploy-mode` flag, because spark shells can only run in client mode. If you try to launch a shell in cluster mode, you'll see an error message as follows: 

<img width="900" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/6606ba7c-8309-4ccf-bfa9-ad135e727266">

Although we see the prompt changes to `>>>` above,  `spark` and `sc` were not successfully initialized.

<br>

### Experiment 1

```shell
pyspark --master yarn --conf spark.dynamicAllocation.executorIdleTimeout=10m
```

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/87c1ca88-170e-40c5-944f-ac0a94dd2bd2">


 
5 containers are created and spread across the 4 core instances. one vCore is used by each container.
the instance `ip-xxxx-48-39` hosts 2 containers.


A Spark application ((id: `application_1717748984266_0001`)) is created.

 <img width="700" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/45f2e918-1f60-4f7e-a546-cc22a9d026b2">


4 executors are created for this application
 
 <img width="700" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/0071bdd9-de48-4193-a181-fbed3a61d39c">


 
<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/135ec1b2-fb40-452b-82d5-c3f00febf746">


 <img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/20b58a73-57ca-46b9-a4c1-9dcfe13f7369">


4 containers (4.97 GB mem and 1 vCore per container) are allocated to host the 4 executors,
whereas 1 container (896 MB and 1 vCore) is allocated to host the application master.


| Instance ID | Instance Type | Software Entities | No. of Containers |
| ------------- |-------------| ------------- | ------------- |
| ip-xxxx-48-39  | core | executor 4 (4 cores and 2GB mem) and the application master (1 core) | 2 |
| ip-xxxx-56-172  | core | executor 1  (4 cores and 2GB mem)| 1 |
| ip-xxxx-59-175  | core |  executor 2 (4 cores and 2GB mem) | 1 |
| ip-xxxx-51-151  | core |  executor 3 (4 cores and 2GB mem)| 1 |
| ip-xxxx-52-12 | primary |  client: Pyspark shell with the driver process running inside it | 0 |

Note that the primary instance is not part of the cluster's resource pool (because no NodeManager is running on it).

Recall that YARN sees 1 vCore per container. So, for an executor, 1 vCore seen by YARN gets mapped to 4 cores seen by Spark.
No cores are assigned to the driver. (does it imply that the driver is not running on any of the worker nodes?)

After 10 minutes, all executors are removed automatically. Only the application master and the driver stay alive.

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/cbade3c8-8025-4d4a-8328-dd461d6f93da">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/4244790b-2a2b-4dff-9a2b-c5bc8a3bf606">

 
<br>

### Experiment 2

```shell
pyspark --master yarn --executor-cores 2 --conf spark.dynamicAllocation.executorIdleTimeout=10m
```

A new Spark application (id: `application_1717748984266_0002`) is created. 5 containers are created by YARN to host 4 executors and 1 application master.

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/2dc4ee09-e26a-4ce9-8dbf-32aefdc62a06">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/97e7547d-7483-4e29-b10f-b8209d1fa6f2">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/54b7d2f6-e825-421e-8dc2-5223495a6cff">

The number of cores per executor seems to be affected by the `--executor-cores 2` flag.
Now 1 vCore seen by YARN gets mapped to 2 cores seen by Spark.

| Instance ID | Instance Type | Software Entities | No. of Containers |
| ------------- |-------------| ------------- | ------------- |
| ip-xxxx-48-39  | core | executor 3 (2 cores and 2GB mem)  | 1 |
| ip-xxxx-56-172  | core | executor 4  (2 cores and 2GB mem)| 1 |
| ip-xxxx-59-175  | core |  executor 2 (2 cores and 2GB mem) | 1 |
| ip-xxxx-51-151  | core |  executor 1 (2 cores and 2GB mem) and the application master (1 core) | 2|
| ip-xxxx-52-12 | primary |  client: Pyspark shell with the driver process running inside it | 0 |


<br>

### Experiment 3

```shell
pyspark --master yarn --executor-memory 2g --conf spark.dynamicAllocation.executorIdleTimeout=10m
``` 

A new Spark application (id: `application_1717748984266_0003`) is created. 

8 containers (2.38 GB mem and 1 vCore per container) are allocated to host the 4 executors,
whereas 1 container (896 MB and 1 vCore) is allocated to host the application master.

> † 2.38 GB memory above has been verified by first letting all executors die out and submitting a job that requires only 1 executor.

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/a5a549b3-4604-4248-a0d6-3aaa974233e1">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/e14a3b7a-bc4c-4205-8605-2bd12e076db0">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/5e231131-a299-4f12-a7f8-06d6d0fe4d7e">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/bd8f649a-d0b9-4bdb-a259-b307c4f60fd2">

8 executors are created for this Spark application. 

| Instance ID | Instance Type | Software Entities | No. of Containers |
| ------------- |-------------| ------------- | ------------- |
| ip-xxxx-48-39  | core | executors 3 & 4 (4 cores and 912M mem / executor)  | 1 |
| ip-xxxx-56-172  | core | executors 5 & 6  (4 cores and 912M mem / executor)| 1 |
| ip-xxxx-59-175  | core |  executors 1 & 2 (4 cores and 912M mem / executor) | 1 |
| ip-xxxx-51-151  | core |  executors 7 & 8 (4 cores and 912M mem / executor ) and the application master (1 core) | 2|
| ip-xxxx-52-12 | primary |  client: Pyspark shell with the driver process running inside it | 0 |



<br>

### Experiment 4

```shell
pyspark --master yarn --executor-memory 2g --executor-cores 2 --conf spark.dynamicAllocation.executorIdleTimeout=10m
```

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/99cab554-4664-4eca-ade3-bacb4b273c38">

The resource allocation is similar except for 2 cores per executor.

| Instance ID | Instance Type | Software Entities | No. of Containers |
| ------------- |-------------| ------------- | ------------- |
| ip-xxxx-48-39  | core | executors 7 & 8 (2 cores and 912M mem / executor)  | 1 |
| ip-xxxx-56-172  | core | executors 5 & 6  (2 cores and 912M mem / executor)| 1 |
| ip-xxxx-59-175  | core |  executors 3 & 4 (2 cores and 912M mem / executor) | 1 |
| ip-xxxx-51-151  | core |  executors 1 & 2 (2 cores and 912M mem / executor ) and the application master (1 core) | 2 |
| ip-xxxx-52-12 | primary |  client: Pyspark shell with the driver process running inside it | 0 |


<br>

### Experiment 5

```shell
pyspark --master yarn --num-executors 4 --executor-memory 2g --executor-cores 3 --conf spark.dynamicAllocation.executorIdleTimeout=10m
```

Still, 9 containers are created by YARN.

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/1df913ba-1424-4251-a7ef-4828cad745df">


<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/fe6edead-a504-4040-bfad-380389e79639">
<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/66aafb5f-fdc3-4ef1-b9f0-e6de3ee350ab">

Even though we've explicitly specified the number of executors to 4, Spark still creates 8 executors (3 cores and 912M mem per executor).

<img width="700" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/4a444a26-0f3e-410f-ad05-9467a69f8d6a">


<br>

### Observations

It seems that:

- The number of executor is determined by the amount of memory per executor (configured via `--executor-memory`) and the total amount of memory available on the cluster.

- The number of cores each executor (the multiplier used to scale no. of vcores) owns can be specified by `--executor-cores`





# Other experiments


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
>>> lines = sc.textFile("hdfs:///input/soc-LiveJournal1Adj.txt")
>>> friend_lists = lines.map(lambda x: x.strip().split("\t")).filter(lambda x: len(x) == 2).mapValues(lambda x: x.split(","))
>>> already_friend_pairs = friend_lists.flatMap(lambda x: [(int(x[0]), int(item)) for item in x[1]]) \
... .map(lambda x: x if x[0] <= x[1] else (x[1], x[0])).distinct()
>>> already_friend_pairs.cache()
PythonRDD[6] at RDD at PythonRDD.scala:53
>>> from itertools import combinations
>>> potential_pairs = friend_lists.flatMap(lambda x: combinations(x[1], 2)).map(lambda x: (int(x[0]), int(x[1])))
>>> print(potential_pairs.toDebugString().decode())
(2) PythonRDD[7] at RDD at PythonRDD.scala:53 []
 |  hdfs:///input/soc-LiveJournal1Adj.txt MapPartitionsRDD[1] at textFile at NativeMethodAccessorImpl.java:0 []
 |  hdfs:///input/soc-LiveJournal1Adj.txt HadoopRDD[0] at textFile at NativeMethodAccessorImpl.java:0 []
>>> rec_pairs = potential_pairs.subtract(already_friend_pairs)
>>> print(rec_pairs.toDebugString().decode())
(4) PythonRDD[15] at RDD at PythonRDD.scala:53 []
 |  MapPartitionsRDD[14] at mapPartitions at PythonRDD.scala:160 []
 |  ShuffledRDD[13] at partitionBy at NativeMethodAccessorImpl.java:0 []
 +-(4) PairwiseRDD[12] at subtract at <stdin>:1 []
    |  PythonRDD[11] at subtract at <stdin>:1 []
    |  UnionRDD[10] at union at NativeMethodAccessorImpl.java:0 []
    |  PythonRDD[8] at RDD at PythonRDD.scala:53 []
    |  hdfs:///input/soc-LiveJournal1Adj.txt MapPartitionsRDD[1] at textFile at NativeMethodAccessorImpl.java:0 []
    |  hdfs:///input/soc-LiveJournal1Adj.txt HadoopRDD[0] at textFile at NativeMethodAccessorImpl.java:0 []
    |  PythonRDD[9] at RDD at PythonRDD.scala:53 []
    |  PythonRDD[6] at RDD at PythonRDD.scala:53 []
    |  MapPartitionsRDD[5] at mapPartitions at PythonRDD.scala:160 []
    |  ShuffledRDD[4] at partitionBy at NativeMethodAccessorImpl.java:0 []
    +-(2) PairwiseRDD[3] at distinct at <stdin>:1 []
       |  PythonRDD[2] at distinct at <stdin>:1 []
       |  hdfs:///input/soc-LiveJournal1Adj.txt MapPartitionsRDD[1] at textFile at NativeMethodAccessorImpl.java:0 []
       |  hdfs:///input/soc-LiveJournal1Adj.txt HadoopRDD[0] at textFile at NativeMethodAccessorImpl.java:0 []
>>> rec_pairs.map(lambda x: (x, 1)).reduceByKey(lambda a,b: a+b).saveAsTextFile("hdfs:///rec_pairs_output")
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

