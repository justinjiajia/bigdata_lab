
# 1 Data Preparation

```shell
wget -O transactions.txt  https://raw.githubusercontent.com/justinjiajia/datafiles/main/browsing.csv

# prepare data on HDFS
hadoop fs -mkdir /input

hadoop fs -put transactions.txt /input
```

<br>

# 2 Run PySpark locally via shell

## Start the shell

```shell
pyspark --master local[*]
```

## PySpark code to run sequentially

```python
transactions = sc.textFile("file:///home/hadoop/transactions.txt")  # absolute path of the input file on local FS

transactions.cache()

transactions.take(10)

from itertools import combinations

pairs = transactions.map(lambda x: x.strip().split(" ")).flatMap(lambda x: combinations(x, 2)).map(lambda x: (x[0], x[1]) if x[0] <= x[1] else (x[1], x[0]))

pairs_count = pairs.map(lambda x: (x, 1)).reduceByKey(lambda x, y: x+y)

rev_pairs_count = pairs_count.map(lambda x: ((x[0][1], x[0][0]), x[1]))

combined_pairs = pairs_count.union(rev_pairs_count).map(lambda x: (x[0][0], [(x[0][1], x[1])]))

rec_pairs_ordered = combined_pairs.reduceByKey(lambda x, y: sorted(x+y, key=lambda val: val[1], reverse=True)[:5])

rec_pairs_ordered.saveAsTextFile("file:///home/hadoop/output")   # absolute path of the output directory file on local FS

quit()
```

## Print the output

```shell
cat output/part-* | grep "ELE96863"
```

<br>

# 3 Run PySpark on Yarn via shell

## Start the shell

```shell
pyspark --master yarn --deploy-mode client
```

## PySpark code to run sequentially

```python
transactions = sc.textFile("hdfs:///input/transactions.txt")  # absolute path of the input file on HDFS

transactions.cache()

transactions.take(10)

from itertools import combinations

pairs = transactions.map(lambda x: x.strip().split(" ")).flatMap(lambda x: combinations(x, 2)).map(lambda x: (x[0], x[1]) if x[0] <= x[1] else (x[1], x[0]))

pairs_count = pairs.map(lambda x: (x, 1)).reduceByKey(lambda x, y: x+y)

rev_pairs_count = pairs_count.map(lambda x: ((x[0][1], x[0][0]), x[1]))

combined_pairs = pairs_count.union(rev_pairs_count).map(lambda x: (x[0][0], [(x[0][1], x[1])]))

rec_pairs_ordered = combined_pairs.reduceByKey(lambda x, y: sorted(x+y, key=lambda val: val[1], reverse=True)[:5])

rec_pairs_ordered.saveAsTextFile("hdfs:///output")  # absolute path of the output directory file on HDFS

quit()
```
## List all hdfs output files from within the shell

```python
hadoop = sc._jvm.org.apache.hadoop
fs = hadoop.fs.FileSystem
conf = hadoop.conf.Configuration()
path = hadoop.fs.Path('/output')
[str(f.getPath()) for f in fs.get(conf).listStatus(path)]
```

## Print the output

```shell
hadoop fs -cat output/part-* | grep "ELE96863"
```

<br>

# 4 Launch PySpark applications with `spark-submit`

## Create a PySpark script file

```shell
nano recommendation.py
```

## Code to paste into the file

```python
from pyspark.sql import SparkSession
from itertools import combinations

spark = SparkSession.builder.appName("recommendation").getOrCreate()
sc = spark.sparkContext

transactions = sc.textFile("hdfs:///input/transactions.txt") 
transactions.cache()
pairs = transactions.map(lambda x: x.strip().split(" ")).flatMap(lambda x: combinations(x, 2)).map(lambda x: (x[0], x[1]) if x[0] <= x[1] else (x[1], x[0]))
pairs_count = pairs.map(lambda x: (x, 1)).reduceByKey(lambda x, y: x+y)
rev_pairs_count = pairs_count.map(lambda x: ((x[0][1], x[0][0]), x[1]))
combined_pairs = pairs_count.union(rev_pairs_count).map(lambda x: (x[0][0], [(x[0][1], x[1])]))
rec_pairs_ordered = combined_pairs.reduceByKey(lambda x, y: sorted(x+y, key=lambda val: val[1], reverse=True)[:5])
rec_pairs_ordered.saveAsTextFile("hdfs:///output")
```

## Submit the job

```shell
spark-submit --master yarn --deploy-mode client --num-executors 4 recommendation.py
```
> Spark properties can be configured separately for each application. 
> Any values specified as flags or in the properties file will be passed on to the application and merged with those specified through [`SparkConf`](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.SparkConf.html). Properties set directly on the `SparkConf` take highest precedence, then flags passed to `spark-submit` or `spark-shell`, then options in the *spark-defaults.conf* file.

Check out this <a href="https://spark.apache.org/docs/latest/submitting-applications" target="_blank">page</a> for more launch options 

## Print the output

```shell
hadoop fs -cat output/part-* | grep "ELE96863"
```
