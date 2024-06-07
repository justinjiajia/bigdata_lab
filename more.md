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
 
More on this confusion: https://repost.aws/questions/QUmbShfKT4ShOy1IX8T6Exng/difference-in-vcore-and-vcpu-ec2-and-emr


When launching a shell, make sure to set executors’ idle timeout ("spark.dynamicAllocation.executorIdleTimeout") to a longer time interval (e.g., 10 minutes).
The default timeout is 60s. If we were not to configure the property to a longer time interval, idle executors would be automatically removed after 1 minute.

We don’t need to specify the "--deploy-mode" flag, because spark shells can only run in client mode


1. Experiment 1


pyspark --master yarn --conf spark.dynamicAllocation.executorIdleTimeout=10m


 

5 containers are created and spread across the 4 core instances.
1 vCore is used by each container.
Memory used on ip-xxxx-48-39 is larger than that used in any of the other core instances, because that instance hosts 2 containers.


1 spark application is created

 

4 executors are created for this application
 

 

5 containers are allocated to host the 4 executors and the application master.

ip-xxxx-48-39: 1 container for executor 4 and the application master
ip-xxxx-56-172: 1 container for executor 1
ip-xxxx-39-175: 1 container for executor 2
ip-xxxx-51-151: 1 container for executor 3
ip-xxxx-31-52: this is the master node of the cluster, which is not part of the cluster’s resource pool (because no NodeManager is running on it).


Each executor owns 4 cores and 2GB memory, while the application master has 1 core.
Recall that YARN sees 1 vCore per container. So, for an executor, 1 vCore seen by YARN gets mapped to 4 cores seen by Spark.
No cores are assigned to the driver, implying the driver is not running on any worker node, but in the master node.


 


