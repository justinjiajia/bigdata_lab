

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


### from HuggingFace

Choose **Files and versions** tab

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/a6928049-4a43-48d9-a307-75280cb9c5d4">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/54ddecef-cf4d-4818-b952-f48d17b970c6">

<img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/5328862c-1643-48cb-8be7-a33cd820ec95">

Find its URL adress and download it with `wget`:

```shell
[hadoop@xxxx ~]$ wget https://huggingface.co/datasets/legacy-datasets/wikipedia/resolve/main/data/20220301.simple/train-00000-of-00001.parquet
[hadoop@ip-172-31-63-215 ~]$ parquet-tools inspect train-00000-of-00001.parquet 

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
```

https://pypi.org/project/parquet-tools/
https://github.com/chhantyal/parquet-cli
