 

# AWS CLI command to launch a Spark cluster on EMR

A workaround when EMR's launch wizard does not function properly:

```shell
aws emr create-cluster --applications Name=Hadoop Name=Spark --release-label emr-7.1.0 --service-role EMR_DefaultRole --ec2-attributes KeyName=vockey,InstanceProfile=EMR_EC2_DefaultRole --instance-groups InstanceGroupType=MASTER,InstanceCount=1,InstanceType=m4.large InstanceGroupType=CORE,InstanceCount=3,InstanceType=m4.large
```

https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-spark-launch.html
