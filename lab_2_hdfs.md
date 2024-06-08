
# EMR settings

- EMR release: 7.1.0
  
- 1 primary instance; type: `m4.large`

- 3 core instances; type: `m4.large`
  
    <img width="300" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/1644cc8c-d79b-4c48-a194-f5c49478d126">

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
                "dfs.block.size": "16M",
                "dfs.replication": "3"
            }
        }
    ]
    ```
    -  Override the `"hadoop.http.staticuser.user"` property's default value (`"dr.who"`) with `"hadoop"` (the default user of EMR instances.) if you want to use the NameNode's Web UI to delete files and create directories.
    
    - Set `"dfs.webhdfs.enabled"` to `"true"` to use WebHDFS if you want to upload files from a local computer. Check out [this manual](webhdfs_emr.md) for more details.

- Make sure the primary node's EC2 security group has a rule allowing for "ALL TCP" from "My IP" and a rule allowing for "SSH" from "Anywhere".

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

You can change all `<Your ITSC Account>` placeholders below to your ITSC account string first. Later, you can just copy and paste the commands to the terminal for execution.

Note that `hadoop fs` and `hdfs dfs` can be interchangeably used below.

```shell
$ hadoop fs -ls /

$ hadoop fs -mkdir -p /<Your ITSC Account>

$ hdfs dfs -ls /

$ hadoop fs -put data /<Your ITSC Account>

$ hadoop fs -ls /<Your ITSC Account>/data

$ hadoop fs -rm /<Your ITSC Account>/data/flights.csv

$ hadoop fs -ls /<Your ITSC Account>/data

$ hadoop fs -cat /<Your ITSC Account>/data/*.txt | tail -n50

$ hadoop fs -get /<Your ITSC Account>/data <A Directory in Local FS> 

$ hadoop fs -setrep -w 2 /<Your ITSC Account>/pg31156.txt
```
