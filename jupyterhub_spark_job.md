



Note: Installing JupyterHub only won't download and install Python libraries such as NumPy. We still need to run a bootstrap action to downloand and install needed Python libraries.



<img width="1409" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/74a924d5-2c13-4fc9-b82a-7e9fe85a20b8">


How to modify a spark application with custom configurations: https://repost.aws/knowledge-center/modify-spark-configuration-emr-notebook

In a Jupyter notebook cell, run the `%%configure` command with desired configurations:

```python
%%configure -f
{"conf": {
    "spark.dynamicAllocation.executorIdleTimeout": "10m"}  
}
```
For example, we may want to increase executors' idle timeout. Otherwise, executors will be automatically removed after 1 minute.

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/fde09276-dc92-45cf-9cad-ac957890cb52">

Running any code will start a new application on YARN with the custom configurations.




<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/4bbb92ac-67d7-4912-90f8-597c054786f3">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/d066c0d1-a265-4228-8129-acf97887b71d">


Later, running the configuration cell every time will launch a new application with new configurations.

```python
%%configure -f
{"conf": {
    "spark.executor.cores": "2", 
    "spark.dynamicAllocation.executorIdleTimeout": "5m"} 
}
```


<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/e05f75f2-880c-4199-98e9-4952cb57ba02">


<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/eb020ee4-7978-48b7-9d78-0d85ec297894">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/da36ada8-af96-4d27-bb69-d41530c33f20">


A Jupyter notebook uses the Sparkmagic kernel as a client for interactively working with Spark in a remote EMR cluster through an Apache Livy server. 