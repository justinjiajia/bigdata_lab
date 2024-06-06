
# Local file system operations for data preparation


```shell
$ mkdir data

$ cd data

$ wget https://archive.org/download/encyclopaediabri31156gut/pg31156.txt

$ wget https://archive.org/download/encyclopaediabri34751gut/pg34751.txt

$ wget https://archive.org/download/encyclopaediabri35236gut/pg35236.txt

$ wget -O nytimes.txt https://raw.githubusercontent.com/justinjiajia/datafiles/main/nytimes_news_articles.txt

$ cd ..

$ du -sh data
```

<br>

# HDFS operations for data preparation

You can change all `<Your ITSC Account>` placeholders below to your ITSC account string first. 
Later, you can just copy and paste the commands to the terminal for execution

Note that `hadoop fs` and `hdfs dfs` can be interchangeably used below.
 
```shell
$ hadoop fs -ls /

$ hadoop fs -mkdir -p /<Your ITSC Account>

$ hadoop fs -ls /

$ hadoop fs -put data /<Your ITSC Account>

$ hadoop fs -ls /<Your ITSC Account>/data
```


# MapReduce job submission


Note that `hadoop jar` and `yarn jar` can be interchangeably used below.

```shell
$ hadoop jar /usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar wordcount /<Your ITSC Account>/data /<Your ITSC Account>/wordcount_output
```

We can use the `-D` flag to define a value for a property in the format of `property=value`.
E.g., we can specify the number of reducers to use as follows:

```shell
$ yarn jar /usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar wordcount -D mapreduce.job.reduces=2  /<Your ITSC Account>/data /<Your ITSC Account>/wordcount_output_1
```

# Get the output

```shell
hadoop fs -cat /<Your ITSC Account>/program_output/part-r-* > combinedresult.txt
head -n20 combinedresult.txt
tail -n20 combinedresult.txt
```


