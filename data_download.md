
<br>

### from S3

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AmazonS3.html

```shell
$ aws s3 cp s3://my_bucket/my_folder/my_file.ext my_copied_file.ext
```

E.g.
```
[hadoop@xxxx ~]$ aws s3 cp s3://ust-bigdata-class/install_python_libraries.sh a.sh
download: s3://ust-bigdata-class/install_python_libraries.sh to ./a.sh
[hadoop@xxxx ~]$ ls
a.sh
$ nano a.sh 
[hadoop@xxxx ~]$ aws s3 cp a.sh s3://ust-bigdata-class/install_python_libraries.sh
upload failed: ./a.sh to s3://ust-bigdata-class/install_python_libraries.sh An error occurred (AccessDenied) when calling the PutObject operation: Access Denied
```

We may need to configure this s3 file to allow for file writes.

<br>

### from HuggingFace

Choose **Files and versions** tab

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/a6928049-4a43-48d9-a307-75280cb9c5d4">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/54ddecef-cf4d-4818-b952-f48d17b970c6">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/5328862c-1643-48cb-8be7-a33cd820ec95">

Find its URL adress and download it with `wget`:

```shell
[hadoop@xxxx ~]$ wget https://huggingface.co/datasets/legacy-datasets/wikipedia/resolve/main/data/20220301.simple/train-00000-of-00001.parquet
[hadoop@xxxx ~]$ parquet-tools inspect train-00000-of-00001.parquet 

############ file meta data ############
created_by: parquet-cpp-arrow version 10.0.1
num_columns: 4
num_rows: 205328
num_row_groups: 206
format_version: 2.6
serialized_size: 407414


############ Columns ############
id
url
title
text

############ Column(id) ############
name: id
path: id
max_definition_level: 1
max_repetition_level: 0
physical_type: BYTE_ARRAY
logical_type: String
converted_type (legacy): UTF8
compression: SNAPPY (space_saved: 37%)

############ Column(url) ############
name: url
path: url
max_definition_level: 1
max_repetition_level: 0
physical_type: BYTE_ARRAY
logical_type: String
converted_type (legacy): UTF8
compression: SNAPPY (space_saved: 74%)

############ Column(title) ############
name: title
path: title
max_definition_level: 1
max_repetition_level: 0
physical_type: BYTE_ARRAY
logical_type: String
converted_type (legacy): UTF8
compression: SNAPPY (space_saved: 25%)

############ Column(text) ############
name: text
path: text
max_definition_level: 1
max_repetition_level: 0
physical_type: BYTE_ARRAY
logical_type: String
converted_type (legacy): UTF8
compression: SNAPPY (space_saved: 42%)

[hadoop@xxxx ~]$ parq train-00000-of-00001.parquet 

 # Metadata 
 <pyarrow._parquet.FileMetaData object at 0x7f36063c3bd0>
  created_by: parquet-cpp-arrow version 10.0.1
  num_columns: 4
  num_rows: 205328
  num_row_groups: 206
  format_version: 2.6
  serialized_size: 407414

[hadoop@xxxx ~]$ parq train-00000-of-00001.parquet --head 5
   id                                                url  \
0   1            https://simple.wikipedia.org/wiki/April   
1   2           https://simple.wikipedia.org/wiki/August   
2   6              https://simple.wikipedia.org/wiki/Art   
3   8                https://simple.wikipedia.org/wiki/A   
4   9              https://simple.wikipedia.org/wiki/Air    

                             title  \
0                            April   
1                           August   
2                              Art   
3                                A   
4                              Air   

                                                text  
0  April is the fourth month of the year in the J...  
1  August (Aug.) is the eighth month of the year ...  
2  Art is a creative activity that expresses imag...  
3  A or a is the first letter of the English alph...  
4  Air refers to the Earth's atmosphere. Air is a...

[hadoop@xxxx ~]$ parquet-tools show -h
usage: parquet-tools show [-h] [--format {psql,github}] [--columns COLUMNS] [--head HEAD] [--awsprofile AWSPROFILE] [--endpoint-url ENDPOINT_URL]
                          FILE [FILE ...]

Show parquet file content with human readability.

positional arguments:
  FILE                  The parquet file to print to stdout. e.g. ./target.parquet or s3://bucket-name/target.parquet or s3://bucket-name/*

optional arguments:
  -h, --help            show this help message and exit
  --format {psql,github}, -f {psql,github}
                        Table format(default: psql).
  --columns COLUMNS, -c COLUMNS
                        Show only the given column, can be specified more than once. e.g. --columns email,name
  --head HEAD, -n HEAD  Show only head record(default:infinity)
  --awsprofile AWSPROFILE
                        awscli profile in ~/.aws/credentials. You use this option when you read parquet file on s3.
  --endpoint-url ENDPOINT_URL
                        A custom S3 endpoint URL

[hadoop@xxxx ~]$ parquet-tools show -c title,url -n 5 train-00000-of-00001.parquet 
+---------+------------------------------------------+
| title   | url                                      |
|---------+------------------------------------------|
| April   | https://simple.wikipedia.org/wiki/April  |
| August  | https://simple.wikipedia.org/wiki/August |
| Art     | https://simple.wikipedia.org/wiki/Art    |
| A       | https://simple.wikipedia.org/wiki/A      |
| Air     | https://simple.wikipedia.org/wiki/Air    |
+---------+------------------------------------------+

[hadoop@xxxx ~]$ hadoop fs -mkdir /input
[hadoop@xxxx ~]$ hadoop fs -put train-00000-of-00001.parquet /input

```

https://pypi.org/project/parquet-tools/
https://github.com/chhantyal/parquet-cli


```shell
[hadoop@xxxx ~]$ pyspark --master local[*]
```

```python
>>> df = spark.read.format("parquet").load("file:///home/hadoop/train-00000-of-00001.parquet")
>>> df.schema
StructType([StructField('id', StringType(), True), StructField('url', StringType(), True), StructField('title', StringType(), True), StructField('text', StringType(), True)])
>>> df.select("url", "title").show(10)
+--------------------+--------------------+                                     
|                 url|               title|
+--------------------+--------------------+
|https://simple.wi...|               April|
|https://simple.wi...|              August|
|https://simple.wi...|                 Art|
|https://simple.wi...|                   A|
|https://simple.wi...|                 Air|
|https://simple.wi...|Autonomous commun...|
|https://simple.wi...|         Alan Turing|
|https://simple.wi...|   Alanis Morissette|
|https://simple.wi...|   Adobe Illustrator|
|https://simple.wi...|           Andouille|
+--------------------+--------------------+
only showing top 10 rows

>>> rdd = df.select("url", "title").rdd
>>> rdd.take(10)
[Row(url='https://simple.wikipedia.org/wiki/April', title='April'), Row(url='https://simple.wikipedia.org/wiki/August', title='August'), Row(url='https://simple.wikipedia.org/wiki/Art', title='Art'), Row(url='https://simple.wikipedia.org/wiki/A', title='A'), Row(url='https://simple.wikipedia.org/wiki/Air', title='Air'), Row(url='https://simple.wikipedia.org/wiki/Autonomous%20communities%20of%20Spain', title='Autonomous communities of Spain'), Row(url='https://simple.wikipedia.org/wiki/Alan%20Turing', title='Alan Turing'), Row(url='https://simple.wikipedia.org/wiki/Alanis%20Morissette', title='Alanis Morissette'), Row(url='https://simple.wikipedia.org/wiki/Adobe%20Illustrator', title='Adobe Illustrator'), Row(url='https://simple.wikipedia.org/wiki/Andouille', title='Andouille')]
```
