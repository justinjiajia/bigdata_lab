



Note: Installing JupyterHub only won't download and install Python libraries such as NumPy. We still need to run a bootstrap action to downloand and install needed Python libraries.



<img width="1409" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/74a924d5-2c13-4fc9-b82a-7e9fe85a20b8">

No containers are allocated until we run some Python code in a notebook cell.


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


Note that the driver process now is started on a core instance. 
> A Jupyter notebook uses the Sparkmagic kernel as a client for interactively working with Spark in a remote EMR cluster through an Apache Livy server.  https://repost.aws/knowledge-center/modify-spark-configuration-emr-notebook


Later, running the configuration cell every time will launch a new application with new configurations.


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
| ip-xxxx-58-45  | core | driver (0 core; 1G mem) & executor 3 (3 cores; 912M mem) | 2 |
| ip-xxxx-48-235  | core | executors 4 & 5 (3 cores; 912M mem)| 2  |
| ip-xxxx-54-228  | core |  executors 1 & 2 (3 cores; 912M mem) | 2  |
| ip-xxxx-63-62  | core |  executors 6 & 7 (3 cores; 912M mem)| 2  |


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
