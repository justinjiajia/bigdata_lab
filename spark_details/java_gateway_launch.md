
`Popen(command, **popen_kwargs)` 


`env` contains the environment variables exported from the starter scripts

```python
{'HOSTNAME': 'ip-172-31-62-159.ec2.internal',
 'SPARK_LOG_DIR': '/var/log/spark',
 'JAVA17_HOME': '/usr/lib/jvm/jre-17',
 'PYTHONHASHSEED': '0',
 'PYSPARK_DRIVER_PYTHON': '/usr/bin/python3', 'JAVA_HOME': '/usr/lib/jvm/jre-17', 'SPARK_WORKER_PORT': '7078',
 'SPARK_MASTER_WEBUI_PORT': '8080',
 'AWS_DEFAULT_REGION': 'us-east-1',
 'HUDI_CONF_DIR': '/etc/hudi/conf',
 'SPARK_DAEMON_JAVA_OPTS': ' -XX:+ExitOnOutOfMemoryError -DAWS_ACCOUNT_ID=154048744197 -DEMR_CLUSTER_ID=j-VC0KIOO5V5LS -DEMR_RELEASE_LABEL=emr-7.1.0',
 'SPARK_SUBMIT_OPTS': ' -DAWS_ACCOUNT_ID=154048744197 -DEMR_CLUSTER_ID=j-VC0KIOO5V5LS -DEMR_RELEASE_LABEL=emr-7.1.0',
 'EMR_CLUSTER_ID': 'j-VC0KIOO5V5LS', 'SPARK_WORKER_WEBUI_PORT': '8081',
 'SPARK_MASTER_IP': 'ip-172-31-62-159.ec2.internal',
 'PYTHONSTARTUP': '/usr/lib/spark/python/pyspark/shell.py',
 'SPARK_MASTER_PORT': '7077', 'HIVE_CONF_DIR': '/etc/hive/conf', 'PYSPARK_PYTHON': '/usr/bin/python3', 'PYTHONPATH': '/usr/lib/spark/python/lib/py4j-0.10.9.7-src.zip:/usr/lib/spark/python/:',
 'HADOOP_CONF_DIR': '/etc/hadoop/conf',
 'HADOOP_HOME': '/usr/lib/hadoop',  'EMR_RELEASE_LABEL': 'emr-7.1.0', 'SPARK_PUBLIC_DNS': 'ip-172-31-62-159.ec2.internal',
 'HIVE_SERVER2_THRIFT_PORT': '10001', 'OLD_PYTHONSTARTUP': '',
 'HIVE_SERVER2_THRIFT_BIND_HOST': '0.0.0.0',
 'AWS_ACCOUNT_ID': '154048744197', 'SPARK_HOME': '/usr/lib/spark', 'STANDALONE_SPARK_MASTER_HOST': 'ip-172-31-62-159.ec2.internal',
 'SPARK_CONF_DIR': '/usr/lib/spark/conf', 'AWS_SPARK_REDSHIFT_CONNECTOR_SERVICE_NAME': 'EMR', 
 '_SPARK_CMD_USAGE': 'Usage: ./bin/pyspark [options]', 'SPARK_WORKER_DIR': '/var/run/spark/work',
 'SPARK_ENV_LOADED': '1',
 'SPARK_SCALA_VERSION': '2.12'}
```

also 2 entries created during the execution of *Main.java*:

```python
{'PYSPARK_SUBMIT_ARGS': '"--master" "yarn" "--conf" "spark.driver.memory=2g" "--name" "PySparkShell" "--executor-memory" "2g" "pyspark-shell"',
'LD_LIBRARY_PATH': '/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server'}
```

and one added in *java_gateway.py*:

```python
{'_PYSPARK_DRIVER_CONN_INFO_PATH': '/tmp/tmpzcoujk06/tmpkbdpvov9'}
```


`command` equals

```python
 ['/usr/lib/spark/./bin/spark-submit', '--master', 'yarn', '--conf', 'spark.driver.memory=2g', '--name', 'PySparkShell', '--executor-memory', '2g', 'pyspark-shell']` after the assignment.
```


<br> 

### [*spark-submit*](https://github.com/apache/spark/blob/master/bin/spark-submit) in */usr/lib/spark/bin*

<br>

 
```shell
#!/usr/bin/env bash
...
exec "${SPARK_HOME}"/bin/spark-class org.apache.spark.deploy.SparkSubmit "$@"
```

- `org.apache.spark.deploy.SparkSubmit` is defined in [scala/org/apache/spark/deploy/SparkSubmit.scala](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/deploy/SparkSubmit.scala)

- `"$@"` expands to `--master yarn --conf spark.driver.memory=2g --name PySparkShell --executor-memory 2g pyspark-shell` 





<br>
  
### [*spark-class*](https://github.com/apache/spark/blob/master/bin/spark-class) in */usr/lib/spark/bin* 


<br>


```shell
#!/usr/bin/env bash
...

build_command() {
  "$RUNNER" -Xmx128m $SPARK_LAUNCHER_OPTS -cp "$LAUNCH_CLASSPATH" org.apache.spark.launcher.Main "$@"
  printf "%d\0" $?
}

...

while IFS= read -d "$DELIM" -r _ARG; do
  ...
done < <(build_command "$@")

...
exec "${CMD[@]}"
```


- Define a function named `build_command()`
  
  -  `"$RUNNER" -Xmx128m $SPARK_LAUNCHER_OPTS -cp "$LAUNCH_CLASSPATH" org.apache.spark.launcher.Main "$@"`
          

   
     - Effectively, this executes `/usr/lib/jvm/jre-17/bin/java -Xmx128m -cp <all files under /usr/lib/spark/jars> org.apache.spark.launcher.Main org.apache.spark.deploy.SparkSubmit --master yarn --conf spark.driver.memory=2g --name PySparkShell --executor-memory 2g pyspark-shell`
  
 


-  Verified the value of `"${CMD[@]}"` with the following code:
  
 

- `exec "${CMD[@]}"` replaces the current shell with the command stored in `CMD`. It executes *python/pyspark/shell.py* and starts a Python intepreter.

    - `echo "${CMD[@]}"` prints:
      ```shell
      env PYSPARK_SUBMIT_ARGS="--master" "yarn" "--conf" "spark.driver.memory=2g" "--name" "PySparkShell" "--executor-memory" "2g" "pyspark-shell" LD_LIBRARY_PATH=/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server /usr/bin/python3
      ```


