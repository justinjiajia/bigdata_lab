
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

Instead of running them one after another, we can also put them into a `.script` file and run them in batch.

```shell
$ nano data_prep.sh
```


Copy and paste the code snippet below into the file:

```shell
#!/bin/bash

rm -r data
mkdir data
cd data
wget https://archive.org/download/encyclopaediabri31156gut/pg31156.txt
wget https://archive.org/download/encyclopaediabri34751gut/pg34751.txt
wget https://archive.org/download/encyclopaediabri35236gut/pg35236.txt
wget -O nytimes.txt https://raw.githubusercontent.com/justinjiajia/datafiles/main/nytimes_news_articles.txt
cd ..
du -sh data
```

Save the change and get back to the shell. Then run:

```shell
bash data_prep.sh
```
or 

```shell
sh data_prep.sh
```

There is a third way: first add the execution permission to the file to make it executable; then run it by typing `./` followed by the script name in the terminal (other):

```shell
$ chmod +x data_prep.sh
$ ./data_prep.sh
```
> By default, the current directory (represented by `"."`) is not included in the `$PATH` variable to prevents the accidental execution of programs in the current directory, which could be malicious or unintended. The `"./"` syntax explicitly tells the shell (e.g., bash, zsh) that you want to execute a file in the current directory, rather than searching for a command or program with the same name in the directories listed in `$PATH`.

Check out [this page](https://www.geeksforgeeks.org/how-to-run-bash-script-in-linux/) for more details on how to run bash scripts.


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

<br>
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

<br>

# Get the output

```shell
hadoop fs -cat /<Your ITSC Account>/program_output/part-r-* > combinedresult.txt
head -n20 combinedresult.txt
tail -n20 combinedresult.txt
```


