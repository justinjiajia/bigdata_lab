
# Types of Spark properties

Spark properties mainly can be divided into two kinds: 

- One is related to deploy, like `spark.driver.memory`, `spark.executor.instances`, this kind of properties may not be affected when setting programmatically through `SparkConf` in runtime, or the behavior is depending on which cluster manager and deploy mode you choose, so it would be suggested to set through configuration file or spark-submit command line options;

- Another is mainly related to Spark runtime control, like `spark.task.maxFailures`, this kind of properties can be set in either way.

<br>


# Configurations

- Spark properties can be set directly on a `SparkConf` passed to your `SparkContext`. E.g.:
  ```scala
  val conf = new SparkConf()
             .setMaster("local[2]")
             .setAppName("CountingSheep")
  val sc = new SparkContext(conf)
  ```

- Spark properties can also be set via command line options. 

  
- `spark-submit` or a shell script will also read configuration options from `conf/spark-defaults.conf`

- If a property is not set in the above user-provided sources, it falls back to the default values defined within Spark's codebase. E.g.,

   - `spark.memory.fraction` defined in [*org/apache/spark/internal/config/package.scala*](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/internal/config/package.scala)
     
       - `package object config {...}`: In Scala, an object is a singleton instance of a class. When the Scala compiler compiles an object, it generates a corresponding class file with a `$` appended to its name.
    
       - `import org.apache.spark.internal.config.package$`
     
   - `spark.yarn.am.memory` defined in [*org/apache/spark/deploy/yarn/config.scala*](https://github.com/apache/spark/blob/master/resource-managers/yarn/src/main/scala/org/apache/spark/deploy/yarn/config.scala) 
 
       - `package object config extends Logging {...}`
    
       - `import org.apache.spark.deploy.yarn.config._`: this is a wildcard import. All members (fields, methods, objects, classes, traits, etc.) defined in the `config` package object will be imported into the current scope.
    
       - sd
         ```scala
         private lazy val IS_HADOOP_PROVIDED: Boolean = {
           val configPath = "org/apache/spark/deploy/yarn/config.properties"
           val propertyKey = "spark.yarn.isHadoopProvided"
           ...

         }
         ```
         
         ```shell
         $ jar tf /usr/lib/spark/jars/spark-yar* | grep deploy/yarn/config
         org/apache/spark/deploy/yarn/config.properties
         org/apache/spark/deploy/yarn/config/
         org/apache/spark/deploy/yarn/config/package$.class
         org/apache/spark/deploy/yarn/config/package.class
         ```


<br>

## Precedence of configuration settings

Any values specified as flags or in the properties file will be passed on to the application and merged with those specified through `SparkConf`.

Properties set directly on the `SparkConf` take highest precedence, then flags passed to [`spark-submit`]() or [`spark-shell`](), then options in the *spark-defaults.conf* file.

More details can be found on [this page](https://spark.apache.org/docs/latest/configuration.htm).

<br>

# Order of execution

when run the `spark-submit` script,

- first execute */usr/lib/load-spark-env.sh* to create a set of environment variables

- then run class Main in `/usr/lib/spark/jars/spark-launcher*.jar` to parse the command line flags and load properties set in `/usr/lib/spark/confspark-defaults.conf`

More defails: https://github.com/justinjiajia/bigdata_lab/blob/main/spark_submit_code.md

<br>

# Locations of configuration files

<br>

## `spark-defaults.conf`

It's not clear why there are two identical copies of configuration files:



```shell
[hadoop@ip-xxxx ~]$ ls /etc/spark/conf
emrfs-site.xml              hive-site.xml      log4j2.properties.template  metrics.properties.template  spark-defaults.conf.template  spark-env.sh.template
fairscheduler.xml.template  log4j2.properties  metrics.properties          spark-defaults.conf          spark-env.sh                  workers.template

[hadoop@ip-xxxx ~]$  ls /usr/lib/spark/conf
emrfs-site.xml              hive-site.xml      log4j2.properties.template  metrics.properties.template  spark-defaults.conf.template  spark-env.sh.template
fairscheduler.xml.template  log4j2.properties  metrics.properties          spark-defaults.conf          spark-env.sh                  workers.template
```

The contents of the same-name files are the same. This can be verified by using `vim -d` or `vimdiff`. E.g.:

```shell
vim -d /etc/spark/conf/spark-defaults.conf /usr/lib/spark/conf/spark-defaults.conf
```



The content of *spark-defaults.conf* is as follows:

```
spark.master                     yarn
spark.driver.extraClassPath      /usr/lib/hadoop-lzo/lib/*:/usr/lib/hadoop/hadoop-aws.jar:/usr/share/aws/aws-java-sdk/*:/usr/share/aws/aws-java-sdk-v2/*:/usr/share/aws/emr/goodies/lib/emr-spark-goodies.jar:/usr/share/aws/emr/security/conf:/usr/share/aws/emr/security/lib/*:/usr/share/aws/redshift/jdbc/*:/usr/share/aws/redshift/spark-redshift/lib/*:/usr/share/aws/kinesis/spark-sql-kinesis/lib/*:/usr/share/aws/hmclient/lib/aws-glue-datacatalog-spark-client.jar:/usr/share/java/Hive-JSON-Serde/hive-openx-serde.jar:/usr/share/aws/emr/s3select/lib/emr-s3-select-spark-connector.jar:/docker/usr/lib/hadoop-lzo/lib/*:/docker/usr/lib/hadoop/hadoop-aws.jar:/docker/usr/share/aws/aws-java-sdk/*:/docker/usr/share/aws/aws-java-sdk-v2/*:/docker/usr/share/aws/emr/goodies/lib/emr-spark-goodies.jar:/docker/usr/share/aws/emr/security/conf:/docker/usr/share/aws/emr/security/lib/*:/docker/usr/share/aws/redshift/jdbc/*:/docker/usr/share/aws/redshift/spark-redshift/lib/*:/docker/usr/share/aws/kinesis/spark-sql-kinesis/lib/*:/docker/usr/share/aws/hmclient/lib/aws-glue-datacatalog-spark-client.jar:/docker/usr/share/java/Hive-JSON-Serde/hive-openx-serde.jar:/docker/usr/share/aws/emr/s3select/lib/emr-s3-select-spark-connector.jar
spark.driver.extraLibraryPath    /usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server
spark.executor.extraClassPath    /usr/lib/hadoop-lzo/lib/*:/usr/lib/hadoop/hadoop-aws.jar:/usr/share/aws/aws-java-sdk/*:/usr/share/aws/aws-java-sdk-v2/*:/usr/share/aws/emr/goodies/lib/emr-spark-goodies.jar:/usr/share/aws/emr/security/conf:/usr/share/aws/emr/security/lib/*:/usr/share/aws/redshift/jdbc/*:/usr/share/aws/redshift/spark-redshift/lib/*:/usr/share/aws/kinesis/spark-sql-kinesis/lib/*:/usr/share/aws/hmclient/lib/aws-glue-datacatalog-spark-client.jar:/usr/share/java/Hive-JSON-Serde/hive-openx-serde.jar:/usr/share/aws/emr/s3select/lib/emr-s3-select-spark-connector.jar:/docker/usr/lib/hadoop-lzo/lib/*:/docker/usr/lib/hadoop/hadoop-aws.jar:/docker/usr/share/aws/aws-java-sdk/*:/docker/usr/share/aws/aws-java-sdk-v2/*:/docker/usr/share/aws/emr/goodies/lib/emr-spark-goodies.jar:/docker/usr/share/aws/emr/security/conf:/docker/usr/share/aws/emr/security/lib/*:/docker/usr/share/aws/redshift/jdbc/*:/docker/usr/share/aws/redshift/spark-redshift/lib/*:/docker/usr/share/aws/kinesis/spark-sql-kinesis/lib/*:/docker/usr/share/aws/hmclient/lib/aws-glue-datacatalog-spark-client.jar:/docker/usr/share/java/Hive-JSON-Serde/hive-openx-serde.jar:/docker/usr/share/aws/emr/s3select/lib/emr-s3-select-spark-connector.jar
spark.executor.extraLibraryPath  /usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server
spark.executorEnv.JAVA_HOME	  /usr/lib/jvm/jre-17
spark.yarn.appMasterEnv.JAVA_HOME /usr/lib/jvm/jre-17
spark.eventLog.enabled           true
spark.eventLog.dir               hdfs:///var/log/spark/apps
spark.history.fs.logDirectory    hdfs:///var/log/spark/apps
spark.sql.warehouse.dir          hdfs:///user/spark/warehouse
spark.yarn.historyServer.address ip-172-31-59-96.ec2.internal:18080
spark.history.ui.port            18080
spark.shuffle.service.enabled    true
spark.executorEnv.AWS_SPARK_REDSHIFT_CONNECTOR_SERVICE_NAME    EMR
spark.yarn.dist.files            /etc/hudi/conf/hudi-defaults.conf
spark.sql.hive.metastore.sharedPrefixes com.amazonaws.services.dynamodbv2
spark.driver.defaultJavaOptions  -XX:OnOutOfMemoryError='kill -9 %p'
spark.dynamicAllocation.enabled  true
spark.blacklist.decommissioning.enabled true
spark.blacklist.decommissioning.timeout 1h
spark.resourceManager.cleanupExpiredHost true
spark.stage.attempt.ignoreOnDecommissionFetchFailure true
spark.decommissioning.timeout.threshold 20
spark.executor.defaultJavaOptions -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:OnOutOfMemoryError='kill -9 %p'
spark.hadoop.yarn.timeline-service.enabled false
spark.yarn.appMasterEnv.SPARK_PUBLIC_DNS $(hostname -f)
spark.files.fetchFailure.unRegisterOutputOnHost true
spark.hadoop.mapreduce.fileoutputcommitter.algorithm.version.emr_internal_use_only.EmrFileSystem 2
spark.hadoop.mapreduce.fileoutputcommitter.cleanup-failures.ignored.emr_internal_use_only.EmrFileSystem true
spark.hadoop.mapreduce.output.fs.optimized.committer.enabled true
spark.hadoop.fs.s3.getObject.initialSocketTimeoutMilliseconds 2000
spark.sql.parquet.output.committer.class com.amazon.emr.committer.EmrOptimizedSparkSqlParquetOutputCommitter
spark.sql.parquet.fs.optimized.committer.optimization-enabled true
spark.sql.emr.internal.extensions com.amazonaws.emr.spark.EmrSparkSessionExtensions
spark.executor.memory            4269M
spark.emr.default.executor.memory 4269M
spark.driver.memory              2048M
spark.executor.cores             4
spark.emr.default.executor.cores 4
```

The content of *spark-env.sh* is as follows:

```shell
JAVA17_HOME=/usr/lib/jvm/jre-17
export JAVA_HOME=$JAVA17_HOME

export SPARK_HOME=${SPARK_HOME:-/usr/lib/spark}
export SPARK_LOG_DIR=${SPARK_LOG_DIR:-/var/log/spark}
export HADOOP_HOME=${HADOOP_HOME:-/usr/lib/hadoop}
export HADOOP_CONF_DIR=${HADOOP_CONF_DIR:-/etc/hadoop/conf}
export HIVE_CONF_DIR=${HIVE_CONF_DIR:-/etc/hive/conf}
export HUDI_CONF_DIR=${HUDI_CONF_DIR:-/etc/hudi/conf}

export STANDALONE_SPARK_MASTER_HOST=ip-172-31-59-96.ec2.internal
export SPARK_MASTER_PORT=7077
export SPARK_MASTER_IP=$STANDALONE_SPARK_MASTER_HOST
export SPARK_MASTER_WEBUI_PORT=8080

export SPARK_WORKER_DIR=${SPARK_WORKER_DIR:-/var/run/spark/work}
export SPARK_WORKER_PORT=7078
export SPARK_WORKER_WEBUI_PORT=8081

export HIVE_SERVER2_THRIFT_BIND_HOST=0.0.0.0
export HIVE_SERVER2_THRIFT_PORT=10001

export AWS_SPARK_REDSHIFT_CONNECTOR_SERVICE_NAME=EMR

export SPARK_DAEMON_JAVA_OPTS="$SPARK_DAEMON_JAVA_OPTS -XX:+ExitOnOutOfMemoryError"
export SPARK_PUBLIC_DNS=ip-172-31-59-96.ec2.internal
export PYSPARK_PYTHON=/usr/bin/python3
```

Note: `export SPARK_HOME=${SPARK_HOME:-/usr/lib/spark}`


<br>


# EMR  configuration API


When creating or modifying an EMR cluster, we can use the Configurations parameter to set Spark configurations.

```json
[
  {
    "Classification": "spark-defaults",
    "Properties": {
      "spark.emr.default.executor.cores": "2",
      "spark.executor.cores": "2"
    }
  },
  {
    "Classification": "spark-env",
    "Properties": {},
    "Configurations": [
      {
        "Classification": "export",
        "Properties": {
          "SPARK_DRIVER_MEMORY": "2g"
        }
      }
    ]
  }
]
```

- Write to both /etc/spark/conf/spark-defaults.conf and /usr/lib/spark/conf/spark-defaults.conf

  The original entries are removed. 2 new entries are added to the end of the file
  ```
  spark.emr.default.executor.cores   2
  spark.executor.cores        2
  ```

appears at the end of the file

- Write to both /etc/spark/conf/spark-env.sh and /usr/lib/spark/conf/spark-env.sh

  A new entry is added to the end of the file
  ```
  export SPARK_DRIVER_MEMORY=2g
  ```


# Effective Configurations


```shell
[hadoop@ip-xxxx ~]$ echo 'sc.getConf.get("spark.submit.deployMode")' | spark-shell --master local
```


Check the effective configurations inside a launched shell:

```shell
[hadoop@ip-xxxx ~]$ spark-shell --master local
```

```scala
scala> sc.getConf.getAll
res0: Array[(String, String)] = Array((spark.eventLog.enabled,true), (spark.driver.extraLibraryPath,/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server), (spark.driver.extraClassPath,/usr/lib/hadoop-lzo/lib/*:/usr/lib/hadoop/hadoop-aws.jar:/usr/share/aws/aws-java-sdk/*:/usr/share/aws/aws-java-sdk-v2/*:/usr/share/aws/emr/goodies/lib/emr-spark-goodies.jar:/usr/share/aws/emr/security/conf:/usr/share/aws/emr/security/lib/*:/usr/share/aws/redshift/jdbc/*:/usr/share/aws/redshift/spark-redshift/lib/*:/usr/share/aws/kinesis/spark-sql-kinesis/lib/*:/usr/share/aws/hmclient/lib/aw...
```

```python
>>> spark.version
'3.5.0-amzn-1
>>> sc.getConf().getAll()
[('spark.eventLog.enabled', 'true'), ('spark.driver.extraLibraryPath', '/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server'), ('spark.driver.extraClassPath', '/usr/lib/hadoop-lzo/lib/*:/usr/lib/hadoop/hadoop-aws.jar:/usr/share/aws/aws-java-sdk/*:/usr/share/aws/aws-java-sdk-v2/*:/usr/share/aws/emr/goodies/lib/emr-spark-goodies.jar:/usr/share/aws/emr/security/conf:/usr/share/aws/emr/security/lib/*:/usr/share/aws/redshift/jdbc/*:/usr/share/aws/redshift/spark-redshift/lib/*:/usr/share/aws/kinesis/spark-sql-kinesis/lib/*:/usr/share/aws/hmclient/lib/aws-glue-datacatalog-spark-client.jar:/usr/share/java/Hive-JSON-Serde/hive-openx-serde.jar:/usr/share/aws/emr/s3select/lib/emr-s3-select-spark-connector.jar:/docker/usr/lib/hadoop-lzo/lib/*:/docker/usr/lib/hadoop/hadoop-aws.jar:/docker/usr/share/aws/aws-java-sdk/*:/docker/usr/share/aws/aws-java-sdk-v2/*:/docker/usr/share/aws/emr/goodies/lib/emr-spark-goodies.jar:/docker/usr/share/aws/emr/security/conf:/docker/usr/share/aws/emr/security/lib/*:/docker/usr/share/aws/redshift/jdbc/*:/docker/usr/share/aws/redshift/spark-redshift/lib/*:/docker/usr/share/aws/kinesis/spark-sql-kinesis/lib/*:/docker/usr/share/aws/hmclient/lib/aws-glue-datacatalog-spark-client.jar:/docker/usr/share/java/Hive-JSON-Serde/hive-openx-serde.jar:/docker/usr/share/aws/emr/s3select/lib/emr-s3-select-spark-connector.jar'), ('spark.sql.parquet.output.committer.class', 'com.amazon.emr.committer.EmrOptimizedSparkSqlParquetOutputCommitter'), ('spark.blacklist.decommissioning.timeout', '1h'), ('spark.yarn.appMasterEnv.SPARK_PUBLIC_DNS', '$(hostname -f)'), ('spark.app.submitTime', '1718257850660'), ('spark.sql.emr.internal.extensions', 'com.amazonaws.emr.spark.EmrSparkSessionExtensions'), ('spark.app.startTime', '1718257851633'), ('spark.eventLog.dir', 'hdfs:///var/log/spark/apps'), ('spark.sql.warehouse.dir', 'hdfs:///user/spark/warehouse'), ('spark.history.fs.logDirectory', 'hdfs:///var/log/spark/apps'), ('spark.hadoop.yarn.timeline-service.enabled', 'false'), ('spark.executor.extraLibraryPath', '/usr/lib/hadoop/lib/native:/usr/lib/hadoop-lzo/lib/native:/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server:/docker/usr/lib/hadoop/lib/native:/docker/usr/lib/hadoop-lzo/lib/native:/docker/usr/lib/jvm/java-17-amazon-corretto.x86_64/lib/server'), ('spark.executor.id', 'driver'), ('spark.executor.extraJavaOptions', "-Djava.net.preferIPv6Addresses=false -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:OnOutOfMemoryError='kill -9 %p' -XX:+IgnoreUnrecognizedVMOptions --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.lang.invoke=ALL-UNNAMED --add-opens=java.base/java.lang.reflect=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.base/java.net=ALL-UNNAMED --add-opens=java.base/java.nio=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/java.util.concurrent=ALL-UNNAMED --add-opens=java.base/java.util.concurrent.atomic=ALL-UNNAMED --add-opens=java.base/sun.nio.ch=ALL-UNNAMED --add-opens=java.base/sun.nio.cs=ALL-UNNAMED --add-opens=java.base/sun.security.action=ALL-UNNAMED --add-opens=java.base/sun.util.calendar=ALL-UNNAMED --add-opens=java.security.jgss/sun.security.krb5=ALL-UNNAMED -Djdk.reflect.useDirectMethodHandle=false -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:OnOutOfMemoryError='kill -9 %p'"), ('spark.app.name', 'PySparkShell'), ('spark.driver.memory', '2048M'), ('spark.hadoop.mapreduce.output.fs.optimized.committer.enabled', 'true'), ('spark.decommissioning.timeout.threshold', '20'), ('spark.sql.catalogImplementation', 'hive'), ('spark.stage.attempt.ignoreOnDecommissionFetchFailure', 'true'), ('spark.hadoop.fs.s3.getObject.initialSocketTimeoutMilliseconds', '2000'), ('spark.hadoop.fs.s3a.committer.magic.enabled', 'true'), ('spark.master', 'local'), ('spark.hadoop.mapreduce.fileoutputcommitter.algorithm.version.emr_internal_use_only.EmrFileSystem', '2'), ('spark.driver.port', '46275'), ('spark.yarn.dist.files', 'file:/etc/hudi/conf.dist/hudi-defaults.conf'), ('spark.executor.cores', '4'), ('spark.hadoop.fs.s3a.committer.name', 'magicv2'), ('spark.yarn.historyServer.address', 'ip-172-31-59-96.ec2.internal:18080'), ('spark.sql.hive.metastore.sharedPrefixes', 'com.amazonaws.services.dynamodbv2'), ('spark.serializer.objectStreamReset', '100'), ('spark.submit.deployMode', 'client'), ('spark.sql.parquet.fs.optimized.committer.optimization-enabled', 'true'), ('spark.hadoop.mapreduce.fileoutputcommitter.cleanup-failures.ignored.emr_internal_use_only.EmrFileSystem', 'true'), ('spark.executorEnv.AWS_SPARK_REDSHIFT_CONNECTOR_SERVICE_NAME', 'EMR'), ('spark.executor.extraClassPath', '/usr/lib/hadoop-lzo/lib/*:/usr/lib/hadoop/hadoop-aws.jar:/usr/share/aws/aws-java-sdk/*:/usr/share/aws/aws-java-sdk-v2/*:/usr/share/aws/emr/goodies/lib/emr-spark-goodies.jar:/usr/share/aws/emr/security/conf:/usr/share/aws/emr/security/lib/*:/usr/share/aws/redshift/jdbc/*:/usr/share/aws/redshift/spark-redshift/lib/*:/usr/share/aws/kinesis/spark-sql-kinesis/lib/*:/usr/share/aws/hmclient/lib/aws-glue-datacatalog-spark-client.jar:/usr/share/java/Hive-JSON-Serde/hive-openx-serde.jar:/usr/share/aws/emr/s3select/lib/emr-s3-select-spark-connector.jar:/docker/usr/lib/hadoop-lzo/lib/*:/docker/usr/lib/hadoop/hadoop-aws.jar:/docker/usr/share/aws/aws-java-sdk/*:/docker/usr/share/aws/aws-java-sdk-v2/*:/docker/usr/share/aws/emr/goodies/lib/emr-spark-goodies.jar:/docker/usr/share/aws/emr/security/conf:/docker/usr/share/aws/emr/security/lib/*:/docker/usr/share/aws/redshift/jdbc/*:/docker/usr/share/aws/redshift/spark-redshift/lib/*:/docker/usr/share/aws/kinesis/spark-sql-kinesis/lib/*:/docker/usr/share/aws/hmclient/lib/aws-glue-datacatalog-spark-client.jar:/docker/usr/share/java/Hive-JSON-Serde/hive-openx-serde.jar:/docker/usr/share/aws/emr/s3select/lib/emr-s3-select-spark-connector.jar'), ('spark.history.ui.port', '18080'), ('spark.executor.memory', '4269M'), ('spark.shuffle.service.enabled', 'true'), ('spark.driver.defaultJavaOptions', "-XX:OnOutOfMemoryError='kill -9 %p'"), ('spark.resourceManager.cleanupExpiredHost', 'true'), ('spark.executor.defaultJavaOptions', "-verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:OnOutOfMemoryError='kill -9 %p'"), ('spark.yarn.appMasterEnv.JAVA_HOME', '/usr/lib/jvm/jre-17'), ('spark.files.fetchFailure.unRegisterOutputOnHost', 'true'), ('spark.emr.default.executor.memory', '4269M'), ('spark.app.id', 'local-1718257852978'), ('spark.rdd.compress', 'True'), ('spark.driver.host', 'ip-172-31-59-96.ec2.internal'), ('spark.submit.pyFiles', ''), ('spark.dynamicAllocation.enabled', 'true'), ('spark.driver.extraJavaOptions', "-Djava.net.preferIPv6Addresses=false -XX:OnOutOfMemoryError='kill -9 %p' -XX:+IgnoreUnrecognizedVMOptions --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.lang.invoke=ALL-UNNAMED --add-opens=java.base/java.lang.reflect=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.base/java.net=ALL-UNNAMED --add-opens=java.base/java.nio=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/java.util.concurrent=ALL-UNNAMED --add-opens=java.base/java.util.concurrent.atomic=ALL-UNNAMED --add-opens=java.base/sun.nio.ch=ALL-UNNAMED --add-opens=java.base/sun.nio.cs=ALL-UNNAMED --add-opens=java.base/sun.security.action=ALL-UNNAMED --add-opens=java.base/sun.util.calendar=ALL-UNNAMED --add-opens=java.security.jgss/sun.security.krb5=ALL-UNNAMED -Djdk.reflect.useDirectMethodHandle=false -XX:OnOutOfMemoryError='kill -9 %p'"), ('spark.executorEnv.JAVA_HOME', '/usr/lib/jvm/jre-17'), ('spark.ui.showConsoleProgress', 'true'), ('spark.emr.default.executor.cores', '4'), ('spark.blacklist.decommissioning.enabled', 'true')]
```



Check the effective configurations via the application web UI (http://<public_DNS_primary>:18080/history/<app_id>/environment/)

- <app_id> is the information displayed in the "App ID" column
  
<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/fbe7b8f7-7b5b-4727-b51b-ffa8f271e247">

They are listed in the “Environment” tab. 


<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/7c28fb8f-947c-4de1-bf9c-025f6d2473b3">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/713f4186-560f-4073-9d69-c9631c3c65ac">


<br>

# Locations of Spark scripts

```
[hadoop@ip-xxxx ~]$ ls /usr/lib/spark/bin
beeline               find-spark-home  load-spark-env.sh  run-example  spark-connect-shell  spark-sql     sparkR
docker-image-tool.sh  load-emr-env.sh  pyspark
```


<br>

# Locations of Compiled JAR files

```shell
[hadoop@ip-xxxx ~]$ ls /usr/lib/spark/jars
[hadoop@ip-xxxx ~]$ jar tf /usr/lib/spark/jars/spark-core*.jar 
```

<br>

# Spark source code of different versions

The source code of a particular Spark version can be found at https://archive.apache.org/dist/spark/ 

For example, the source code for Spark 5.1.0 is in spark-3.5.0.tgz at https://archive.apache.org/dist/spark/spark-3.5.0/.
