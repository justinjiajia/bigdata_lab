

## Hadoop configuration files on EMR


In Hadoop, the `CLASSPATH` is used to locate and load classes and resources, including configuration files, from various directories and JAR files. 

```shell
[hadoop@ip-xxxx ~]$ hadoop classpath
/etc/hadoop/conf:/usr/lib/hadoop/lib/*:/usr/lib/hadoop/.//*:/usr/lib/hadoop-hdfs/./:/usr/lib/hadoop-hdfs/lib/*:/usr/lib/hadoop-hdfs/.//*:/usr/lib/hadoop-mapreduce/.//*:/usr/lib/hadoop-yarn/lib/*:/usr/lib/hadoop-yarn/.//*:/usr/lib/hadoop-lzo/lib/hadoop-lzo-0.4.19.jar:/usr/lib/hadoop-lzo/lib/hadoop-lzo.jar:/usr/lib/hadoop-lzo/lib/native:/usr/share/aws/aws-java-sdk/LICENSE.txt:/usr/share/aws/aws-java-sdk/NOTICE.txt:/usr/share/aws/aws-java-sdk/README.md:/usr/share/aws/aws-java-sdk/aws-java-sdk-bundle-1.12.656.jar:/usr/share/aws/aws-java-sdk-v2/LICENSE.txt:/usr/share/aws/aws-java-sdk-v2/NOTICE.txt:/usr/share/aws/aws-java-sdk-v2/README.md:/usr/share/aws/aws-java-sdk-v2/aws-sdk-java-bundle-2.23.18.jar:/usr/share/aws/emr/emrfs/conf:/usr/share/aws/emr/emrfs/lib/animal-sniffer-annotations-1.14.jar:/usr/share/aws/emr/emrfs/lib/annotations-16.0.2.jar:/usr/share/aws/emr/emrfs/lib/aopalliance-1.0.jar:/usr/share/aws/emr/emrfs/lib/bcprov-ext-jdk15on-1.66.jar:/usr/share/aws/emr/emrfs/lib/checker-qual-2.5.2.jar:/usr/share/aws/emr/emrfs/lib/emrfs-hadoop-assembly-2.62.0.jar:/usr/share/aws/emr/emrfs/lib/error_prone_annotations-2.1.3.jar:/usr/share/aws/emr/emrfs/lib/findbugs-annotations-3.0.1.jar:/usr/share/aws/emr/emrfs/lib/j2objc-annotations-1.1.jar:/usr/share/aws/emr/emrfs/lib/javax.inject-1.jar:/usr/share/aws/emr/emrfs/lib/jmespath-java-1.12.656.jar:/usr/share/aws/emr/emrfs/lib/jsr305-3.0.2.jar:/usr/share/aws/emr/emrfs/auxlib/*:/usr/share/aws/emr/ddb/lib/emr-ddb-hadoop.jar:/usr/share/aws/emr/goodies/lib/emr-hadoop-goodies.jar:/usr/share/aws/emr/kinesis/lib/emr-kinesis-hadoop.jar:/usr/share/aws/emr/cloudwatch-sink/lib/cloudwatch-sink-2.10.0.jar:/usr/share/aws/emr/cloudwatch-sink/lib/cloudwatch-sink.jar
```

> When a resource (like a configuration file) is requested, the first occurrence found in the `CLASSPATH` is used.
E.g., *core-site.xml* is present in multiple locations within the `CLASSPATH` (e.g., `/etc/hadoop/conf`, `/usr/lib/hadoop/lib/*`), and the one appearing first is loaded.


Hadoop's configuration is driven by two types of configuration files: 

- Read-only default configuration: *core-default.xml*, *hdfs-default.xml*, *yarn-default.xml*, and *mapred-default.xml*. They can be located in different JAR files in `/usr/lib/`.

- Site-specific configuration: *core-site.xml*, *hdfs-site.xml*, *yarn-site.xml*, and *mapred-site.xml*.

Default configuration files (e.g., *core-default.xml*, *hdfs-default.xml*, etc.) are loaded first. Site-specific configuration files (e.g., *core-site.xml*, *hdfs-site.xml*, etc.) are loaded next. Any user-specified configuration files are loaded last.


<br>

### Why are they loaded in this order?


It seems that the order is indicated by Java code:

- [Configuration.java#L786](https://github.com/apache/hadoop/blob/trunk/hadoop-common-project/hadoop-common/src/main/java/org/apache/hadoop/conf/Configuration.java#L786C1-L789C41)

  ```java
    static {
      // Add default resources
      addDefaultResource("core-default.xml");
      addDefaultResource("core-site.xml");
      ...
  ```

- [HdfsConfiguration.java#L33](https://github.com/apache/hadoop/blob/trunk/hadoop-hdfs-project/hadoop-hdfs-client/src/main/java/org/apache/hadoop/hdfs/HdfsConfiguration.java#L33C1-L41C4)

  ```java
    static {
      ...
      // adds the default resources
      Configuration.addDefaultResource("hdfs-default.xml");
      Configuration.addDefaultResource("hdfs-rbf-default.xml");
      Configuration.addDefaultResource("hdfs-site.xml");
      Configuration.addDefaultResource("hdfs-rbf-site.xml");
    }
  ```

- [YarnConfiguration.java#L100](https://github.com/apache/hadoop/blob/trunk/hadoop-yarn-project/hadoop-yarn/hadoop-yarn-api/src/main/java/org/apache/hadoop/yarn/conf/YarnConfiguration.java#L100)
  ```java
    static {
      ...
      Configuration.addDefaultResource(YARN_DEFAULT_CONFIGURATION_FILE);
      Configuration.addDefaultResource(YARN_SITE_CONFIGURATION_FILE);
      Configuration.addDefaultResource(RESOURCE_TYPES_CONFIGURATION_FILE);
    }
  ```

https://github.com/apache/hadoop/blob/trunk/hadoop-tools/hadoop-extras/src/main/java/org/apache/hadoop/mapred/tools/GetGroups.java#L35


<br>

### Precedence of configuration settings

- Within a single configuration file, properties defined later can override earlier properties.

- Across multiple configuration files, properties in site-specific files (e.g., *core-site.xml*, *hdfs-site.xml*) override properties in default files (e.g., *core-default.xml*, *hdfs-default.xml*). User-specified configurations, if loaded afterward, can override both default and site-specific configurations.


<br>

## Locations of configuration files

<br>

##  Default configuration files

```shell
[hadoop@ip-xxxx ~]$ jar tf /usr/lib/hadoop/hadoop-common.jar | grep default
core-default.xml
```

```shell
[hadoop@ip-xxxx ~]$ ls /usr/lib/hadoop-mapreduce | grep mapreduce-client
hadoop-mapreduce-client-app-3.3.6-amzn-3.jar
hadoop-mapreduce-client-app.jar
hadoop-mapreduce-client-common-3.3.6-amzn-3.jar
hadoop-mapreduce-client-common.jar
hadoop-mapreduce-client-core-3.3.6-amzn-3.jar
hadoop-mapreduce-client-core.jar
hadoop-mapreduce-client-hs-3.3.6-amzn-3.jar
hadoop-mapreduce-client-hs-plugins-3.3.6-amzn-3.jar
hadoop-mapreduce-client-hs-plugins.jar
hadoop-mapreduce-client-hs.jar
hadoop-mapreduce-client-jobclient-3.3.6-amzn-3-tests.jar
hadoop-mapreduce-client-jobclient-3.3.6-amzn-3.jar
hadoop-mapreduce-client-jobclient.jar
hadoop-mapreduce-client-nativetask-3.3.6-amzn-3.jar
hadoop-mapreduce-client-nativetask.jar
hadoop-mapreduce-client-shuffle-3.3.6-amzn-3.jar
hadoop-mapreduce-client-shuffle.jar
hadoop-mapreduce-client-uploader-3.3.6-amzn-3.jar
hadoop-mapreduce-client-uploader.jar

[hadoop@ip-xxxx ~]$ jar tf /usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-core.jar | grep default
mapred-default.xml

[hadoop@ip-xxxx ~]$ jar xf /usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-core.jar mapred-default.xml

[hadoop@ip-xxxx ~]$ jar tf /usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-core.jar | grep default | xargs  cat | grep -A 5 slow
  <name>mapreduce.job.speculative.slowtaskthreshold</name>
  <value>1.0</value>
  <description>The number of standard deviations by which a task's
  ave progress-rates must be lower than the average of all running tasks'
  for the task to be considered too slow.
  </description>
</property>

<property>
  <name>mapreduce.job.ubertask.enable</name>
--
  <name>mapreduce.job.reduce.slowstart.completedmaps</name>
  <value>0.05</value>
  <description>Fraction of the number of maps in the job which should be
  complete before reduces are scheduled for the job.
  </description>
</property>
```

 Key properties set in `mapred-default.xml`:
 
 <img width="600" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/3dfc8113-e328-4ce8-8fb0-aa900f15e242">

 <img width="600" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/e7d6e56b-c4de-4ba2-96cb-6fb44da1ee14">


### Site-specific configuration files



```shell
[hadoop@ip-xxxx ~]$ ls /usr/lib/hadoop/etc/hadoop
capacity-scheduler.xml          container-log4j.properties.default  hdfs-env.sh              httpfs-site.xml           mapred-queues.xml.template  ssl-server.xml.example
capacity-scheduler.xml.default  core-site.xml                       hdfs-rbf-site.xml        log4j.properties          mapred-site.xml             taskcontroller.cfg
configuration.xsl               hadoop-env.sh                       hdfs-site.xml            log4j.properties.default  ssl-client.xml              workers
container-executor.cfg          hadoop-metrics2.properties          httpfs-env.sh            mapred-env.sh             ssl-client.xml.example      yarn-env.sh
container-log4j.properties      hadoop-policy.xml                   httpfs-signature.secret  mapred-env.sh.default     ssl-server.xml              yarn-site.xml

[hadoop@ip-xxxx ~]$ ls /etc/hadoop/conf
capacity-scheduler.xml          container-log4j.properties.default  hdfs-env.sh              httpfs-site.xml           mapred-queues.xml.template  ssl-server.xml.example
capacity-scheduler.xml.default  core-site.xml                       hdfs-rbf-site.xml        log4j.properties          mapred-site.xml             taskcontroller.cfg
configuration.xsl               hadoop-env.sh                       hdfs-site.xml            log4j.properties.default  ssl-client.xml              workers
container-executor.cfg          hadoop-metrics2.properties          httpfs-env.sh            mapred-env.sh             ssl-client.xml.example      yarn-env.sh
container-log4j.properties      hadoop-policy.xml                   httpfs-signature.secret  mapred-env.sh.default     ssl-server.xml              yarn-site.xml
```


Note: `/etc/hadoop/conf` precedes `/usr/lib/hadoop/lib/*` in `ClASSPATH`.  




Key properties set in `/etc/hadoop/conf/mapred-site.xml`:


<img width="370" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/787d2e16-7f53-45b0-b724-00bcaf3fa67c">

<img width="670" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/2b00fe35-11ce-4caa-80b8-0848254512d9">


<br>

### Effective Configurations


The detail of effective configurations can be found from the ResourceManager's Web UI:

<img width="300" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/80e385ee-84ba-4452-a67a-67f3500beff7">

Each entry shows both the effective value and where it is defined. E.g.:

```xml
<property>
<name>mapreduce.job.maps</name>
<value>16</value>
<final>false</final>
<source>mapred-site.xml</source>
</property>
...
<property>
<name>mapreduce.map.memory.mb</name>
<value>1536</value>
<final>false</final>
<source>mapred-site.xml</source>
</property>
...
<property>
<name>yarn.app.mapreduce.am.containerlauncher.threadpool-initial-size</name>
<value>10</value>
<final>false</final>
<source>mapred-default.xml</source>
</property>
...
<property>
<name>mapreduce.job.reduce.slowstart.completedmaps</name>
<value>0.05</value>
<final>false</final>
<source>mapred-default.xml</source>
</property>
...
<property>
<name>mapreduce.reduce.memory.mb</name>
<value>3072</value>
<final>false</final>
<source>mapred-site.xml</source>
</property>
```

Administrators typically define parameters as final in `core-site.xml` for values that user applications may not alter.

<br>


## What EMR Instances Include

- Compiled JAR Files:
  EMR instances come with the necessary Hadoop binaries in the form of JAR files. These JAR files include the compiled classes for Hadoop, HDFS, YARN, and other components. The JAR files are usually located in directories like /usr/lib/hadoop, /usr/lib/hadoop-hdfs, /usr/lib/hadoop-yarn, etc.

- Configuration Files:
  EMR instances also include configuration files, typically found in directories such as /etc/hadoop/conf. These files (e.g., core-site.xml, hdfs-site.xml, yarn-site.xml) are used to configure Hadoop settings.

- Libraries and Dependencies:
  Additional libraries and dependencies required by Hadoop are included as JAR files.
  EMR includes libraries for integration with other AWS services, such as Amazon S3, Amazon DynamoDB, and Amazon CloudWatch. These are also provided as compiled JAR files.

- EMR instances typically do not contain the raw Java source code of Hadoop. To find the source code, visit: https://github.com/apache/hadoop/


