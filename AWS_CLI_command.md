 

# AWS CLI command to launch a Spark cluster on EMR

A workaround when EMR's launch wizard does not function properly.
first create a AWS Cloud9:

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
