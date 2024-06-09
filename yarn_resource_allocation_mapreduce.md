
- 1 primary; 4 core

```shell
wget -O nytimes.txt https://raw.githubusercontent.com/justinjiajia/datafiles/main/nytimes_news_articles.txt
hadoop fs -mkdir /input
hadoop fs -put  nytimes.txt /input
```

<img width="1178" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/e62eaa0f-0755-40d5-864a-c34da7df2951">

| Instance | file | block | 
|------|-----------|--------|
| ip-xxxx-52-142.xxxx| block 1 |
| ip-xxxx-52-162.xxxx| block 0 |
| ip-xxxx-52-162.xxxx| block 2 |
| ip-xxxx-57-35.xxxx   | block 0 |
| ip-xxxx-57-35.xxxx  | block 1 |
| ip-xxxx-63-90.xxxx  | block 2 |
 

```shell
[hadoop@ip-172-31-53-255 ~]$ mapred streaming -D mapreduce.job.reduces=2 -files mapper.py,reducer.py -input /input -output /output_1 -mapper mapper.py -reducer reducer.py
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

http://<primary_node_DNS>:19888/jobhistory/job/<job_id>

ec2-52-72-29-130.compute-1.amazonaws.com
http://ec2-52-72-29-130.compute-1.amazonaws.com:19888/jobhistory/job/job_1717955085543_0001

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/c3a9cf55-44d4-4991-9e3d-89b31e37eea3">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/c94ba8a6-19a7-4268-9d31-639eebbffcda">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/4e65d3c2-5bb0-4261-8e64-2fb3b39d35c6">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/d32e0bd8-658a-414f-8061-07970b84f0e6">


<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/6bcf33a9-d5d0-4996-92e5-0fa8e27bf94f">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/7f2d0bbb-e739-4bfa-8098-993fc835d892">


| Instance | Map tasks |  Reduce tasks | ApplicationMaster|
|------|-----------|--------|--------|
| ip-xxxx-52-142.xxxx|  7, 8, 9, 10 | | |
| ip-xxxx-52-162.xxxx| 5, 6, 11, 12| | yes |
| ip-xxxx-57-35.xxxx  | 1, 2, 3, 4 | | |
| ip-xxxx-63-90.xxxx  | 0, 13,  14, 15| 0, 1| |


<img width="1416" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/34ab5b87-303e-455a-96e2-9860ee9d8781">
<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/de45b913-6ac0-48cf-84aa-ca85d4281fc5">


<img width="1432" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/62ea968e-9321-4041-99fd-588a27a94068">


On YARN timeline server UI,

<img width="1417" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/86ba2a8c-0660-4bed-aea9-815c1cc85045">
<img width="1413" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/767506e5-0fdd-4c1a-8357-70ad6f916b05">
<img width="1427" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/89028599-19d2-4fa2-a3d4-9a6aed37d915">
<img width="1428" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/b177ccf8-db48-4115-8537-22357db5485b">

container 1 (3072 Memory, 1 VCores; Priority: 0) is used to host the application master.

| Instance | Containers |
|------|-----------|
| ip-xxxx-52-142.xxxx| 10, 11, 12, 13 |  
| ip-xxxx-52-162.xxxx| 14, 15, 16, 17 |  
| ip-xxxx-57-35.xxxx  | 6, 7, 8, 9   |
| ip-xxxx-63-90.xxxx  | 2, 3, 4, 5 , 18, 19, 20|

containers 2-17: 1536 Memory, 1 VCores each; Priority: 20
containers 18-20: 3072 Memory, 1 VCores each; Priority:	10

The Diagnostics field of container 20: *Diagnostics: Container released by application*. The same field of the other containers is all empty.


<img width="1431" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/52882800-bd6d-4134-b681-26403ad5d2f0">

<img width="1430" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/04e02f4e-a849-46b5-bc25-c71a3f7f7960">


<img width="1429" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/efacec64-a94d-4c89-840e-4f7d9d47969e">


 <img width="800" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/f2ecfbcd-5191-47e8-8b62-ced4e2b19eef">

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
