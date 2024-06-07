Dynamic allocation on EMR with YARN

Settings:
1 primary instance; type: m4.large
4 core instances; type: 4 m4.large 

<img width="300" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/1644cc8c-d79b-4c48-a194-f5c49478d126">


EMR release: 7.1.0

--

YARN resource manager Web UI. It shows that we logged in as hadoop. This is because we set `hadoop.http.staticuser.user` to `hadoop` in the EMR launch wizard before the cluster is spin off. Otherwise, it will be shown as "logged in as: dr.who").

 <img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/acddf4d5-1bb4-407d-a0ff-3d3c5ac3060f">



 

The cluster metrics section shows that there are 24 GB memory and 16 vCores.
It seems that YARN sees 8 GB memory and 4 vCores per core instance. 

SSHing into the primary node of the same instance type to further verifies that there were 2 CPUs (1 core each) at work in each core instance.

<img width="883" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/eb78025d-bd72-4102-9c4b-aa9e9ce506cc">



This confusion seems to arise from the configuration for the `yarn.nodemanager.resource.cpu-vcores` property. 

<img width="186" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/5d49060a-c976-4493-a809-b4a640b6f500">



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


When launching a shell, make sure to set executors’ idle timeout ("spark.dynamicAllocation.executorIdleTimeout") to a longer time interval (e.g., 10 minutes).
The default timeout is 60s. If we were not to configure the property to a longer time interval, idle executors would be automatically removed after 1 minute.

We don’t need to specify the "--deploy-mode" flag, because spark shells can only run in client mode



### Experiment 1

```shell
pyspark --master yarn --conf spark.dynamicAllocation.executorIdleTimeout=10m
```

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/87c1ca88-170e-40c5-944f-ac0a94dd2bd2">


 
5 containers are created and spread across the 4 core instances. one vCore is used by each container.
the instance `ip-xxxx-48-39` hosts 2 containers.


1 spark application is created

 <img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/45f2e918-1f60-4f7e-a546-cc22a9d026b2">


4 executors are created for this application
 
 <img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/0071bdd9-de48-4193-a181-fbed3a61d39c">


 
<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/135ec1b2-fb40-452b-82d5-c3f00febf746">


 <img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/20b58a73-57ca-46b9-a4c1-9dcfe13f7369">


5 containers are allocated to host the 4 executors and the application master.


| Instance ID | Instance Type | Software Entities | No. of Containers |
| ------------- |-------------| ------------- | ------------- |
| ip-xxxx-48-39  | core | executor 4 (4 cores and 2GB mem) and the application master (1 core) | 2 |
| ip-xxxx-56-172  | core | executor 1  (4 cores and 2GB mem)| 1 |
| ip-xxxx-39-175  | core |  executor 2 (4 cores and 2GB mem) | 1 |
| ip-xxxx-51-151  | core |  executor 3 (4 cores and 2GB mem)| 1 |
| ip-xxxx-31-52 | primary |  client: Pyspark shell with the driver process running inside it | 0 |

Note that the primary instance is not part of the cluster’s resource pool (because no NodeManager is running on it).

Recall that YARN sees 1 vCore per container. So, for an executor, 1 vCore seen by YARN gets mapped to 4 cores seen by Spark.
No cores are assigned to the driver. (does it imply that the driver is not running on any of the worker nodes?)

After 10 minutes, all executors are removed automatically. Only the application master and the driver stay alive.

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/cbade3c8-8025-4d4a-8328-dd461d6f93da">

 <img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/4244790b-2a2b-4dff-9a2b-c5bc8a3bf606">

### Experiment 2



