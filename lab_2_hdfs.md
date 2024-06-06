

# EMR software configuration 

```json
[
    {
        "classification": "hdfs-site",
        "properties": {
            "dfs.block.size": "16M",
            "dfs.replication": "3"
        }
    }
]
```

<br>

# Check HDFS daemon processes on EMR

```shell
systemctl --type=service | grep -i hadoop
```

<br>

# Local file system operations for data preparation

```shell
mkdir data

cd data

wget https://archive.org/download/encyclopaediabri31156gut/pg31156.txt

wget https://archive.org/download/encyclopaediabri34751gut/pg34751.txt

wget https://archive.org/download/encyclopaediabri35236gut/pg35236.txt

wget -O nytimes.txt https://raw.githubusercontent.com/justinjiajia/datafiles/main/nytimes_news_articles.txt

wget -O flights.csv https://raw.githubusercontent.com/justinjiajia/datafiles/main/International_Report_Passengers.csv

cd ..

du -sh data
```

<br>

# HDFS operations

# you can first change all <Your ITSC Account> placeholders below to your ITSC account string
# so that later on you can just copy and paste the commands to the terminal for execution

```shell
$ hadoop fs -ls /

$ hadoop fs -mkdir -p /<Your ITSC Account>

$ hadoop fs -ls /

$ hadoop fs -put data /<Your ITSC Account>

$ hadoop fs -ls /<Your ITSC Account>/data

$ hadoop fs -rm /<Your ITSC Account>/data/flights.csv

$ hadoop fs -ls /<Your ITSC Account>/data

$ hadoop fs -cat /<Your ITSC Account>/data/*.txt | tail -n50

$ hadoop fs -get /<Your ITSC Account>/data <A Directory in Local FS> 

```
$ hadoop fs -setrep -w 2 /<Your ITSC Account>/pg31156.txt
