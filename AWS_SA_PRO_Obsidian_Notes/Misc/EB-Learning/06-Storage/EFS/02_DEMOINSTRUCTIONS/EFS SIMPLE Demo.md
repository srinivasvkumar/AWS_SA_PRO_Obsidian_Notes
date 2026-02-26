---
aliases:
- "EFS DEMO"
---

# EFS DEMO

Welcome ... in this demo lesson you will implement a simple [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] architecture within an AWS [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]  

# Stage 1 - APPLY BASE INFRASTRUCTURE template  
This stage will create a basic infrastructure in us-east-1, consisting of  

- A [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] using the 10.16.0.0/16 range  
- 2 x Web Subnets, WebA and WebB  
- 2 x App Subnets, AppA and AppB  
- An Internet Gateway  
- [[appsync|Security]] Groups  
- Instance Role  
- [[ssm]] Endpoints  
- Route Tables  
- Two [[ec2]] Instances - 1 running in WebA the other in WebB  

Apply the base [[cloudformation]] template by clicking [Here](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/quickcreate?templateURL=https://learn-cantrill-labs.s3.amazonaws.com/aws-simple-demos/aws_simple_efs/EFSInfra.yaml&stackName=EFS)  
** be sure to tick the AWS::[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]]::Role capabilities checkbox** before clicking `Create STack`  
Wait until the `EFS` stack moves into the `CREATE_COMPLETE` status before continuing  

# Stage 2 - Create [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] File System and Mount Points  
Move to the `EFS Console` https://console.aws.amazon.com/efs/home?region=us-east-1  
Click `Create file system`  
Set the name to `A4LEFS`  
In the [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Dropdown choose `a4l-vpc1`  
Click `Customize`  
Ensure the options are set as below (these should be the default)  

`Enable Automatic Backups` = `Enabled` ... this integrates with the AWS Backups product for automatic backups  
`Lifecycle Management` = `30 days since last access` .. this ensures a cost effective implementation by moving files to lower cost storage  
`Performance Mode` = `General Purpose` .. this is the default and good for most usecases as its lowlatency and great overall performance. `MAX I/O` is higher latency but better for larger  scale parallel workloads.  

`Throughput Mode` = `Bursting` this is like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] where the performance scales with the Size of the storage. `Provisioned` allows control independantly of size.  
 
`Encryption` = `Enable encryption of data at rest` .. this uses [[kms]] to encrypt all `at rest` data. Optionally it allows selection of a specific key.  

Click `Next`  
[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] should still be selected as `a4l-vpc1` make sure it is.  

Next you need to configure mount targets, 1 per AZ.  
In the top row ...`us-east-1a` `sn-app-A` and `EFS-InstanceSG-XXXXX` as the [[appsync|security]] group (there will be randomness at the end of the name , thats ok)  
In the bottom row .. `us-east-1b` `sn-app-B` and `EFS-InstanceSG-XXXXX` as the [[appsync|security]] group (there will be randomness at the end of the name , thats ok)  
Click `Next`  

Don't set any of these File SYstem [[policies]]  

Click `next`  
Scroll down & Click `Create` to create the `file system` and `mount targets`  

The file system will start in the `File System State` of `Creating` and then move to `Available`  
Once it does, click `A4LEFS`  
Click `Network` Tab  
These are the ENIs which your [[ec2]] instances will connect too ... they are ENIs with IP addresses in the [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]  
The two `mount targets` will start off in the `Creating` State ... wait until both are in the `Available` state before continuing.  

Keep clicking the `Refresh` button until both show as `Available` before continuing.  


# Stage 3 - Mount [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] on InstanceA  
In this stage, you will login to one [[ec2]] instance, install the software required to work with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]], configure and mount the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] file system and then test.  
Open a new tab to the [[ec2]] console (Click Services, type [[ec2]], right click, open in new tab)  https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Home:  

Click `Running Instances`  
Select `A4L-EFSInstanceA`, right click, connect  
Select `Session manager` & click `Connect`  

Verify there are no currently mounted [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] file systems  

```
sudo ls -la /
df -k
```

To support [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]], even though it's based on NFS, you need to install a package  
```
sudo yum -y install amazon-efs-utils
```

Create a `mount point` for the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] file system  
```
sudo mkdir -p /efs/wp-content
```

Edit the /etc/fstab file which controls which file systems are mounted to a linux server  

```
sudo nano /etc/fstab
```

Paste in this line at the bottom  
```
file-system-id:/ /efs/wp-content efs _netdev,tls,iam 0 0
```

You will need to replace the file system ID  
Position your cursor over `:` delete to the start of the line  
Return to the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] console https://console.aws.amazon.com/efs/home?region=us-east-1#/file-systems  
Locate the file system ID `fs-xxxxxx` of the A4LEFS file system, copy this into your clipboard  
Return to the Session Manager sessio on the instance  
Position your curson over `:` and then paste in the file system ID  

Save the file with `ctrl+o` and exit with `ctrl+x`  

Next , mount the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] file system in the mount point by running  
```
sudo mount /efs/wp-content  
```

and then verify its mount by running  
```
df -k
```

lets add a test file  
```
cd /efs/wp-content
sudo touch amazingtestfile.txt
sudo nano amazingtestfile.txt
```
enter something cool  
then `ctrl+o` to save and `ctrl+x` to exit  




# Stage 4 - Mount [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] on InstanceB
Click `Running Instances`  
Select `A4L-EFSInstanceB`, right click, connect  
Select `Session manager` & click `Connect`  

Verify there are no currently mounted [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] file systems  

```
sudo ls -la /
df -k
```

To support [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]], even though it's based on NFS, you need to install a package  
```
sudo yum -y install amazon-efs-utils
```

Youre now going to mount the file system on this instance  

First, create the mount point for the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] filesystem  

```
sudo mkdir -p /efs/wp-content
```

then mount the file system into this mountpoint  

```
sudo nano /etc/fstab
```

Paste in this line at the bottom  

```
file-system-id:/ /efs/wp-content efs _netdev,tls,iam 0 0
```

You will need to replace the file system ID  
Position your cursor over `:` delete to the start of the line  
Return to the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] console https://console.aws.amazon.com/efs/home?region=us-east-1#/file-systems  
Locate the file system ID `fs-xxxxxx` of the A4LEFS file system, copy this into your clipboard  
Return to the Session Manager sessio on the instance  
Position your curson over `:` and then paste in the file system ID  

Save the file with `ctrl+o` and exit with `ctrl+x`  

Next , mount the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] file system in the mount point by running  
```
sudo mount /efs/wp-content
```

and then verify its mount by running  
```
df -k
```

then lets move into that folder and check the content from the previous [[ec2]] instance is there  

```
cd /efs/wp-content
ls -la
cat amazingtestfile.txt
```


# Stage X - Cleanup
Move back to the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EFS]] console https://console.aws.amazon.com/efs/home?region=us-east-1#/file-systems  
Select the `A4LEFS` File System  
Click `Delete`  
Type, or paste in the FS-XXXXX id into the box and click `Confirm` to remove the file system  
Wait for this to complete  
Then return to the [[cloudformation]] console https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filteringText=&filteringStatus=active&viewNested=true&hideStacks=false  
Select the `EFS` stack  
Click `Delete`  
Click `Delete Stack`  
Once deleted, the account will be in the same state as it was at the start of the demo  



