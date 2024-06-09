

 <img width="800" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/f2ecfbcd-5191-47e8-8b62-ced4e2b19eef">

|file name | file size|
|------|-----------|
|nytimes.txt | 42.72M|
|pg31156.txt |1.12M |
|pg34751.txt | 1.17M|
|pg35236.txt |1.04M|

| Instance | file | block | 
|------|-----------|--------|
| ip-xxxx-58-44.xxxx| nytimes.txt   | block 0 |
| ip-xxxx-58-44.xxxx| nytimes.txt   | block 1 |
| ip-xxxx-58-44.xxxx| nytimes.txt   | block 2 |
| ip-xxxx-58-44.xxxx| pg31156.txt   | block 0 |
| ip-xxxx-58-44.xxxx| pg34751.txt  | block 0 |
| ip-xxxx-58-44.xxxx| pg35236.txt   | block 0 |
| ip-xxxx-61-223.xxxx| nytimes.txt   | block 0 |
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

<img width="1392" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/d9fde575-730d-4b62-a321-ac99b5770253">


