

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
