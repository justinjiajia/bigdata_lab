
# EMR settings

- EMR release: 7.1.0 

- Application: Hadoop
  
- Primary instance: type: `m4.large`, quantity: 1

- Core instance: type: `m4.large`, quantity: 4
  

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
                "dfs.blocksize": "16M"
            }
        }
    ]
    ```
    


```shell
wget -O nytimes.txt https://raw.githubusercontent.com/justinjiajia/datafiles/main/nytimes_news_articles.txt
hadoop fs -mkdir /input
hadoop fs -put  nytimes.txt /input
```

<img width="1178" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/e62eaa0f-0755-40d5-864a-c34da7df2951">

The file is divided into 3 blocks, which are spread across 4 core instances.


| Instance | block | 
|------|--------|
| ip-xxxx-52-142.xxxx| block 1 |
| ip-xxxx-52-162.xxxx| block 0 |
| ip-xxxx-52-162.xxxx| block 2 |
| ip-xxxx-57-35.xxxx  | block 0 |
| ip-xxxx-57-35.xxxx  | block 1 |
| ip-xxxx-63-90.xxxx  | block 2 |

primary node: ip-xxxx-53-255.xxxx

# Run a word count application

Refer to [this manual](../lab_4_hadoop_streaming.md) for the code used by mappers and reducers


```shell
[hadoop@ip-xxxx ~]$ mapred streaming -D mapreduce.job.reduces=2 -files mapper.py,reducer.py -input /input -output /output_1 -mapper mapper.py -reducer reducer.py
packageJobJar: [] [/usr/lib/hadoop/hadoop-streaming-3.3.6-amzn-3.jar] /tmp/streamjob12540940317434631248.jar tmpDir=null
2024-06-09 18:16:22,825 INFO client.DefaultNoHARMFailoverProxyProvider: Connecting to ResourceManager at ip-172-31-53-255.ec2.internal/172.31.53.255:8032
2024-06-09 18:16:23,030 INFO client.AHSProxy: Connecting to Application History server at ip-172-31-53-255.ec2.internal/172.31.53.255:10200
2024-06-09 18:16:23,096 INFO client.DefaultNoHARMFailoverProxyProvider: Connecting to ResourceManager at ip-172-31-53-255.ec2.internal/172.31.53.255:8032
2024-06-09 18:16:23,097 INFO client.AHSProxy: Connecting to Application History server at ip-172-31-53-255.ec2.internal/172.31.53.255:10200
2024-06-09 18:16:23,451 INFO mapreduce.JobResourceUploader: Disabling Erasure Coding for path: /tmp/hadoop-yarn/staging/hadoop/.staging/job_1717955085543_0001
2024-06-09 18:16:23,921 INFO lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:16:23,954 INFO lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:16:24,024 INFO mapred.FileInputFormat: Total input files to process : 1
2024-06-09 18:16:24,029 INFO net.NetworkTopology: Adding a new node: /default-rack/172.31.57.35:9866
2024-06-09 18:16:24,030 INFO net.NetworkTopology: Adding a new node: /default-rack/172.31.52.162:9866
2024-06-09 18:16:24,030 INFO net.NetworkTopology: Adding a new node: /default-rack/172.31.52.142:9866
2024-06-09 18:16:24,031 INFO net.NetworkTopology: Adding a new node: /default-rack/172.31.63.90:9866
2024-06-09 18:16:24,125 INFO mapreduce.JobSubmitter: number of splits:16
2024-06-09 18:16:24,513 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1717955085543_0001
2024-06-09 18:16:24,513 INFO mapreduce.JobSubmitter: Executing with tokens: []
2024-06-09 18:16:24,755 INFO conf.Configuration: resource-types.xml not found
2024-06-09 18:16:24,755 INFO resource.ResourceUtils: Unable to find 'resource-types.xml'.
2024-06-09 18:16:25,294 INFO impl.YarnClientImpl: Submitted application application_1717955085543_0001
2024-06-09 18:16:25,415 INFO mapreduce.Job: The url to track the job: http://ip-172-31-53-255.ec2.internal:20888/proxy/application_1717955085543_0001/
2024-06-09 18:16:25,417 INFO mapreduce.Job: Running job: job_1717955085543_0001
2024-06-09 18:16:37,585 INFO mapreduce.Job: Job job_1717955085543_0001 running in uber mode : false
2024-06-09 18:16:37,586 INFO mapreduce.Job:  map 0% reduce 0%
2024-06-09 18:16:55,823 INFO mapreduce.Job:  map 6% reduce 0%
2024-06-09 18:16:56,828 INFO mapreduce.Job:  map 13% reduce 0%
2024-06-09 18:17:09,917 INFO mapreduce.Job:  map 19% reduce 0%
2024-06-09 18:17:10,924 INFO mapreduce.Job:  map 38% reduce 0%
2024-06-09 18:17:12,941 INFO mapreduce.Job:  map 54% reduce 0%
2024-06-09 18:17:13,946 INFO mapreduce.Job:  map 67% reduce 0%
2024-06-09 18:17:15,957 INFO mapreduce.Job:  map 69% reduce 0%
2024-06-09 18:17:16,971 INFO mapreduce.Job:  map 75% reduce 0%
2024-06-09 18:17:17,976 INFO mapreduce.Job:  map 96% reduce 0%
2024-06-09 18:17:18,981 INFO mapreduce.Job:  map 100% reduce 0%
2024-06-09 18:17:32,047 INFO mapreduce.Job:  map 100% reduce 39%
2024-06-09 18:17:34,057 INFO mapreduce.Job:  map 100% reduce 87%
2024-06-09 18:17:36,066 INFO mapreduce.Job:  map 100% reduce 89%
2024-06-09 18:17:38,075 INFO mapreduce.Job:  map 100% reduce 100%
2024-06-09 18:17:39,088 INFO mapreduce.Job: Job job_1717955085543_0001 completed successfully
2024-06-09 18:17:39,229 INFO mapreduce.Job: Counters: 56
	File System Counters
		FILE: Number of bytes read=4682233
		FILE: Number of bytes written=17780414
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=45104845
		HDFS: Number of bytes written=2015382
		HDFS: Number of read operations=58
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=4
		HDFS: Number of bytes read erasure-coded=0
	Job Counters 
		Killed map tasks=1
		Launched map tasks=16
		Launched reduce tasks=2
		Data-local map tasks=14
		Rack-local map tasks=2
		Total time spent by all maps in occupied slots (ms)=23454336
		Total time spent by all reduces in occupied slots (ms)=4710240
		Total time spent by all map tasks (ms)=488632
		Total time spent by all reduce tasks (ms)=49065
		Total vcore-milliseconds taken by all map tasks=488632
		Total vcore-milliseconds taken by all reduce tasks=49065
		Total megabyte-milliseconds taken by all map tasks=750538752
		Total megabyte-milliseconds taken by all reduce tasks=150727680
	Map-Reduce Framework
		Map input records=192577
		Map output records=7124665
		Map output bytes=56117703
		Map output materialized bytes=7802355
		Input split bytes=1776
		Combine input records=0
		Combine output records=0
		Reduce input groups=127867
		Reduce shuffle bytes=7802355
		Reduce input records=7124665
		Reduce output records=127867
		Spilled Records=14249330
		Shuffled Maps =32
		Failed Shuffles=0
		Merged Map outputs=32
		GC time elapsed (ms)=1749
		CPU time spent (ms)=99880
		Physical memory (bytes) snapshot=8752455680
		Virtual memory (bytes) snapshot=60125720576
		Total committed heap usage (bytes)=8860467200
		Peak Map Physical memory (bytes)=529059840
		Peak Map Virtual memory (bytes)=3193434112
		Peak Reduce Physical memory (bytes)=350130176
		Peak Reduce Virtual memory (bytes)=4778999808
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters 
		Bytes Read=45103069
	File Output Format Counters 
		Bytes Written=2015382
2024-06-09 18:17:39,230 INFO streaming.StreamJob: Output directory: /output_1
```

Job ID: job_1717955085543_0001
No. of input splits: 16


### MapReduce jobhistory Web UI

http://<primary_node_DNS>:19888/jobhistory/job/<job_id>

ec2-52-72-29-130.compute-1.amazonaws.com
http://ec2-52-72-29-130.compute-1.amazonaws.com:19888/jobhistory/job/job_1717955085543_0001

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/c3a9cf55-44d4-4991-9e3d-89b31e37eea3">


#### Map tasks

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/c94ba8a6-19a7-4268-9d31-639eebbffcda">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/4e65d3c2-5bb0-4261-8e64-2fb3b39d35c6">



<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/d32e0bd8-658a-414f-8061-07970b84f0e6">



Click the link in the **Logs** field, scroll down to the "syslog" section, and open the full log

```
Log Type: syslog
...
```

map task 0

```
/************************************************************
[system properties]
os.name: Linux
os.version: 6.1.91-99.172.amzn2023.x86_64
java.home: /usr/lib/jvm/java-17-amazon-corretto.x86_64
java.runtime.version: 17.0.11+9-LTS
java.vendor: Amazon.com Inc.
java.version: 17.0.11
java.vm.name: OpenJDK 64-Bit Server VM
java.class.path: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000002:/etc/hadoop/conf:/usr/lib/hadoop/hadoop-aliyun-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-aliyun.jar:/usr/lib/hadoop/hadoop-annotations-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-annotations.jar:/usr/lib/hadoop/hadoop-archive-logs-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-archive-logs.jar:/usr/lib/hadoop/hadoop-archives-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-archives.jar:/usr/lib/hadoop/hadoop-auth-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-auth.jar:/usr/lib/hadoop/hadoop-aws-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-aws.jar:/usr/lib/hadoop/hadoop-azure-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-azure-datalake-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-azure-datalake.jar:/usr/lib/hadoop/hadoop-azure.jar:/usr/lib/hadoop/hadoop-client-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-client.jar:/usr/lib/hadoop/hadoop-common-3.3.6-amzn-3-tests.jar:/usr/lib/hadoop/hadoop-common-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-common.jar:/usr/lib/hadoop/hadoop-datajoin-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-datajoin.jar:/usr/lib/hadoop/hadoop-distcp-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-distcp.jar:/usr/lib/hadoop/hadoop-dynamometer-blockgen-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-dynamometer-blockgen.jar:/usr/lib/hadoop/hadoop-dynamometer-infra-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-dynamometer-infra.jar:/usr/lib/hadoop/hadoop-dynamometer-workload-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-dynamometer-workload.jar:/usr/lib/hadoop/hadoop-extras-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-extras.jar:/usr/lib/hadoop/hadoop-fs2img-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-fs2img.jar:/usr/lib/hadoop/hadoop-gridmix-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-gridmix.jar:/usr/lib/hadoop/hadoop-kafka-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-kafka.jar:/usr/lib/hadoop/hadoop-kms-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-kms.jar:/usr/lib/hadoop/hadoop-minicluster-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-minicluster.jar:/usr/lib/hadoop/hadoop-nfs-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-nfs.jar:/usr/lib/hadoop/hadoop-registry-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-registry.jar:/usr/lib/hadoop/hadoop-resourceestimator-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-resourceestimator.jar:/usr/lib/hadoop/hadoop-rumen-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-rumen.jar:/usr/lib/hadoop/hadoop-shaded-guava-1.1.1.jar:/usr/lib/hadoop/hadoop-shaded-protobuf_3_7-1.1.1.jar:/usr/lib/hadoop/hadoop-sls-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-sls.jar:/usr/lib/hadoop/hadoop-streaming-3.3.6-amzn-3.jar:/usr/lib/hadoop/hadoop-streaming.jar:/usr/lib/hadoop/lib/netty-transport-native-epoll-4.1.100.Final-linux-aarch_64.jar:/usr/lib/hadoop/lib/netty-transport-native-kqueue-4.1.100.Final-osx-aarch_64.jar:/usr/lib/hadoop/lib/netty-transport-native-unix-common-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-transport-sctp-4.1.100.Final.jar:/usr/lib/hadoop/lib/nimbus-jose-jwt-9.8.1.jar:/usr/lib/hadoop/lib/protobuf-java-2.5.0.jar:/usr/lib/hadoop/lib/reload4j-1.2.22.jar:/usr/lib/hadoop/lib/slf4j-reload4j-1.7.36.jar:/usr/lib/hadoop/lib/stax2-api-4.2.1.jar:/usr/lib/hadoop/lib/token-provider-1.0.1.jar:/usr/lib/hadoop/lib/woodstox-core-5.4.0.jar:/usr/lib/hadoop/lib/zookeeper-3.9.1-amzn-0.jar:/usr/lib/hadoop/lib/zookeeper-d*.jar:/usr/lib/hadoop/lib/zookeeper-jute-3.9.1-amzn-0.jar:/usr/lib/hadoop/lib/animal-sniffer-annotations-1.17.jar:/usr/lib/hadoop/lib/audience-annotations-0.12.0.jar:/usr/lib/hadoop/lib/avro-1.7.7.jar:/usr/lib/hadoop/lib/checker-qual-2.5.2.jar:/usr/lib/hadoop/lib/commons-beanutils-1.9.4.jar:/usr/lib/hadoop/lib/commons-cli-1.2.jar:/usr/lib/hadoop/lib/commons-codec-1.15.jar:/usr/lib/hadoop/lib/commons-collections-3.2.2.jar:/usr/lib/hadoop/lib/commons-compress-1.21.jar:/usr/lib/hadoop/lib/commons-configuration2-2.8.0.jar:/usr/lib/hadoop/lib/commons-daemon-1.0.13.jar:/usr/lib/hadoop/lib/commons-io-2.8.0.jar:/usr/lib/hadoop/lib/commons-lang3-3.12.0.jar:/usr/lib/hadoop/lib/commons-logging-1.1.3.jar:/usr/lib/hadoop/lib/commons-math3-3.1.1.jar:/usr/lib/hadoop/lib/commons-net-3.9.0.jar:/usr/lib/hadoop/lib/commons-text-1.10.0.jar:/usr/lib/hadoop/lib/curator-client-5.2.0.jar:/usr/lib/hadoop/lib/curator-framework-5.2.0.jar:/usr/lib/hadoop/lib/curator-recipes-5.2.0.jar:/usr/lib/hadoop/lib/dnsjava-2.1.7.jar:/usr/lib/hadoop/lib/failureaccess-1.0.jar:/usr/lib/hadoop/lib/gson-2.9.0.jar:/usr/lib/hadoop/lib/guava-27.0-jre.jar:/usr/lib/hadoop/lib/httpclient-4.5.13.jar:/usr/lib/hadoop/lib/httpcore-4.4.13.jar:/usr/lib/hadoop/lib/j2objc-annotations-1.1.jar:/usr/lib/hadoop/lib/jackson-annotations-2.12.7.jar:/usr/lib/hadoop/lib/jackson-core-2.12.7.jar:/usr/lib/hadoop/lib/jackson-core-asl-1.9.13.jar:/usr/lib/hadoop/lib/jackson-databind-2.12.7.1.jar:/usr/lib/hadoop/lib/jackson-mapper-asl-1.9.13.jar:/usr/lib/hadoop/lib/jakarta.activation-api-1.2.1.jar:/usr/lib/hadoop/lib/javax.servlet-api-3.1.0.jar:/usr/lib/hadoop/lib/jaxb-api-2.2.11.jar:/usr/lib/hadoop/lib/jaxb-impl-2.2.3-1.jar:/usr/lib/hadoop/lib/jcip-annotations-1.0-1.jar:/usr/lib/hadoop/lib/jersey-core-1.19.4.jar:/usr/lib/hadoop/lib/jersey-json-1.20.jar:/usr/lib/hadoop/lib/jersey-server-1.19.4.jar:/usr/lib/hadoop/lib/jersey-servlet-1.19.4.jar:/usr/lib/hadoop/lib/jettison-1.5.4.jar:/usr/lib/hadoop/lib/jetty-http-9.4.53.v20231009.jar:/usr/lib/hadoop/lib/jetty-io-9.4.53.v20231009.jar:/usr/lib/hadoop/lib/jetty-security-9.4.53.v20231009.jar:/usr/lib/hadoop/lib/jetty-server-9.4.53.v20231009.jar:/usr/lib/hadoop/lib/jetty-servlet-9.4.53.v20231009.jar:/usr/lib/hadoop/lib/jetty-util-9.4.53.v20231009.jar:/usr/lib/hadoop/lib/jetty-util-ajax-9.4.53.v20231009.jar:/usr/lib/hadoop/lib/jetty-webapp-9.4.53.v20231009.jar:/usr/lib/hadoop/lib/jetty-xml-9.4.53.v20231009.jar:/usr/lib/hadoop/lib/jsch-0.1.55.jar:/usr/lib/hadoop/lib/jsp-api-2.1.jar:/usr/lib/hadoop/lib/jsr305-3.0.2.jar:/usr/lib/hadoop/lib/jsr311-api-1.1.1.jar:/usr/lib/hadoop/lib/jul-to-slf4j-1.7.36.jar:/usr/lib/hadoop/lib/kerb-admin-1.0.1.jar:/usr/lib/hadoop/lib/kerb-client-1.0.1.jar:/usr/lib/hadoop/lib/kerb-common-1.0.1.jar:/usr/lib/hadoop/lib/kerb-core-1.0.1.jar:/usr/lib/hadoop/lib/kerb-crypto-1.0.1.jar:/usr/lib/hadoop/lib/kerb-identity-1.0.1.jar:/usr/lib/hadoop/lib/kerb-server-1.0.1.jar:/usr/lib/hadoop/lib/kerb-simplekdc-1.0.1.jar:/usr/lib/hadoop/lib/kerb-util-1.0.1.jar:/usr/lib/hadoop/lib/kerby-asn1-1.0.1.jar:/usr/lib/hadoop/lib/kerby-config-1.0.1.jar:/usr/lib/hadoop/lib/kerby-pkix-1.0.1.jar:/usr/lib/hadoop/lib/kerby-util-1.0.1.jar:/usr/lib/hadoop/lib/kerby-xdr-1.0.1.jar:/usr/lib/hadoop/lib/listenablefuture-9999.0-empty-to-avoid-conflict-with-guava.jar:/usr/lib/hadoop/lib/metrics-core-3.2.4.jar:/usr/lib/hadoop/lib/netty-all-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-buffer-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-codec-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-codec-dns-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-codec-haproxy-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-codec-http-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-codec-http2-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-codec-memcache-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-codec-mqtt-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-codec-redis-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-codec-smtp-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-codec-socks-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-codec-stomp-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-codec-xml-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-common-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-handler-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-handler-proxy-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-handler-ssl-ocsp-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-resolver-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-resolver-dns-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-resolver-dns-classes-macos-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-resolver-dns-native-macos-4.1.100.Final-osx-aarch_64.jar:/usr/lib/hadoop/lib/netty-resolver-dns-native-macos-4.1.100.Final-osx-x86_64.jar:/usr/lib/hadoop/lib/netty-tcnative-boringssl-static-2.0.61.Final-linux-aarch_64.jar:/usr/lib/hadoop/lib/netty-tcnative-boringssl-static-2.0.61.Final-linux-x86_64.jar:/usr/lib/hadoop/lib/netty-tcnative-boringssl-static-2.0.61.Final-osx-aarch_64.jar:/usr/lib/hadoop/lib/netty-tcnative-boringssl-static-2.0.61.Final-osx-x86_64.jar:/usr/lib/hadoop/lib/netty-tcnative-boringssl-static-2.0.61.Final-windows-x86_64.jar:/usr/lib/hadoop/lib/netty-tcnative-boringssl-static-2.0.61.Final.jar:/usr/lib/hadoop/lib/netty-tcnative-classes-2.0.61.Final.jar:/usr/lib/hadoop/lib/netty-transport-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-transport-classes-epoll-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-transport-classes-kqueue-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-transport-native-epoll-4.1.100.Final-linux-x86_64.jar:/usr/lib/hadoop/lib/netty-transport-native-kqueue-4.1.100.Final-osx-x86_64.jar:/usr/lib/hadoop/lib/netty-transport-rxtx-4.1.100.Final.jar:/usr/lib/hadoop/lib/netty-transport-udt-4.1.100.Final.jar:/usr/lib/hadoop/lib/paranamer-2.3.jar:/usr/lib/hadoop/lib/re2j-1.1.jar:/usr/lib/hadoop/lib/slf4j-api-1.7.36.jar:/usr/lib/hadoop/lib/snappy-java-1.1.10.1.jar:/usr/lib/hadoop-hdfs/hadoop-hdfs-3.3.6-amzn-3-tests.jar:/usr/lib/hadoop-hdfs/hadoop-hdfs-3.3.6-amzn-3.jar:/usr/lib/hadoop-hdfs/hadoop-hdfs-client-3.3.6-amzn-3-tests.jar:/usr/lib/hadoop-hdfs/hadoop-hdfs-client-3.3.6-amzn-3.jar:/usr/lib/hadoop-hdfs/hadoop-hdfs-client.jar:/usr/lib/hadoop-hdfs/hadoop-hdfs-httpfs-3.3.6-amzn-3.jar:/usr/lib/hadoop-hdfs/hadoop-hdfs-httpfs.jar:/usr/lib/hadoop-hdfs/hadoop-hdfs-native-client-3.3.6-amzn-3-tests.jar:/usr/lib/hadoop-hdfs/hadoop-hdfs-native-client-3.3.6-amzn-3.jar:/usr/lib/hadoop-hdfs/hadoop-hdfs-native-client.jar:/usr/lib/hadoop-hdfs/hadoop-hdfs-nfs-3.3.6-amzn-3.jar:/usr/lib/hadoop-hdfs/hadoop-hdfs-nfs.jar:/usr/lib/hadoop-hdfs/hadoop-hdfs-rbf-3.3.6-amzn-3-tests.jar:/usr/lib/hadoop-hdfs/hadoop-hdfs-rbf-3.3.6-amzn-3.jar:/usr/lib/hadoop-hdfs/hadoop-hdfs-rbf.jar:/usr/lib/hadoop-hdfs/hadoop-hdfs.jar:/usr/lib/hadoop-hdfs/lib/netty-common-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-handler-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-handler-proxy-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-handler-ssl-ocsp-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-resolver-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-resolver-dns-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-resolver-dns-classes-macos-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-resolver-dns-native-macos-4.1.100.Final-osx-aarch_64.jar:/usr/lib/hadoop-hdfs/lib/netty-resolver-dns-native-macos-4.1.100.Final-osx-x86_64.jar:/usr/lib/hadoop-hdfs/lib/netty-tcnative-boringssl-static-2.0.61.Final-linux-aarch_64.jar:/usr/lib/hadoop-hdfs/lib/netty-tcnative-boringssl-static-2.0.61.Final-linux-x86_64.jar:/usr/lib/hadoop-hdfs/lib/netty-tcnative-boringssl-static-2.0.61.Final-osx-aarch_64.jar:/usr/lib/hadoop-hdfs/lib/netty-tcnative-boringssl-static-2.0.61.Final-osx-x86_64.jar:/usr/lib/hadoop-hdfs/lib/netty-tcnative-boringssl-static-2.0.61.Final-windows-x86_64.jar:/usr/lib/hadoop-hdfs/lib/netty-tcnative-boringssl-static-2.0.61.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-tcnative-classes-2.0.61.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-transport-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-transport-classes-epoll-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-transport-classes-kqueue-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-transport-native-epoll-4.1.100.Final-linux-aarch_64.jar:/usr/lib/hadoop-hdfs/lib/netty-transport-native-epoll-4.1.100.Final-linux-x86_64.jar:/usr/lib/hadoop-hdfs/lib/netty-transport-native-kqueue-4.1.100.Final-osx-aarch_64.jar:/usr/lib/hadoop-hdfs/lib/netty-transport-native-kqueue-4.1.100.Final-osx-x86_64.jar:/usr/lib/hadoop-hdfs/lib/netty-transport-native-unix-common-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-transport-rxtx-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-transport-sctp-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-transport-udt-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/nimbus-jose-jwt-9.8.1.jar:/usr/lib/hadoop-hdfs/lib/okhttp-4.9.3.jar:/usr/lib/hadoop-hdfs/lib/okio-2.8.0.jar:/usr/lib/hadoop-hdfs/lib/paranamer-2.3.jar:/usr/lib/hadoop-hdfs/lib/protobuf-java-2.5.0.jar:/usr/lib/hadoop-hdfs/lib/re2j-1.1.jar:/usr/lib/hadoop-hdfs/lib/reload4j-1.2.22.jar:/usr/lib/hadoop-hdfs/lib/snappy-java-1.1.10.1.jar:/usr/lib/hadoop-hdfs/lib/stax2-api-4.2.1.jar:/usr/lib/hadoop-hdfs/lib/token-provider-1.0.1.jar:/usr/lib/hadoop-hdfs/lib/woodstox-core-5.4.0.jar:/usr/lib/hadoop-hdfs/lib/zookeeper-3.9.1-amzn-0.jar:/usr/lib/hadoop-hdfs/lib/zookeeper-jute-3.9.1-amzn-0.jar:/usr/lib/hadoop-hdfs/lib/HikariCP-java7-2.4.12.jar:/usr/lib/hadoop-hdfs/lib/animal-sniffer-annotations-1.17.jar:/usr/lib/hadoop-hdfs/lib/audience-annotations-0.12.0.jar:/usr/lib/hadoop-hdfs/lib/avro-1.7.7.jar:/usr/lib/hadoop-hdfs/lib/checker-qual-2.5.2.jar:/usr/lib/hadoop-hdfs/lib/commons-beanutils-1.9.4.jar:/usr/lib/hadoop-hdfs/lib/commons-cli-1.2.jar:/usr/lib/hadoop-hdfs/lib/commons-codec-1.15.jar:/usr/lib/hadoop-hdfs/lib/commons-collections-3.2.2.jar:/usr/lib/hadoop-hdfs/lib/commons-compress-1.21.jar:/usr/lib/hadoop-hdfs/lib/commons-configuration2-2.8.0.jar:/usr/lib/hadoop-hdfs/lib/commons-daemon-1.0.13.jar:/usr/lib/hadoop-hdfs/lib/commons-io-2.8.0.jar:/usr/lib/hadoop-hdfs/lib/commons-lang3-3.12.0.jar:/usr/lib/hadoop-hdfs/lib/commons-logging-1.1.3.jar:/usr/lib/hadoop-hdfs/lib/commons-math3-3.1.1.jar:/usr/lib/hadoop-hdfs/lib/commons-net-3.9.0.jar:/usr/lib/hadoop-hdfs/lib/commons-text-1.10.0.jar:/usr/lib/hadoop-hdfs/lib/curator-client-5.2.0.jar:/usr/lib/hadoop-hdfs/lib/curator-framework-5.2.0.jar:/usr/lib/hadoop-hdfs/lib/curator-recipes-5.2.0.jar:/usr/lib/hadoop-hdfs/lib/dnsjava-2.1.7.jar:/usr/lib/hadoop-hdfs/lib/failureaccess-1.0.jar:/usr/lib/hadoop-hdfs/lib/gson-2.9.0.jar:/usr/lib/hadoop-hdfs/lib/guava-27.0-jre.jar:/usr/lib/hadoop-hdfs/lib/httpclient-4.5.13.jar:/usr/lib/hadoop-hdfs/lib/httpcore-4.4.13.jar:/usr/lib/hadoop-hdfs/lib/j2objc-annotations-1.1.jar:/usr/lib/hadoop-hdfs/lib/jackson-annotations-2.12.7.jar:/usr/lib/hadoop-hdfs/lib/jackson-core-2.12.7.jar:/usr/lib/hadoop-hdfs/lib/jackson-core-asl-1.9.13.jar:/usr/lib/hadoop-hdfs/lib/jackson-databind-2.12.7.1.jar:/usr/lib/hadoop-hdfs/lib/jackson-mapper-asl-1.9.13.jar:/usr/lib/hadoop-hdfs/lib/jakarta.activation-api-1.2.1.jar:/usr/lib/hadoop-hdfs/lib/javax.servlet-api-3.1.0.jar:/usr/lib/hadoop-hdfs/lib/jaxb-api-2.2.11.jar:/usr/lib/hadoop-hdfs/lib/jaxb-impl-2.2.3-1.jar:/usr/lib/hadoop-hdfs/lib/jcip-annotations-1.0-1.jar:/usr/lib/hadoop-hdfs/lib/jersey-core-1.19.4.jar:/usr/lib/hadoop-hdfs/lib/jersey-json-1.20.jar:/usr/lib/hadoop-hdfs/lib/jersey-server-1.19.4.jar:/usr/lib/hadoop-hdfs/lib/jersey-servlet-1.19.4.jar:/usr/lib/hadoop-hdfs/lib/jettison-1.5.4.jar:/usr/lib/hadoop-hdfs/lib/jetty-http-9.4.53.v20231009.jar:/usr/lib/hadoop-hdfs/lib/jetty-io-9.4.53.v20231009.jar:/usr/lib/hadoop-hdfs/lib/jetty-security-9.4.53.v20231009.jar:/usr/lib/hadoop-hdfs/lib/jetty-server-9.4.53.v20231009.jar:/usr/lib/hadoop-hdfs/lib/jetty-servlet-9.4.53.v20231009.jar:/usr/lib/hadoop-hdfs/lib/jetty-util-9.4.53.v20231009.jar:/usr/lib/hadoop-hdfs/lib/jetty-util-ajax-9.4.53.v20231009.jar:/usr/lib/hadoop-hdfs/lib/jetty-webapp-9.4.53.v20231009.jar:/usr/lib/hadoop-hdfs/lib/jetty-xml-9.4.53.v20231009.jar:/usr/lib/hadoop-hdfs/lib/jsch-0.1.55.jar:/usr/lib/hadoop-hdfs/lib/json-simple-1.1.1.jar:/usr/lib/hadoop-hdfs/lib/jsr305-3.0.2.jar:/usr/lib/hadoop-hdfs/lib/jsr311-api-1.1.1.jar:/usr/lib/hadoop-hdfs/lib/kerb-admin-1.0.1.jar:/usr/lib/hadoop-hdfs/lib/kerb-client-1.0.1.jar:/usr/lib/hadoop-hdfs/lib/kerb-common-1.0.1.jar:/usr/lib/hadoop-hdfs/lib/kerb-core-1.0.1.jar:/usr/lib/hadoop-hdfs/lib/kerb-crypto-1.0.1.jar:/usr/lib/hadoop-hdfs/lib/kerb-identity-1.0.1.jar:/usr/lib/hadoop-hdfs/lib/kerb-server-1.0.1.jar:/usr/lib/hadoop-hdfs/lib/kerb-simplekdc-1.0.1.jar:/usr/lib/hadoop-hdfs/lib/kerb-util-1.0.1.jar:/usr/lib/hadoop-hdfs/lib/kerby-asn1-1.0.1.jar:/usr/lib/hadoop-hdfs/lib/kerby-config-1.0.1.jar:/usr/lib/hadoop-hdfs/lib/kerby-pkix-1.0.1.jar:/usr/lib/hadoop-hdfs/lib/kerby-util-1.0.1.jar:/usr/lib/hadoop-hdfs/lib/kerby-xdr-1.0.1.jar:/usr/lib/hadoop-hdfs/lib/kotlin-stdlib-1.4.10.jar:/usr/lib/hadoop-hdfs/lib/kotlin-stdlib-common-1.4.10.jar:/usr/lib/hadoop-hdfs/lib/leveldbjni-all-1.8.jar:/usr/lib/hadoop-hdfs/lib/listenablefuture-9999.0-empty-to-avoid-conflict-with-guava.jar:/usr/lib/hadoop-hdfs/lib/metrics-core-3.2.4.jar:/usr/lib/hadoop-hdfs/lib/netty-all-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-buffer-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-codec-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-codec-dns-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-codec-haproxy-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-codec-http-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-codec-http2-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-codec-memcache-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-codec-mqtt-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-codec-redis-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-codec-smtp-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-codec-socks-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-codec-stomp-4.1.100.Final.jar:/usr/lib/hadoop-hdfs/lib/netty-codec-xml-4.1.100.Final.jar:/usr/lib/hadoop-mapreduce/aliyun-java-sdk-core-4.5.10.jar:/usr/lib/hadoop-mapreduce/aliyun-java-sdk-kms-2.11.0.jar:/usr/lib/hadoop-mapreduce/aliyun-java-sdk-ram-3.1.0.jar:/usr/lib/hadoop-mapreduce/aliyun-sdk-oss-3.13.0.jar:/usr/lib/hadoop-mapreduce/azure-data-lake-store-sdk-2.3.9.jar:/usr/lib/hadoop-mapreduce/azure-keyvault-core-1.0.0.jar:/usr/lib/hadoop-mapreduce/azure-storage-7.0.1.jar:/usr/lib/hadoop-mapreduce/hadoop-aliyun-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-aliyun.jar:/usr/lib/hadoop-mapreduce/hadoop-archive-logs-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-archive-logs.jar:/usr/lib/hadoop-mapreduce/hadoop-archives-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-archives.jar:/usr/lib/hadoop-mapreduce/hadoop-aws-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-aws.jar:/usr/lib/hadoop-mapreduce/hadoop-azure-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-azure-datalake-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-azure-datalake.jar:/usr/lib/hadoop-mapreduce/hadoop-azure.jar:/usr/lib/hadoop-mapreduce/hadoop-client-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-client.jar:/usr/lib/hadoop-mapreduce/hadoop-datajoin-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-datajoin.jar:/usr/lib/hadoop-mapreduce/hadoop-distcp-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-distcp.jar:/usr/lib/hadoop-mapreduce/hadoop-dynamometer-blockgen-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-dynamometer-blockgen.jar:/usr/lib/hadoop-mapreduce/hadoop-dynamometer-infra-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-dynamometer-infra.jar:/usr/lib/hadoop-mapreduce/hadoop-dynamometer-workload-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-dynamometer-workload.jar:/usr/lib/hadoop-mapreduce/hadoop-extras-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-extras.jar:/usr/lib/hadoop-mapreduce/hadoop-fs2img-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-fs2img.jar:/usr/lib/hadoop-mapreduce/hadoop-gridmix-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-gridmix.jar:/usr/lib/hadoop-mapreduce/hadoop-kafka-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-kafka.jar:/usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-app-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-app.jar:/usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-common-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-common.jar:/usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-core-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-core.jar:/usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-hs-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-hs-plugins-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-hs-plugins.jar:/usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-hs.jar:/usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-jobclient-3.3.6-amzn-3-tests.jar:/usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-jobclient-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-jobclient.jar:/usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-nativetask-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-nativetask.jar:/usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-shuffle-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-shuffle.jar:/usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-uploader-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-mapreduce-client-uploader.jar:/usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar:/usr/lib/hadoop-mapreduce/hadoop-minicluster-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-minicluster.jar:/usr/lib/hadoop-mapreduce/hadoop-resourceestimator-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-resourceestimator.jar:/usr/lib/hadoop-mapreduce/hadoop-rumen-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-rumen.jar:/usr/lib/hadoop-mapreduce/hadoop-sls-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-sls.jar:/usr/lib/hadoop-mapreduce/hadoop-streaming-3.3.6-amzn-3.jar:/usr/lib/hadoop-mapreduce/hadoop-streaming.jar:/usr/lib/hadoop-mapreduce/hamcrest-core-1.3.jar:/usr/lib/hadoop-mapreduce/ini4j-0.5.4.jar:/usr/lib/hadoop-mapreduce/jdk.tools-1.8.jar:/usr/lib/hadoop-mapreduce/jdom2-2.0.6.jar:/usr/lib/hadoop-mapreduce/junit-4.13.2.jar:/usr/lib/hadoop-mapreduce/kafka-clients-2.8.2.jar:/usr/lib/hadoop-mapreduce/lz4-java-1.7.1.jar:/usr/lib/hadoop-mapreduce/ojalgo-43.0.jar:/usr/lib/hadoop-mapreduce/opentracing-api-0.33.0.jar:/usr/lib/hadoop-mapreduce/opentracing-noop-0.33.0.jar:/usr/lib/hadoop-mapreduce/opentracing-util-0.33.0.jar:/usr/lib/hadoop-mapreduce/org.jacoco.agent-0.8.5-runtime.jar:/usr/lib/hadoop-mapreduce/wildfly-openssl-1.1.3.Final.jar:/usr/lib/hadoop-mapreduce/zstd-jni-1.4.9-1.jar:/usr/lib/hadoop-mapreduce/lib/*:/usr/lib/hadoop-yarn/hadoop-yarn-api-3.3.6-amzn-3.jar:/usr/lib/hadoop-yarn/hadoop-yarn-api.jar:/usr/lib/hadoop-yarn/hadoop-yarn-applications-distributedshell-3.3.6-amzn-3.jar:/usr/lib/hadoop-yarn/hadoop-yarn-applications-distributedshell.jar:/usr/lib/hadoop-yarn/hadoop-yarn-applications-mawo-core-3.3.6-amzn-3.jar:/usr/lib/hadoop-yarn/hadoop-yarn-applications-mawo-core.jar:/usr/lib/hadoop-yarn/hadoop-yarn-applications-unmanaged-am-launcher-3.3.6-amzn-3.jar:/usr/lib/hadoop-yarn/hadoop-yarn-applications-unmanaged-am-launcher.jar:/usr/lib/hadoop-yarn/hadoop-yarn-client-3.3.6-amzn-3.jar:/usr/lib/hadoop-yarn/hadoop-yarn-client.jar:/usr/lib/hadoop-yarn/hadoop-yarn-common-3.3.6-amzn-3.jar:/usr/lib/hadoop-yarn/hadoop-yarn-common.jar:/usr/lib/hadoop-yarn/hadoop-yarn-registry-3.3.6-amzn-3.jar:/usr/lib/hadoop-yarn/hadoop-yarn-registry.jar:/usr/lib/hadoop-yarn/hadoop-yarn-server-applicationhistoryservice-3.3.6-amzn-3.jar:/usr/lib/hadoop-yarn/hadoop-yarn-server-applicationhistoryservice.jar:/usr/lib/hadoop-yarn/hadoop-yarn-server-common-3.3.6-amzn-3.jar:/usr/lib/hadoop-yarn/hadoop-yarn-server-common.jar:/usr/lib/hadoop-yarn/hadoop-yarn-server-nodemanager-3.3.6-amzn-3.jar:/usr/lib/hadoop-yarn/hadoop-yarn-server-nodemanager.jar:/usr/lib/hadoop-yarn/hadoop-yarn-server-resourcemanager-3.3.6-amzn-3.jar:/usr/lib/hadoop-yarn/hadoop-yarn-server-resourcemanager.jar:/usr/lib/hadoop-yarn/hadoop-yarn-server-router-3.3.6-amzn-3.jar:/usr/lib/hadoop-yarn/hadoop-yarn-server-router.jar:/usr/lib/hadoop-yarn/hadoop-yarn-server-sharedcachemanager-3.3.6-amzn-3.jar:/usr/lib/hadoop-yarn/hadoop-yarn-server-sharedcachemanager.jar:/usr/lib/hadoop-yarn/hadoop-yarn-server-tests-3.3.6-amzn-3.jar:/usr/lib/hadoop-yarn/hadoop-yarn-server-tests.jar:/usr/lib/hadoop-yarn/hadoop-yarn-server-timeline-pluginstorage-3.3.6-amzn-3.jar:/usr/lib/hadoop-yarn/hadoop-yarn-server-timeline-pluginstorage.jar:/usr/lib/hadoop-yarn/hadoop-yarn-server-web-proxy-3.3.6-amzn-3.jar:/usr/lib/hadoop-yarn/hadoop-yarn-server-web-proxy.jar:/usr/lib/hadoop-yarn/hadoop-yarn-services-api-3.3.6-amzn-3.jar:/usr/lib/hadoop-yarn/hadoop-yarn-services-api.jar:/usr/lib/hadoop-yarn/hadoop-yarn-services-core-3.3.6-amzn-3.jar:/usr/lib/hadoop-yarn/hadoop-yarn-services-core.jar:/usr/lib/hadoop-yarn/lib/aopalliance-1.0.jar:/usr/lib/hadoop-yarn/lib/asm-commons-9.6.jar:/usr/lib/hadoop-yarn/lib/asm-tree-9.6.jar:/usr/lib/hadoop-yarn/lib/bcpkix-jdk15on-1.68.jar:/usr/lib/hadoop-yarn/lib/bcprov-jdk15on-1.68.jar:/usr/lib/hadoop-yarn/lib/ehcache-3.3.1.jar:/usr/lib/hadoop-yarn/lib/fst-2.50.jar:/usr/lib/hadoop-yarn/lib/geronimo-jcache_1.0_spec-1.0-alpha-1.jar:/usr/lib/hadoop-yarn/lib/guice-5.1.0.jar:/usr/lib/hadoop-yarn/lib/guice-servlet-5.1.0.jar:/usr/lib/hadoop-yarn/lib/jackson-jaxrs-base-2.12.7.jar:/usr/lib/hadoop-yarn/lib/jackson-jaxrs-json-provider-2.12.7.jar:/usr/lib/hadoop-yarn/lib/jackson-module-jaxb-annotations-2.12.7.jar:/usr/lib/hadoop-yarn/lib/jakarta.xml.bind-api-2.3.2.jar:/usr/lib/hadoop-yarn/lib/java-util-1.9.0.jar:/usr/lib/hadoop-yarn/lib/javax-websocket-client-impl-9.4.53.v20231009.jar:/usr/lib/hadoop-yarn/lib/javax-websocket-server-impl-9.4.53.v20231009.jar:/usr/lib/hadoop-yarn/lib/javax.inject-1.jar:/usr/lib/hadoop-yarn/lib/javax.websocket-api-1.0.jar:/usr/lib/hadoop-yarn/lib/javax.websocket-client-api-1.0.jar:/usr/lib/hadoop-yarn/lib/jersey-client-1.19.4.jar:/usr/lib/hadoop-yarn/lib/jersey-guice-1.19.4.jar:/usr/lib/hadoop-yarn/lib/jetty-annotations-9.4.53.v20231009.jar:/usr/lib/hadoop-yarn/lib/jetty-client-9.4.53.v20231009.jar:/usr/lib/hadoop-yarn/lib/jetty-jndi-9.4.53.v20231009.jar:/usr/lib/hadoop-yarn/lib/jetty-plus-9.4.53.v20231009.jar:/usr/lib/hadoop-yarn/lib/jline-3.9.0.jar:/usr/lib/hadoop-yarn/lib/jna-5.2.0.jar:/usr/lib/hadoop-yarn/lib/json-io-2.5.1.jar:/usr/lib/hadoop-yarn/lib/mssql-jdbc-6.2.1.jre7.jar:/usr/lib/hadoop-yarn/lib/objenesis-2.6.jar:/usr/lib/hadoop-yarn/lib/snakeyaml-2.0.jar:/usr/lib/hadoop-yarn/lib/swagger-annotations-1.5.4.jar:/usr/lib/hadoop-yarn/lib/websocket-api-9.4.53.v20231009.jar:/usr/lib/hadoop-yarn/lib/websocket-client-9.4.53.v20231009.jar:/usr/lib/hadoop-yarn/lib/websocket-common-9.4.53.v20231009.jar:/usr/lib/hadoop-yarn/lib/websocket-server-9.4.53.v20231009.jar:/usr/lib/hadoop-yarn/lib/websocket-servlet-9.4.53.v20231009.jar:/usr/lib/hadoop-lzo/lib/hadoop-lzo-0.4.19.jar:/usr/lib/hadoop-lzo/lib/hadoop-lzo.jar:/usr/share/aws/emr/emrfs/conf:/usr/share/aws/emr/emrfs/lib/animal-sniffer-annotations-1.14.jar:/usr/share/aws/emr/emrfs/lib/annotations-16.0.2.jar:/usr/share/aws/emr/emrfs/lib/aopalliance-1.0.jar:/usr/share/aws/emr/emrfs/lib/bcprov-ext-jdk15on-1.66.jar:/usr/share/aws/emr/emrfs/lib/checker-qual-2.5.2.jar:/usr/share/aws/emr/emrfs/lib/emrfs-hadoop-assembly-2.62.0.jar:/usr/share/aws/emr/emrfs/lib/error_prone_annotations-2.1.3.jar:/usr/share/aws/emr/emrfs/lib/findbugs-annotations-3.0.1.jar:/usr/share/aws/emr/emrfs/lib/j2objc-annotations-1.1.jar:/usr/share/aws/emr/emrfs/lib/javax.inject-1.jar:/usr/share/aws/emr/emrfs/lib/jmespath-java-1.12.656.jar:/usr/share/aws/emr/emrfs/lib/jsr305-3.0.2.jar:/usr/share/aws/emr/emrfs/auxlib/*:/usr/share/aws/emr/lib/*:/usr/share/aws/emr/ddb/lib/emr-ddb-hadoop.jar:/usr/share/aws/emr/goodies/lib/emr-hadoop-goodies.jar:/usr/share/aws/emr/kinesis/lib/emr-kinesis-hadoop.jar:/usr/lib/spark/yarn/lib/datanucleus-api-jdo.jar:/usr/lib/spark/yarn/lib/datanucleus-core.jar:/usr/lib/spark/yarn/lib/datanucleus-rdbms.jar:/usr/share/aws/emr/cloudwatch-sink/lib/cloudwatch-sink-2.10.0.jar:/usr/share/aws/emr/cloudwatch-sink/lib/cloudwatch-sink.jar:/usr/share/aws/aws-java-sdk/aws-java-sdk-bundle-1.12.656.jar:/usr/share/aws/aws-java-sdk-v2/aws-sdk-java-bundle-2.23.18.jar:/usr/lib/hadoop-mapreduce/share/hadoop/mapreduce/*:/usr/lib/hadoop-mapreduce/share/hadoop/mapreduce/lib/*:/usr/lib/hadoop-lzo/lib/hadoop-lzo-0.4.19.jar:/usr/lib/hadoop-lzo/lib/hadoop-lzo.jar:/usr/share/aws/emr/emrfs/conf:/usr/share/aws/emr/emrfs/lib/animal-sniffer-annotations-1.14.jar:/usr/share/aws/emr/emrfs/lib/annotations-16.0.2.jar:/usr/share/aws/emr/emrfs/lib/aopalliance-1.0.jar:/usr/share/aws/emr/emrfs/lib/bcprov-ext-jdk15on-1.66.jar:/usr/share/aws/emr/emrfs/lib/checker-qual-2.5.2.jar:/usr/share/aws/emr/emrfs/lib/emrfs-hadoop-assembly-2.62.0.jar:/usr/share/aws/emr/emrfs/lib/error_prone_annotations-2.1.3.jar:/usr/share/aws/emr/emrfs/lib/findbugs-annotations-3.0.1.jar:/usr/share/aws/emr/emrfs/lib/j2objc-annotations-1.1.jar:/usr/share/aws/emr/emrfs/lib/javax.inject-1.jar:/usr/share/aws/emr/emrfs/lib/jmespath-java-1.12.656.jar:/usr/share/aws/emr/emrfs/lib/jsr305-3.0.2.jar:/usr/share/aws/emr/emrfs/auxlib/*:/usr/share/aws/emr/lib/*:/usr/share/aws/emr/ddb/lib/emr-ddb-hadoop.jar:/usr/share/aws/emr/goodies/lib/emr-hadoop-goodies.jar:/usr/share/aws/emr/kinesis/lib/emr-kinesis-hadoop.jar:/usr/share/aws/emr/cloudwatch-sink/lib/cloudwatch-sink-2.10.0.jar:/usr/share/aws/emr/cloudwatch-sink/lib/cloudwatch-sink.jar:/usr/share/aws/aws-java-sdk/aws-java-sdk-bundle-1.12.656.jar:/usr/share/aws/aws-java-sdk-v2/aws-sdk-java-bundle-2.23.18.jar:job.jar/job.jar:job.jar/classes/:job.jar/lib/*:/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000002/job.jar
java.io.tmpdir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000002/tmp
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000002
user.name: yarn
************************************************************/
2024-06-09 18:16:54,367 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:16:57,199 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:16:57,199 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:16:57,200 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:16:57,300 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:16:58,320 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:41991735+2799454
2024-06-09 18:16:58,433 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:16:58,449 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:16:58,559 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:16:59,384 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:16:59,384 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:16:59,386 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:16:59,386 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:16:59,386 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:16:59,408 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:16:59,489 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000002/./mapper.py]
```

map task 1
```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000006
user.name: yarn
************************************************************/
2024-06-09 18:17:01,102 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:17:04,160 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:17:04,160 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:17:04,169 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:17:04,281 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:05,142 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:0+2799449
2024-06-09 18:17:05,228 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:17:05,237 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:17:05,394 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:17:06,140 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:17:06,140 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:17:06,141 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:17:06,141 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:17:06,141 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:17:06,220 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:17:06,259 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000006/./mapper.py]
```

map task 2
```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000007
user.name: yarn
************************************************************/
2024-06-09 18:17:00,563 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:17:04,056 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:17:04,056 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:17:04,070 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:17:04,190 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:05,095 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:2799449+2799449
2024-06-09 18:17:05,230 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:17:05,254 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:17:05,356 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:17:06,091 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:17:06,091 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:17:06,091 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:17:06,091 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:17:06,091 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:17:06,116 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:17:06,219 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000007/./mapper.py]
```
map task 3

```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000008
user.name: yarn
************************************************************/
2024-06-09 18:17:00,571 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:17:04,107 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:17:04,107 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:17:04,108 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:17:04,211 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:05,064 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:5598898+2799449
2024-06-09 18:17:05,206 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:17:05,238 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:17:05,367 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:17:06,098 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:17:06,098 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:17:06,098 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:17:06,098 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:17:06,098 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:17:06,112 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:17:06,183 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000008/./mapper.py]
```

map task 4

```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000009
user.name: yarn
************************************************************/
2024-06-09 18:17:00,753 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:17:03,930 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:17:03,930 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:17:03,931 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:17:04,027 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:05,008 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:8398347+2799449
2024-06-09 18:17:05,165 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:17:05,179 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:17:05,306 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:17:06,169 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:17:06,169 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:17:06,169 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:17:06,169 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:17:06,169 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:17:06,258 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:17:06,302 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000009/./mapper.py]

```


5

```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000014
user.name: yarn
************************************************************/
2024-06-09 18:16:48,209 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:16:49,522 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:16:49,522 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:16:49,523 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:16:49,568 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:16:50,113 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:11197796+2799449
2024-06-09 18:16:50,146 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:16:50,174 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:16:50,228 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:16:50,760 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:16:50,760 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:16:50,760 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:16:50,760 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:16:50,760 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:16:50,787 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:16:50,838 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000014/./mapper.py]
```

6
```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000015
user.name: yarn
************************************************************/
2024-06-09 18:16:47,804 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:16:49,205 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:16:49,205 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:16:49,216 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:16:49,309 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:16:49,745 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:13997245+2799449
2024-06-09 18:16:49,812 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:16:49,819 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:16:49,869 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:16:50,153 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:16:50,153 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:16:50,153 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:16:50,153 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:16:50,153 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:16:50,180 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:16:50,251 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000015/./mapper.py]
```

map task 7

```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000010
user.name: yarn
************************************************************/
2024-06-09 18:16:56,213 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:16:59,065 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:16:59,065 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:16:59,066 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:16:59,219 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:00,369 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:16796694+2799449
2024-06-09 18:17:00,555 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:17:00,570 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:17:00,867 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:17:02,478 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:17:02,478 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:17:02,478 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:17:02,478 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:17:02,478 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:17:02,517 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:17:02,638 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000010/./mapper.py]
```

map task 8

```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000011
user.name: yarn
************************************************************/
2024-06-09 18:16:57,058 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:17:00,104 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:17:00,105 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:17:00,105 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:17:00,229 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:01,931 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:19596143+2799449
2024-06-09 18:17:02,127 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:17:02,157 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:17:02,402 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:17:03,549 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:17:03,550 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:17:03,555 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:17:03,555 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:17:03,555 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:17:03,616 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:17:03,714 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000011/./mapper.py]
```

map task 9
```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000012
user.name: yarn
************************************************************/
2024-06-09 18:16:56,840 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:16:59,854 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:16:59,854 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:16:59,855 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:16:59,934 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:01,761 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:22395592+2799449
2024-06-09 18:17:02,029 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:17:02,088 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:17:02,351 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:17:04,038 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:17:04,038 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:17:04,039 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:17:04,039 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:17:04,039 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:17:04,078 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:17:04,202 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000012/./mapper.py]

```

map task 10
```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000013
user.name: yarn
************************************************************/
2024-06-09 18:16:57,512 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:17:00,652 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:17:00,652 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:17:00,653 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:17:01,010 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:03,068 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:25195041+2799449
2024-06-09 18:17:03,289 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:17:03,332 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:17:03,589 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:17:05,621 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:17:05,621 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:17:05,622 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:17:05,622 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:17:05,622 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:17:05,657 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:17:05,784 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000013/./mapper.py]
```



map task 11
```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000016
user.name: yarn
************************************************************/
2024-06-09 18:17:04,164 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:17:05,609 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:17:05,609 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:17:05,610 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:17:05,684 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:06,036 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:27994490+2799449
2024-06-09 18:17:06,085 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:17:06,094 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:17:06,154 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:17:06,545 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:17:06,546 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:17:06,546 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:17:06,546 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:17:06,546 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:17:06,561 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:17:06,602 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000016/./mapper.py]
```


map task 12
```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000017
user.name: yarn
************************************************************/
2024-06-09 18:17:03,998 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:17:05,365 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:17:05,365 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:17:05,366 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:17:05,423 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:05,838 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:30793939+2799449
2024-06-09 18:17:05,863 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:17:05,867 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:17:05,911 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:17:06,242 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:17:06,242 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:17:06,242 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:17:06,242 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:17:06,242 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:17:06,248 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:17:06,261 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000017/./mapper.py]
```


map task 13
```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000003
user.name: yarn
************************************************************/
2024-06-09 18:16:53,926 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:16:56,438 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:16:56,438 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:16:56,439 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:16:56,547 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:16:57,525 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:33593388+2799449
2024-06-09 18:16:57,625 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:16:57,638 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:16:57,720 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:16:58,669 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:16:58,669 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:16:58,669 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:16:58,669 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:16:58,669 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:16:58,699 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:16:58,747 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000003/./mapper.py]
```


map task 14
```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000004
user.name: yarn
************************************************************/
2024-06-09 18:16:54,055 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:16:56,555 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:16:56,555 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:16:56,556 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:16:56,704 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:16:57,635 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:36392837+2799449
2024-06-09 18:16:57,675 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:16:57,691 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:16:57,773 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:16:58,637 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:16:58,638 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:16:58,638 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:16:58,638 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:16:58,638 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:16:58,676 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:16:58,742 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000004/./mapper.py]
2024-06-09 18:16:58,782 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: mapred.work.output.dir is deprecated. Instead, use mapreduce.task.output.dir
```


map task 15
```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000005
user.name: yarn
************************************************************/
2024-06-09 18:16:54,280 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:16:56,855 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:16:56,865 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:16:56,866 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:16:56,952 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:16:57,876 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:39192286+2799449
2024-06-09 18:16:58,007 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:16:58,023 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:16:58,116 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:16:58,909 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:16:58,909 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:16:58,909 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:16:58,909 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:16:58,909 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:16:58,930 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:16:59,024 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000005/./mapper.py]
```

#### Reduce tasks

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/6bcf33a9-d5d0-4996-92e5-0fa8e27bf94f">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/7f2d0bbb-e739-4bfa-8098-993fc835d892">


reduce task 0

```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000018
user.name: yarn
************************************************************/
2024-06-09 18:17:16,987 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:17:18,509 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:17:18,509 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:17:18,511 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:17:18,613 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:18,736 INFO [main] org.apache.hadoop.mapred.ReduceTask: Using ShuffleConsumerPlugin: org.apache.hadoop.mapreduce.task.reduce.Shuffle@25d2f66
2024-06-09 18:17:18,738 WARN [main] org.apache.hadoop.metrics2.impl.MetricsSystemImpl: ReduceTask metrics system already initialized!
2024-06-09 18:17:19,112 INFO [fetcher#3] org.apache.hadoop.io.compress.CodecPool: Got brand-new decompressor [.snappy]
2024-06-09 18:17:19,149 INFO [fetcher#1] org.apache.hadoop.io.compress.CodecPool: Got brand-new decompressor [.snappy]
2024-06-09 18:17:19,152 INFO [fetcher#4] org.apache.hadoop.io.compress.CodecPool: Got brand-new decompressor [.snappy]
2024-06-09 18:17:19,714 INFO [main] org.apache.hadoop.io.compress.CodecPool: Got brand-new compressor [.snappy]
2024-06-09 18:17:24,418 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000018/./reducer.py]
```

reduce task 1
```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000019
user.name: yarn
************************************************************/
2024-06-09 18:17:19,197 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:17:20,887 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:17:20,888 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:17:20,888 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:17:20,941 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:21,029 INFO [main] org.apache.hadoop.mapred.ReduceTask: Using ShuffleConsumerPlugin: org.apache.hadoop.mapreduce.task.reduce.Shuffle@25d2f66
2024-06-09 18:17:21,031 WARN [main] org.apache.hadoop.metrics2.impl.MetricsSystemImpl: ReduceTask metrics system already initialized!
2024-06-09 18:17:21,269 INFO [fetcher#1] org.apache.hadoop.io.compress.CodecPool: Got brand-new decompressor [.snappy]
2024-06-09 18:17:21,295 INFO [fetcher#2] org.apache.hadoop.io.compress.CodecPool: Got brand-new decompressor [.snappy]
2024-06-09 18:17:21,297 INFO [fetcher#3] org.apache.hadoop.io.compress.CodecPool: Got brand-new decompressor [.snappy]
2024-06-09 18:17:21,299 INFO [fetcher#4] org.apache.hadoop.io.compress.CodecPool: Got brand-new decompressor [.snappy]
2024-06-09 18:17:21,638 INFO [main] org.apache.hadoop.io.compress.CodecPool: Got brand-new compressor [.snappy]
2024-06-09 18:17:23,919 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000019/./reducer.py]
```

| Instance | Map tasks |  Reduce tasks | ApplicationMaster|
|------|-----------|--------|--------|
| ip-xxxx-52-142.xxxx|  7, 8, 9, 10 | | |
| ip-xxxx-52-162.xxxx| 5, 6, 11, 12| | yes |
| ip-xxxx-57-35.xxxx  | 1, 2, 3, 4 | | |
| ip-xxxx-63-90.xxxx  | 0, 13,  14, 15| 0, 1| |


<br>

### Resource Manager Web UI

<img width="1416" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/34ab5b87-303e-455a-96e2-9860ee9d8781">
<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/de45b913-6ac0-48cf-84aa-ca85d4281fc5">


<img width="1432" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/62ea968e-9321-4041-99fd-588a27a94068">


Click the link in the Logs field, and replace the private DNS with the public DNS of the primary node 




### YARN timeline Server Web UI

<img width="1417" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/86ba2a8c-0660-4bed-aea9-815c1cc85045">
<img width="1413" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/767506e5-0fdd-4c1a-8357-70ad6f916b05">
<img width="1427" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/89028599-19d2-4fa2-a3d4-9a6aed37d915">
<img width="1428" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/b177ccf8-db48-4115-8537-22357db5485b">

<img width="1430" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/04e02f4e-a849-46b5-bc25-c71a3f7f7960">


<img width="1429" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/efacec64-a94d-4c89-840e-4f7d9d47969e">


 <img width="800" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/f2ecfbcd-5191-47e8-8b62-ced4e2b19eef">
 
- Container 1 (3072 Memory, 1 VCores; Priority: 0) is used to host the application master.

- containers 2-17: 1536 Memory, 1 VCores each; Priority: 20
  
- Containers 18-20: 3072 Memory, 1 VCores each; Priority: 10
  
| Instance | Containers |
|------|-----------|
| ip-xxxx-52-142.xxxx| 10, 11, 12, 13 |  
| ip-xxxx-52-162.xxxx| 1, 14, 15, 16, 17 |  
| ip-xxxx-57-35.xxxx  | 6, 7, 8, 9  |
| ip-xxxx-63-90.xxxx  | 2, 3, 4, 5 , 18, 19, 20|



The Diagnostics field of container 20: *Diagnostics: Container released by application*. The Diagnostics field of the other containers is all empty. Higher Integer value indicates higher priority?
only container 20's exit status is -100.


#### Logs


<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/3590d279-a13e-426a-a244-e8ad225dc95a">
click the link in the last section
Log Type: /logs/containers/application_1717955085543_0001/container_1717955085543_0001_01_000001/syslog.gz


<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/1feb7bfd-da49-45d7-a322-2e912c917e4f">

click the link in the last section shows

```
...
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000002
user.name: yarn
************************************************************/
2024-06-09 18:16:54,367 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:16:57,199 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:16:57,199 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:16:57,200 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:16:57,300 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:16:58,320 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:41991735+2799454
2024-06-09 18:16:58,433 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:16:58,449 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:16:58,559 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:16:59,384 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:16:59,384 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:16:59,386 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:16:59,386 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:16:59,386 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:16:59,408 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:16:59,489 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000002/./mapper.py]
...
```

```
...
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000003
user.name: yarn
************************************************************/
2024-06-09 18:16:53,926 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:16:56,438 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:16:56,438 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:16:56,439 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:16:56,547 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:16:57,525 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:33593388+2799449
2024-06-09 18:16:57,625 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:16:57,638 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:16:57,720 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:16:58,669 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:16:58,669 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:16:58,669 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:16:58,669 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:16:58,669 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:16:58,699 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:16:58,747 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000003/./mapper.py]
...
```

```
...
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000004
user.name: yarn
************************************************************/
2024-06-09 18:16:54,055 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:16:56,555 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:16:56,555 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:16:56,556 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:16:56,704 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:16:57,635 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:36392837+2799449
2024-06-09 18:16:57,675 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:16:57,691 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:16:57,773 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:16:58,637 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:16:58,638 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:16:58,638 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:16:58,638 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:16:58,638 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:16:58,676 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:16:58,742 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000004/./mapper.py]
...
```

```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000005
user.name: yarn
************************************************************/
2024-06-09 18:16:54,280 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:16:56,855 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:16:56,865 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:16:56,866 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:16:56,952 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:16:57,876 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:39192286+2799449
2024-06-09 18:16:58,007 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:16:58,023 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:16:58,116 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:16:58,909 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:16:58,909 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:16:58,909 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:16:58,909 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:16:58,909 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:16:58,930 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:16:59,024 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000005/./mapper.py]
```

```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000006
user.name: yarn
************************************************************/
2024-06-09 18:17:01,102 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:17:04,160 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:17:04,160 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:17:04,169 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:17:04,281 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:05,142 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:0+2799449
2024-06-09 18:17:05,228 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:17:05,237 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:17:05,394 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:17:06,140 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:17:06,140 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:17:06,141 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:17:06,141 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:17:06,141 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:17:06,220 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:17:06,259 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000006/./mapper.py]
```

```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000007
user.name: yarn
************************************************************/
2024-06-09 18:17:00,563 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:17:04,056 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:17:04,056 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:17:04,070 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:17:04,190 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:05,095 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:2799449+2799449
2024-06-09 18:17:05,230 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:17:05,254 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:17:05,356 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:17:06,091 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:17:06,091 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:17:06,091 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:17:06,091 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:17:06,091 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:17:06,116 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:17:06,219 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000007/./mapper.py]
```

```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000008
user.name: yarn
************************************************************/
2024-06-09 18:17:00,571 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:17:04,107 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:17:04,107 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:17:04,108 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:17:04,211 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:05,064 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:5598898+2799449
2024-06-09 18:17:05,206 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:17:05,238 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:17:05,367 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:17:06,098 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:17:06,098 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:17:06,098 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:17:06,098 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:17:06,098 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:17:06,112 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:17:06,183 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000008/./mapper.py]
```


```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000009
user.name: yarn
************************************************************/
2024-06-09 18:17:00,753 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:17:03,930 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:17:03,930 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:17:03,931 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:17:04,027 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:05,008 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:8398347+2799449
2024-06-09 18:17:05,165 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:17:05,179 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:17:05,306 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:17:06,169 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:17:06,169 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:17:06,169 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:17:06,169 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:17:06,169 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:17:06,258 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:17:06,302 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000009/./mapper.py]
```

```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000010
user.name: yarn
************************************************************/
2024-06-09 18:16:56,213 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:16:59,065 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:16:59,065 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:16:59,066 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:16:59,219 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:00,369 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:16796694+2799449
2024-06-09 18:17:00,555 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:17:00,570 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:17:00,867 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:17:02,478 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:17:02,478 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:17:02,478 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:17:02,478 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:17:02,478 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:17:02,517 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:17:02,638 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000010/./mapper.py]
```

```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000011
user.name: yarn
************************************************************/
2024-06-09 18:16:57,058 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:17:00,104 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:17:00,105 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:17:00,105 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:17:00,229 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:01,931 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:19596143+2799449
2024-06-09 18:17:02,127 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:17:02,157 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:17:02,402 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:17:03,549 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:17:03,550 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:17:03,555 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:17:03,555 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:17:03,555 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:17:03,616 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:17:03,714 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000011/./mapper.py]
```

```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000012
user.name: yarn
************************************************************/
2024-06-09 18:16:56,840 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:16:59,854 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:16:59,854 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:16:59,855 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:16:59,934 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:01,761 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:22395592+2799449
2024-06-09 18:17:02,029 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:17:02,088 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:17:02,351 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:17:04,038 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:17:04,038 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:17:04,039 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:17:04,039 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:17:04,039 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:17:04,078 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:17:04,202 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000012/./mapper.py]
```

```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000013
user.name: yarn
************************************************************/
2024-06-09 18:16:57,512 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:17:00,652 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:17:00,652 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:17:00,653 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:17:01,010 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:03,068 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:25195041+2799449
2024-06-09 18:17:03,289 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:17:03,332 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:17:03,589 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:17:05,621 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:17:05,621 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:17:05,622 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:17:05,622 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:17:05,622 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:17:05,657 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:17:05,784 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000013/./mapper.py]
```

```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000014
user.name: yarn
************************************************************/
2024-06-09 18:16:48,209 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:16:49,522 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:16:49,522 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:16:49,523 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:16:49,568 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:16:50,113 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:11197796+2799449
2024-06-09 18:16:50,146 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:16:50,174 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:16:50,228 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:16:50,760 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:16:50,760 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:16:50,760 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:16:50,760 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:16:50,760 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:16:50,787 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:16:50,838 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000014/./mapper.py]
```

```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000015
user.name: yarn
************************************************************/
2024-06-09 18:16:47,804 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:16:49,205 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:16:49,205 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:16:49,216 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:16:49,309 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:16:49,745 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:13997245+2799449
2024-06-09 18:16:49,812 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:16:49,819 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:16:49,869 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:16:50,153 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:16:50,153 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:16:50,153 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:16:50,153 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:16:50,153 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:16:50,180 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:16:50,251 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000015/./mapper.py]
```


```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000016
user.name: yarn
************************************************************/
2024-06-09 18:17:04,164 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:17:05,609 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:17:05,609 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:17:05,610 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:17:05,684 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:06,036 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:27994490+2799449
2024-06-09 18:17:06,085 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:17:06,094 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:17:06,154 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:17:06,545 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:17:06,546 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:17:06,546 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:17:06,546 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:17:06,546 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:17:06,561 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:17:06,602 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000016/./mapper.py]
```

```
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000017
user.name: yarn
************************************************************/
2024-06-09 18:17:03,998 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:17:05,365 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:17:05,365 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:17:05,366 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:17:05,423 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:05,838 INFO [main] org.apache.hadoop.mapred.MapTask: Processing split: hdfs://ip-172-31-53-255.ec2.internal:8020/input/nytimes.txt:30793939+2799449
2024-06-09 18:17:05,863 INFO [main] com.hadoop.compression.lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 18:17:05,867 INFO [main] com.hadoop.compression.lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 18:17:05,911 INFO [main] org.apache.hadoop.mapred.MapTask: numReduceTasks: 2
2024-06-09 18:17:06,242 INFO [main] org.apache.hadoop.mapred.MapTask: (EQUATOR) 0 kvi 52428796(209715184)
2024-06-09 18:17:06,242 INFO [main] org.apache.hadoop.mapred.MapTask: mapreduce.task.io.sort.mb: 200
2024-06-09 18:17:06,242 INFO [main] org.apache.hadoop.mapred.MapTask: soft limit at 167772160
2024-06-09 18:17:06,242 INFO [main] org.apache.hadoop.mapred.MapTask: bufstart = 0; bufvoid = 209715200
2024-06-09 18:17:06,242 INFO [main] org.apache.hadoop.mapred.MapTask: kvstart = 52428796; length = 13107200
2024-06-09 18:17:06,248 INFO [main] org.apache.hadoop.mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
2024-06-09 18:17:06,261 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000017/./mapper.py]
```



##### Reduce tasks

<img width="1420" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/cc22e45a-42c8-46ae-9763-7e119e09072a">

click the link in the second-to-last section

```
...
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000018
user.name: yarn
************************************************************/
2024-06-09 18:17:16,987 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:17:18,509 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:17:18,509 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:17:18,511 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:17:18,613 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:18,736 INFO [main] org.apache.hadoop.mapred.ReduceTask: Using ShuffleConsumerPlugin: org.apache.hadoop.mapreduce.task.reduce.Shuffle@25d2f66
2024-06-09 18:17:18,738 WARN [main] org.apache.hadoop.metrics2.impl.MetricsSystemImpl: ReduceTask metrics system already initialized!
2024-06-09 18:17:19,112 INFO [fetcher#3] org.apache.hadoop.io.compress.CodecPool: Got brand-new decompressor [.snappy]
2024-06-09 18:17:19,149 INFO [fetcher#1] org.apache.hadoop.io.compress.CodecPool: Got brand-new decompressor [.snappy]
2024-06-09 18:17:19,152 INFO [fetcher#4] org.apache.hadoop.io.compress.CodecPool: Got brand-new decompressor [.snappy]
2024-06-09 18:17:19,714 INFO [main] org.apache.hadoop.io.compress.CodecPool: Got brand-new compressor [.snappy]
2024-06-09 18:17:24,418 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000018/./reducer.py]
...
2024-06-09 18:17:36,487 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: Saved output of task 'attempt_1717955085543_0001_r_000000_0' to hdfs://ip-172-31-53-255.ec2.internal:8020/output_1
...
```

```
...
user.dir: /mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000019
user.name: yarn
************************************************************/
2024-06-09 18:17:19,197 INFO [main] org.apache.hadoop.conf.Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
2024-06-09 18:17:20,887 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: File Output Committer Algorithm version is 2
2024-06-09 18:17:20,888 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: FileOutputCommitter skip cleanup _temporary folders under output directory:false, ignore cleanup failures: false
2024-06-09 18:17:20,888 INFO [main] org.apache.hadoop.mapreduce.lib.output.DirectFileOutputCommitter: Direct Write: DISABLED
2024-06-09 18:17:20,941 INFO [main] org.apache.hadoop.mapred.Task:  Using ResourceCalculatorProcessTree : [ ]
2024-06-09 18:17:21,029 INFO [main] org.apache.hadoop.mapred.ReduceTask: Using ShuffleConsumerPlugin: org.apache.hadoop.mapreduce.task.reduce.Shuffle@25d2f66
2024-06-09 18:17:21,031 WARN [main] org.apache.hadoop.metrics2.impl.MetricsSystemImpl: ReduceTask metrics system already initialized!
2024-06-09 18:17:21,269 INFO [fetcher#1] org.apache.hadoop.io.compress.CodecPool: Got brand-new decompressor [.snappy]
2024-06-09 18:17:21,295 INFO [fetcher#2] org.apache.hadoop.io.compress.CodecPool: Got brand-new decompressor [.snappy]
2024-06-09 18:17:21,297 INFO [fetcher#3] org.apache.hadoop.io.compress.CodecPool: Got brand-new decompressor [.snappy]
2024-06-09 18:17:21,299 INFO [fetcher#4] org.apache.hadoop.io.compress.CodecPool: Got brand-new decompressor [.snappy]
2024-06-09 18:17:21,638 INFO [main] org.apache.hadoop.io.compress.CodecPool: Got brand-new compressor [.snappy]
2024-06-09 18:17:23,919 INFO [main] org.apache.hadoop.streaming.PipeMapRed: PipeMapRed exec [/mnt/yarn/usercache/hadoop/appcache/application_1717955085543_0001/container_1717955085543_0001_01_000019/./reducer.py]
...
2024-06-09 18:17:34,121 INFO [main] org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter: Saved output of task 'attempt_1717955085543_0001_r_000001_0' to hdfs://ip-172-31-53-255.ec2.internal:8020/output_1
...
```



Notiong displayed for container 20

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/f92ea347-508c-4a68-bf08-a1ad4abc7c26">


----
drop???



|file name | file size|
|------|-----------|
|nytimes.txt | 42.72M|
|pg31156.txt |1.12M |
|pg34751.txt | 1.17M|
|pg35236.txt |1.04M|

| Instance | file | block | 
|------|-----------|--------|
| ip-xxxx-58-44.xxxx| nytimes.txt  | block 0 |
| ip-xxxx-58-44.xxxx| nytimes.txt  | block 1 |
| ip-xxxx-58-44.xxxx| nytimes.txt  | block 2 |
| ip-xxxx-58-44.xxxx| pg31156.txt  | block 0 |
| ip-xxxx-58-44.xxxx| pg34751.txt | block 0 |
| ip-xxxx-58-44.xxxx| pg35236.txt   | block 0 |
| ip-xxxx-61-223.xxxx| nytimes.txt  | block 0 |
| ip-xxxx-61-223.xxxx | nytimes.txt  | block 1 |
| ip-xxxx-61-223.xxxx | pg31156.txt   | block 0 |
| ip-xxxx-61-223.xxxx | pg34751.txt  | block 0 |
| ip-xxxx-61-223.xxxx | pg35236.txt  | block 0 |
| ip-xxxx-62-71.xxxx| nytimes.txt   | block 0 |
| ip-xxxx-62-71.xxxx| nytimes.txt   | block 1 |
| ip-xxxx-62-71.xxxx| nytimes.txt   | block 2 |
| ip-xxxx-62-71.xxxx| pg35236.txt   | block 0 |
| ip-xxxx-63-3.xxxx | nytimes.txt   | block 2 |
| ip-xxxx-63-3.xxxx | pg31156.txt | block 0 |
| ip-xxxx-63-3.xxxx | pg34751.txt | block 0 |


<br>

Submit a job for word counting:
 
```shell
[hadoop@ip-xxxx ~]$ mapred streaming -D mapreduce.job.reduces=2 -files mapper.py,reducer.py -input /input -output /output_1 -mapper mapper.py -reducer reducer.py
packageJobJar: [] [/usr/lib/hadoop/hadoop-streaming-3.3.6-amzn-3.jar] /tmp/streamjob17256461406247448522.jar tmpDir=null
2024-06-09 17:17:15,274 INFO client.DefaultNoHARMFailoverProxyProvider: Connecting to ResourceManager at ip-172-31-59-82.ec2.internal/172.31.59.82:8032
2024-06-09 17:17:15,461 INFO client.AHSProxy: Connecting to Application History server at ip-172-31-59-82.ec2.internal/172.31.59.82:10200
2024-06-09 17:17:15,514 INFO client.DefaultNoHARMFailoverProxyProvider: Connecting to ResourceManager at ip-172-31-59-82.ec2.internal/172.31.59.82:8032
2024-06-09 17:17:15,516 INFO client.AHSProxy: Connecting to Application History server at ip-172-31-59-82.ec2.internal/172.31.59.82:10200
2024-06-09 17:17:15,787 INFO mapreduce.JobResourceUploader: Disabling Erasure Coding for path: /tmp/hadoop-yarn/staging/hadoop/.staging/job_1717949687705_0003
2024-06-09 17:17:16,195 INFO lzo.GPLNativeCodeLoader: Loaded native gpl library
2024-06-09 17:17:16,197 INFO lzo.LzoCodec: Successfully loaded & initialized native-lzo library [hadoop-lzo rev 049362b7cf53ff5f739d6b1532457f2c6cd495e8]
2024-06-09 17:17:16,244 INFO mapred.FileInputFormat: Total input files to process : 4
2024-06-09 17:17:16,252 INFO net.NetworkTopology: Adding a new node: /default-rack/172.31.61.223:9866
2024-06-09 17:17:16,253 INFO net.NetworkTopology: Adding a new node: /default-rack/172.31.62.71:9866
2024-06-09 17:17:16,253 INFO net.NetworkTopology: Adding a new node: /default-rack/172.31.58.44:9866
2024-06-09 17:17:16,254 INFO net.NetworkTopology: Adding a new node: /default-rack/172.31.63.3:9866
2024-06-09 17:17:16,344 INFO mapreduce.JobSubmitter: number of splits:18
2024-06-09 17:17:16,652 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1717949687705_0003
2024-06-09 17:17:16,653 INFO mapreduce.JobSubmitter: Executing with tokens: []
2024-06-09 17:17:16,855 INFO conf.Configuration: resource-types.xml not found
2024-06-09 17:17:16,857 INFO resource.ResourceUtils: Unable to find 'resource-types.xml'.
2024-06-09 17:17:16,958 INFO impl.YarnClientImpl: Submitted application application_1717949687705_0003
2024-06-09 17:17:17,011 INFO mapreduce.Job: The url to track the job: http://ip-172-31-59-82.ec2.internal:20888/proxy/application_1717949687705_0003/
2024-06-09 17:17:17,013 INFO mapreduce.Job: Running job: job_1717949687705_0003
2024-06-09 17:17:26,191 INFO mapreduce.Job: Job job_1717949687705_0003 running in uber mode : false
2024-06-09 17:17:26,193 INFO mapreduce.Job:  map 0% reduce 0%
2024-06-09 17:17:43,385 INFO mapreduce.Job:  map 6% reduce 0%
2024-06-09 17:17:44,393 INFO mapreduce.Job:  map 11% reduce 0%
2024-06-09 17:17:55,510 INFO mapreduce.Job:  map 33% reduce 0%
2024-06-09 17:17:56,521 INFO mapreduce.Job:  map 44% reduce 0%
2024-06-09 17:17:57,529 INFO mapreduce.Job:  map 56% reduce 0%
2024-06-09 17:18:00,549 INFO mapreduce.Job:  map 67% reduce 0%
2024-06-09 17:18:02,563 INFO mapreduce.Job:  map 89% reduce 0%
2024-06-09 17:18:11,615 INFO mapreduce.Job:  map 100% reduce 0%
2024-06-09 17:18:14,633 INFO mapreduce.Job:  map 100% reduce 37%
2024-06-09 17:18:17,651 INFO mapreduce.Job:  map 100% reduce 50%
2024-06-09 17:18:20,666 INFO mapreduce.Job:  map 100% reduce 100%
2024-06-09 17:18:20,676 INFO mapreduce.Job: Job job_1717949687705_0003 completed successfully
2024-06-09 17:18:20,768 INFO mapreduce.Job: Counters: 56
	File System Counters
		FILE: Number of bytes read=5070336
		FILE: Number of bytes written=19340990
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=49102225
		HDFS: Number of bytes written=2196033
		HDFS: Number of read operations=64
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=4
		HDFS: Number of bytes read erasure-coded=0
	Job Counters 
		Killed map tasks=1
		Killed reduce tasks=1
		Launched map tasks=18
		Launched reduce tasks=2
		Data-local map tasks=18
		Total time spent by all maps in occupied slots (ms)=21342432
		Total time spent by all reduces in occupied slots (ms)=3981696
		Total time spent by all map tasks (ms)=444634
		Total time spent by all reduce tasks (ms)=41476
		Total vcore-milliseconds taken by all map tasks=444634
		Total vcore-milliseconds taken by all reduce tasks=41476
		Total megabyte-milliseconds taken by all map tasks=682957824
		Total megabyte-milliseconds taken by all reduce tasks=127414272
	Map-Reduce Framework
		Map input records=249985
		Map output records=7671019
		Map output bytes=60402766
		Map output materialized bytes=8386812
		Input split bytes=1980
		Combine input records=0
		Combine output records=0
		Reduce input groups=143068
		Reduce shuffle bytes=8386812
		Reduce input records=7671019
		Reduce output records=143068
		Spilled Records=15342038
		Shuffled Maps =36
		Failed Shuffles=0
		Merged Map outputs=36
		GC time elapsed (ms)=1569
		CPU time spent (ms)=90100
		Physical memory (bytes) snapshot=9702985728
		Virtual memory (bytes) snapshot=66503630848
		Total committed heap usage (bytes)=9817817088
		Peak Map Physical memory (bytes)=532000768
		Peak Map Virtual memory (bytes)=3192672256
		Peak Reduce Physical memory (bytes)=352796672
		Peak Reduce Virtual memory (bytes)=4779577344
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters 
		Bytes Read=49100245
	File Output Format Counters 
		Bytes Written=2196033
2024-06-09 17:18:20,769 INFO streaming.StreamJob: Output directory: /output_1
```


Job ID: job_1717949687705_0003
No. of input splits: 18


#### MapReduce jobhistory Web UI

http://<primary_node_DNS>:19888/jobhistory/job/<job_id>

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/7b51091d-100c-4e58-bfab-4ebf0da71bf0">


18 Map tasks:

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/15623e6c-766c-4b46-a2a3-7023b712fd43">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/91c59479-6fe7-4b19-9184-66bd3abe984b">


<img width="1432" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/fd276f59-7741-4985-b2bf-86cbbea0548d">


| Instance | Map tasks | 
|------|-----------|--------|
| ip-xxxx-58-44.xxxx| 2, 3, 4, 5|  
| ip-xxxx-61-223.xxxx | 6, 7, 8, 9|
| ip-xxxx-62-71.xxxx| 0, 1, 10   | 
| ip-xxxx-63-3.xxxx | 11, 12, 13, 14, 15, 16|

| ip-xxxx-58-44.xxxx| nytimes.txt  | block 0 |
| ip-xxxx-58-44.xxxx| nytimes.txt  | block 1 |
| ip-xxxx-58-44.xxxx| nytimes.txt  | block 2 |
| ip-xxxx-58-44.xxxx| pg31156.txt  | block 0 |
| ip-xxxx-58-44.xxxx| pg34751.txt | block 0 |
| ip-xxxx-58-44.xxxx| pg35236.txt   | block 0 |
| ip-xxxx-61-223.xxxx| nytimes.txt  | block 0 |
| ip-xxxx-61-223.xxxx | nytimes.txt  | block 1 |
| ip-xxxx-61-223.xxxx | pg31156.txt   | block 0 |
| ip-xxxx-61-223.xxxx | pg34751.txt  | block 0 |
| ip-xxxx-61-223.xxxx | pg35236.txt  | block 0 |
| ip-xxxx-62-71.xxxx| nytimes.txt   | block 0 |
| ip-xxxx-62-71.xxxx| nytimes.txt   | block 1 |
| ip-xxxx-62-71.xxxx| nytimes.txt   | block 2 |
| ip-xxxx-62-71.xxxx| pg35236.txt   | block 0 |
| ip-xxxx-63-3.xxxx | nytimes.txt   | block 2 |
| ip-xxxx-63-3.xxxx | pg31156.txt | block 0 |
| ip-xxxx-63-3.xxxx | pg34751.txt | block 0 |
