
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


SSHing into the primary node of the same instance type to further verifies that there were 2 CPUs (1 core each) at work in each core instance.

<img width="883" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/eb78025d-bd72-4102-9c4b-aa9e9ce506cc">



## YARN Resource Manager Web UI

It shows that we logged in as hadoop. This is because we set `hadoop.http.staticuser.user` to `hadoop` in the EMR launch wizard before the cluster is spin up. Otherwise, it will be shown as "logged in as: dr.who").

 <img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/acddf4d5-1bb4-407d-a0ff-3d3c5ac3060f">


The cluster metrics section shows that there are 24 GB memory and 16 vCores. So, YARN sees 6 GB memory and 4 vCores per core instance. 

All effective configurations can be found in the configuration tab (URL:`http://<primary-node-dns>:8088/conf`):

<img width="150" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/5d49060a-c976-4493-a809-b4a640b6f500">


```xml
<property>
<name>yarn.nodemanager.resource.cpu-vcores</name>
<value>4</value>
<final>false</final>
<source>yarn-site.xml</source>
...
<property>
<name>yarn.nodemanager.resource.memory-mb</name>
<value>6144</value>
<final>false</final>
<source>yarn-site.xml</source>
</property>

```
 


When launching a shell, make sure to set executors' idle timeout (`spark.dynamicAllocation.executorIdleTimeout`) to a longer time interval (e.g., 10 minutes). 

The default timeout is 60s (set [here](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/config/package.scala#L707) ? as it's not present in files *spark-defaults.conf* and *spark-env.sh* under both directories */etc/spark/conf* and */usr/lib/spark/conf* ). If we were not to configure the property to a longer time interval, idle executors would be automatically removed after 1 minute.

We don't need to specify the `--deploy-mode` flag, because spark shells can only run in client mode. If you try to launch a shell in cluster mode, you'll see an error message as follows: 

<img width="900" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/6606ba7c-8309-4ccf-bfa9-ad135e727266">

In the above screenshot, although we see the prompt changes to `>>>` above,  `spark` and `sc` were not successfully initialized.

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
| ip-xxxx-48-39  | core | executor 4 (4 cores; 2G mem) and the application master (1 core) | 2 |
| ip-xxxx-56-172  | core | executor 1  (4 cores; 2G mem)| 1 |
| ip-xxxx-59-175  | core |  executor 2 (4 cores; 2G mem) | 1 |
| ip-xxxx-51-151  | core |  executor 3 (4 cores; 2G mem) | 1 |
| ip-xxxx-52-12 | primary |  client: Pyspark shell with the driver process (0 cores; 1G mem)<br>running inside it | 0 |



Deploy mode is client

| Variable | Property Key  | Value | Meaning | Set via |
| ----- | ------------- |-------------| ------------- |------------- |
|`executorMemory`|`spark.executor.memory`| 4269M | Amount of memory to use per executor process| [Client.scala#L119](https://github.com/apache/spark/blob/master/resource-managers/yarn/src/main/scala/org/apache/spark/deploy/yarn/Client.scala#L119)|
| `executorMemoryOverhead` |`spark.executor.memoryOverhead`|executorMemory * 0.1, with minimum of 384M |Amount of additional memory to be allocated per executor process, in MiB unless otherwise specified. This is memory that accounts for things like VM overheads, interned strings, other native overheads, etc.| [Client.scala#L125](https://github.com/apache/spark/blob/master/resource-managers/yarn/src/main/scala/org/apache/spark/deploy/yarn/Client.scala#L125)|
|`amMemory`|`spark.yarn.am.memory` |  512M | Amount of memory to use for the YARN Application Master in client mode. In cluster mode, use `spark.driver.memory` instead.| [Client.scala#L92](https://github.com/apache/spark/blob/master/resource-managers/yarn/src/main/scala/org/apache/spark/deploy/yarn/Client.scala#L92)|
|`amCores`|`spark.yarn.am.cores` |  1 | Number of cores to use for the YARN Application Master in client mode. In cluster mode, use spark.driver.cores instead.| [Client.scala#L112](https://github.com/apache/spark/blob/master/resource-managers/yarn/src/main/scala/org/apache/spark/deploy/yarn/Client.scala#L112) |
|`amMemoryOverhead`|`spark.yarn.am.memoryOverhead`| AM memory * 0.1, with minimum of 384M | Amount of non-heap memory for the YARN Application Master in client mode. This is memory that accounts for things like VM overheads, interned strings, other native overheads, etc. | [Client.scala#L105](https://github.com/apache/spark/blob/master/resource-managers/yarn/src/main/scala/org/apache/spark/deploy/yarn/Client.scala#L105)|
|`executorOffHeapMemory`|`spark.memory.offHeap.size`|0M| The absolute amount of memory which can be used for off-heap allocation |[Client.scala#L121](https://github.com/apache/spark/blob/master/resource-managers/yarn/src/main/scala/org/apache/spark/deploy/yarn/Client.scala#L121)|


 

in *spark-defaults.conf*:

```shell
spark.executor.memory            4269M
spark.emr.default.executor.memory 4269M
spark.driver.memory              2048M
spark.executor.cores             4
spark.emr.default.executor.cores 4
```

in [org/apache/spark/deploy/yarn/config.scala](https://github.com/apache/spark/blob/master/resource-managers/yarn/src/main/scala/org/apache/spark/deploy/yarn/config.scala)

```scala
  private[spark] val AM_CORES = ConfigBuilder("spark.yarn.am.cores")
    .version("1.3.0")
    .intConf
    .createWithDefault(1)
```

in [org/apache/spark/internal/config/package.scala](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/config/package.scala)
```java


  private[spark] val EXECUTOR_MIN_MEMORY_OVERHEAD =
    ConfigBuilder("spark.executor.minMemoryOverhead")
    .doc("The minimum amount of non-heap memory to be allocated per executor " +
      "in MiB unless otherwise specified. This value is ignored if " +
      "spark.executor.memoryOverhead is set directly.")
    .version("4.0.0")
    .bytesConf(ByteUnit.MiB)
    .createWithDefaultString("384m")

  private[spark] val EXECUTOR_MEMORY_OVERHEAD_FACTOR =
    ConfigBuilder("spark.executor.memoryOverheadFactor")
      .doc("Fraction of executor memory to be allocated as additional non-heap memory per " +
        "executor process. This is memory that accounts for things like VM overheads, " +
        "interned strings, other native overheads, etc. This tends to grow with the container " +
        "size. This value defaults to 0.10 except for Kubernetes non-JVM jobs, which defaults " +
        "to 0.40. This is done as non-JVM tasks need more non-JVM heap space and such tasks " +
        "commonly fail with \"Memory Overhead Exceeded\" errors. This preempts this error " +
        "with a higher default. This value is ignored if spark.executor.memoryOverhead is set " +
        "directly.")
      .version("3.3.0")
      .doubleConf
      .checkValue(factor => factor > 0,
        "Ensure that memory overhead is a double greater than 0")
      .createWithDefault(0.1)
```


> The maximum memory size of container to running executor is determined by the sum of spark.executor.memoryOverhead, spark.executor.memory, spark.memory.offHeap.size and spark.executor.pyspark.memory. https://spark.apache.org/docs/latest/configuration.html

Note that the primary instance is not part of the cluster's resource pool (because no NodeManager is running on it).

Recall that YARN sees 1 vCore per container. So, for an executor, 1 vCore seen by YARN gets mapped to 4 cores seen by Spark.

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
| ip-xxxx-48-39  | core | executor 3 (2 cores; 2GB mem)  | 1 |
| ip-xxxx-56-172  | core | executor 4  (2 cores; 2GB mem)| 1 |
| ip-xxxx-59-175  | core |  executor 2 (2 cores; 2GB mem) | 1 |
| ip-xxxx-51-151  | core |  executor 1 (2 cores; 2GB mem) and the application master (1 core) | 2|
| ip-xxxx-52-12 | primary |  client: Pyspark shell with the driver process (0 cores; 1G mem)<br>running inside it | 0 |


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
| ip-xxxx-48-39  | core | executors 3 & 4 (4 cores, 912M mem each)  | 1 |
| ip-xxxx-56-172  | core | executors 5 & 6  (4 cores, 912M mem each)| 1 |
| ip-xxxx-59-175  | core |  executors 1 & 2 (4 cores, 912M mem each)| 1 |
| ip-xxxx-51-151  | core |  executors 7 & 8 (4 cores, 912M mem each) and the application master<br>(1 core)| 2|
| ip-xxxx-52-12 | primary |  client: Pyspark shell with the driver process (0 cores; 1G mem)<br>running inside it| 0 |



<br>

### Experiment 4

```shell
pyspark --master yarn --executor-memory 2g --executor-cores 2 --conf spark.dynamicAllocation.executorIdleTimeout=10m
```

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/99cab554-4664-4eca-ade3-bacb4b273c38">

The resource allocation is similar except for 2 cores per executor.

| Instance ID | Instance Type | Software Entities | No. of Containers |
| ------------- |-------------| ------------- | ------------- |
| ip-xxxx-48-39  | core | executors 7 & 8 (2 cores, 912M mem each)  | 1 |
| ip-xxxx-56-172  | core | executors 5 & 6  (2 cores, 912M mem each)| 1 |
| ip-xxxx-59-175  | core |  executors 3 & 4 (2 cores, 912M mem each) | 1 |
| ip-xxxx-51-151  | core |  executors 1 & 2 (2 cores, 912M mem each) and the application master (1 core) | 2 |
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

Spark on EMR enables dynamic allocation. The `pyspark` help explains why this is the case:

> Spark on YARN and Kubernetes only:
  --num-executors NUM         Number of executors to launch (Default: 2).
                              If dynamic allocation is enabled, the initial number of
                              executors will be at least NUM.

<img width="700" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/4a444a26-0f3e-410f-ad05-9467a69f8d6a">


<br>

### Observations

It seems that:

- The number of executors is determined by the amount of memory per executor (configured via `--executor-memory`) and the total amount of memory available on the cluster.

- The number of cores each executor (the multiplier used to scale no. of vcores) owns can be specified by `--executor-cores`




#### 

 

