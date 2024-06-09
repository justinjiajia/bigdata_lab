

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
```

https://pypi.org/project/parquet-tools/
https://github.com/chhantyal/parquet-cli
