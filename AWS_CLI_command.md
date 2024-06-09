 

A workaround to launch a Spark cluster on EMR when EMR's launch wizard does not function properly.

### Create a Cloud9 environment


Go to AWS Cloud9, AWS’ native cloud CLI environment. Choose **Create Environment** and open the Cloud9 IDE once ready.

Configurations:

- Give a name: e.g., <itsc-string>-emr-launch 
- Instance type: Choose t3-small(2 GiB RAM + 2 vCPU)
- Connection:  Secure Shell (SSH)

 <img width="1011" alt="image" src="https://github.com/justinjiajia/bigdata_lab/assets/8945640/0ed522d8-3784-4a73-8842-7c73f3eb5f72">


### Run AWS CLI commands

Examples:

```shell
aws emr create-cluster \
 --release-label emr-7.1.0 \
 --applications Name=Hadoop Name=Hive Name=Spark  \
 --service-role EMR_DefaultRole \
 --ec2-attributes KeyName=vockey,InstanceProfile=EMR_EC2_DefaultRole \
 --configurations '[{"Classification":"core-site","Properties":{"hadoop.http.staticuser.user":"hadoop"}},{"Classification":"hdfs-site","Properties":{"dfs.replication":"3"}}]' \
 --instance-groups InstanceGroupType=MASTER,InstanceCount=1,InstanceType=m4.large InstanceGroupType=CORE,InstanceCount=3,InstanceType=m4.large
```

```shell
aws emr create-cluster \
 --name "My cluster" \
 --log-uri "s3://aws-logs-339712892718-us-east-1/elasticmapreduce" \
 --release-label "emr-7.1.0" \
 --service-role "arn:aws:iam::339712892718:role/EMR_DefaultRole" \
 --unhealthy-node-replacement \
 --ec2-attributes '{"InstanceProfile":"EMR_EC2_DefaultRole","EmrManagedMasterSecurityGroup":"sg-0e760c758acd676b7","EmrManagedSlaveSecurityGroup":"sg-091de8a5a48f351e5","KeyName":"vockey","AdditionalMasterSecurityGroups":[],"AdditionalSlaveSecurityGroups":[],"SubnetId":"subnet-00d31ecbf9ca1d2f3"}' \
 --applications Name=Hadoop Name=Hive Name=Spark \
 --configurations '[{"Classification":"core-site","Properties":{"hadoop.http.staticuser.user":"hadoop"}},{"Classification":"hdfs-site","Properties":{"dfs.replication":"3"}}]' \
 --instance-groups '[{"InstanceCount":4,"InstanceGroupType":"CORE","Name":"Core","InstanceType":"m4.large","EbsConfiguration":{"EbsBlockDeviceConfigs":[{"VolumeSpecification":{"VolumeType":"gp2","SizeInGB":32},"VolumesPerInstance":1}]}},{"InstanceCount":1,"InstanceGroupType":"MASTER","Name":"Primary","InstanceType":"m4.large","EbsConfiguration":{"EbsBlockDeviceConfigs":[{"VolumeSpecification":{"VolumeType":"gp2","SizeInGB":32},"VolumesPerInstance":1}]}}]' \
 --bootstrap-actions '[{"Args":[],"Name":"python_libraries","Path":"s3://ust-bigdata-class/install_python_libraries.sh"}]' \
 --scale-down-behavior "TERMINATE_AT_TASK_COMPLETION" \
 --auto-termination-policy '{"IdleTimeout":3600}' \
 --region "us-east-1"
```

https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-spark-launch.html
