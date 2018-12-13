# multi-instance-qmanager
Instructions for multi-instance qmanager on two AZs

### Prepare MQ Binary in EC2 / S3

```
Upload to IBM_MQ_9.1_LINUX_X86-64_TRIAL.tar.gz to s3
Red Hat Enterprise Linux 7.6 (HVM), SSD Volume Type - ami-011b3ccf1bd6db744 (64-bit x86) / ami-0e3688b4a755ad736 (64-bit Arm)
Select EC2 - t2.large
Create IAM Role for EC2 to access s3 -> Select AmazonS3FullAccess on permissions tab -> Role Name MQFELLOW-S3FullAccess
ssh -i "blockchain.pem" ec2-user@ec2-54-196-124-22.compute-1.amazonaws.com
```

### Install AWSCLI

```
cd ~
curl "https://s3.amazonaws.com/aws-cli/awscli-bundle.zip" -o "awscli-bundle.zip"
sudo yum install unzip -y
unzip awscli-bundle.zip
sudo ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
aws s3 cp s3://mqfellow-us-east-1/mq-installer/IBM_MQ_9.1_LINUX_X86-64_TRIAL.tar.gz /home/ec2-user/IBM_MQ_9.1_LINUX_X86-64_TRIAL.tar.gz
tar -xzvf IBM_MQ_9.1_LINUX_X86-64_TRIAL.tar.gz
```

### Install MQ

* https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_9.1.0/com.ibm.mq.ins.doc/q008640_.htm

```
cd MQServer
echo 1 | sudo ./mqlicense.sh 
sudo rpm -ivh MQSeriesRuntime*.rpm
sudo rpm -ivh MQSeriesJRE*.rpm
sudo rpm -ivh MQSeriesJava*.rpm
sudo rpm -ivh MQSeriesServer*.rpm
sudo rpm -ivh MQSeriesWeb*.rpm
sudo rpm -ivh MQSeriesFTBase*.rpm
sudo rpm -ivh MQSeriesFTAgent*.rpm
sudo rpm -ivh MQSeriesFTService*.rpm
sudo rpm -ivh MQSeriesFTLogger*.rpm
sudo rpm -ivh MQSeriesFTTools*.rpm
sudo rpm -ivh MQSeriesAMQP*.rpm
sudo rpm -ivh MQSeriesAMS*.rpm
sudo rpm -ivh MQSeriesXRService*.rpm
sudo rpm -ivh MQSeriesExplorer*.rpm
sudo rpm -ivh MQSeriesGSKit*.rpm
sudo rpm -ivh MQSeriesClient*.rpm
sudo rpm -ivh MQSeriesMan*.rpm
sudo rpm -ivh MQSeriesMsg*.rpm
sudo rpm -ivh MQSeriesSamples*.rpm
sudo rpm -ivh MQSeriesSDK*.rpm
sudo rpm -ivh MQSeriesSFBridge*.rpm
sudo rpm -ivh MQSeriesBCBridge*.rpm
sudo rpm -qa | grep MQ
cd /opt/mqm/bin/
sudo ./setmqinst -i -p /opt/mqm
echo "passw0rd" | sudo passwd --stdin mqm 
echo "passw0rd" | su - mqm -c 'dspmqver'

in CFT
./setmqinst -i -p /opt/mqm
echo "passw0rd" | passwd --stdin mqm 
su - mqm -c 'dspmqver'
```

### EFS Setup

Source https://docs.aws.amazon.com/efs/latest/ug/wt1-test.html

select two subnet 
select sg 
Choose performance mode
General Purpose
Choose throughput mode
Bursting

Enable encryption
If you enable encryption for your file system, all data on your file system will be encrypted at rest. You can select a KMS key from your account to protect your file system, or you can provide the ARN of a key from a different account. Encryption of data at rest can only be enabled during file system creation. Encryption of data in transit is configured when mounting your file system.

Enable encryption of data at rest - 
Key ARN	arn:aws:kms:us-east-1:962934755172:key/e86a4ce8-5f3f-4fa1-98eb-3fe818ac5752
Description	Default master key that protects my EFS filesystems when no other key is defined

It will create two ENI

DNS name	fs-7a74b89b.efs.us-east-1.amazonaws.com
Amazon EC2 mount instructions (from local VPC)
Amazon EC2 mount instructions (across VPC peering connection)
On-premises mount instructions

To set up your EC2 instance:

Using the Amazon EC2 console, associate your EC2 instance with a VPC security group that enables access to your mount target. For example, if you assigned the "default" security group to your mount target, you should assign the "default" security group to your EC2 instance. Learn more
Open an SSH client and connect to your EC2 instance. (Find out how to connect.)
If you're using an Amazon Linux EC2 instance, install the EFS mount helper with the following command:
sudo yum install -y amazon-efs-utils
You can still use the EFS mount helper if you're not using an Amazon Linux instance. Learn more

sudo yum install -y nfs-utils

sudo mkdir /media/mqfellow-efs

before mounting, ensure that security group i.e. default is added into the instance.

```
sudo mount -t nfs4  -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-7a74b89b.efs.us-east-1.amazonaws.com:/ /media/mqfellow-efs

mkdir -p /media/mqfellow-efs/logs /media/mqfellow-efs/qmgrs 
sudo chown -R /media/mqfellow-efs/
sudo chmod -R ug+rwx /media/mqfellow-efs/
```
sudo umount efs to unmount - not required.


### IP1 MQ EFS Configuration 

Multi-instance queue managers
https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_9.1.0/com.ibm.mq.con.doc/q018140_.htm

Creating Multi Instance Queue Managers On Linux using MQv7.1
https://www.youtube.com/watch?v=Fv4tLQmsHpI

```
sudo mkdir /media/mqfellow-efs/logs /media/mqfellow-efs/qmgrs
sudo chown -R /media/mqfellow-efs/ 
sudo chmod -R ug+rwx /media/mqfellow-efs/ 

su - mqm

crtmqm -ld /media/mqfellow-efs/logs -md /media/mqfellow-efs/qmgrs -q PSAP

Generate the command to run in other server
$ dspmqinf -o command PSAP
addmqinf -s QueueManager -v Name=PSAP -v Directory=PSAP -v Prefix=/var/mqm -v DataPath=/media/mqfellow-efs/qmgrs/PSAP
```

### Create another EC2 instance on IP2

t2.large
security group - add launchwizard. add the default sg later - all acces - for testing - same as security group in efs
subnet us-east-1b - same as subnet that is used in efs
s3 - mqfellow-s3fullaccess
use your own keychain

install awscli, s3, efs and mq

```
cd ~
curl "https://s3.amazonaws.com/aws-cli/awscli-bundle.zip" -o "awscli-bundle.zip"
sudo yum install unzip -y
unzip awscli-bundle.zip
sudo ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
aws s3 cp s3://mqfellow-us-east-1/mq-installer/IBM_MQ_9.1_LINUX_X86-64_TRIAL.tar.gz /home/ec2-user/IBM_MQ_9.1_LINUX_X86-64_TRIAL.tar.gz
tar -xzvf IBM_MQ_9.1_LINUX_X86-64_TRIAL.tar.gz
cd MQServer
echo 1 | sudo ./mqlicense.sh 
sudo rpm -ivh MQSeriesRuntime*.rpm
sudo rpm -ivh MQSeriesJRE*.rpm
sudo rpm -ivh MQSeriesJava*.rpm
sudo rpm -ivh MQSeriesServer*.rpm
sudo rpm -ivh MQSeriesWeb*.rpm
sudo rpm -ivh MQSeriesFTBase*.rpm
sudo rpm -ivh MQSeriesFTAgent*.rpm
sudo rpm -ivh MQSeriesFTService*.rpm
sudo rpm -ivh MQSeriesFTLogger*.rpm
sudo rpm -ivh MQSeriesFTTools*.rpm
sudo rpm -ivh MQSeriesAMQP*.rpm
sudo rpm -ivh MQSeriesAMS*.rpm
sudo rpm -ivh MQSeriesXRService*.rpm
sudo rpm -ivh MQSeriesExplorer*.rpm
sudo rpm -ivh MQSeriesGSKit*.rpm
sudo rpm -ivh MQSeriesClient*.rpm
sudo rpm -ivh MQSeriesMan*.rpm
sudo rpm -ivh MQSeriesMsg*.rpm
sudo rpm -ivh MQSeriesSamples*.rpm
sudo rpm -ivh MQSeriesSDK*.rpm
sudo rpm -ivh MQSeriesSFBridge*.rpm
sudo rpm -ivh MQSeriesBCBridge*.rpm
sudo rpm -qa | grep MQ
cd /opt/mqm/bin/
sudo ./setmqinst -i -p /opt/mqm
echo "passw0rd" | sudo passwd --stdin mqm 
echo "passw0rd" | su - mqm -c 'dspmqver'
sudo yum install -y nfs-utils
sudo mkdir /media/mqfellow-efs
sudo mount -t nfs4  -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport fs-7a74b89b.efs.us-east-1.amazonaws.com:/ /media/mqfellow-efs
```

### Install and test qmgr in IP2

```
addmqinf -s QueueManager -v Name=PSAP -v Directory=PSAP -v Prefix=/var/mqm -v DataPath=/media/mqfellow-efs/qmgrs/PSAP

On IP1, start the qmanager
strmqm -x PSAP
dspmq
QMNAME(PSAP)                                              STATUS(Running)
dspmq -x -o standby
QMNAME(PSAP)                                              STANDBY(Permitted)
    INSTANCE(ip-172-31-23-68.ec2.internal) MODE(Active)

On IP2, start the qmanager
dspmq
QMNAME(PSAP)                                              STATUS(Running elsewhere)
strmqm -x PSAP
The queue manager is associated with installation 'Installation1'.
A standby instance of queue manager 'PSAP' has been started. The active
instance is running elsewhere.
$ dspmq -x -o standby
QMNAME(PSAP)                                              STANDBY(Permitted)
    INSTANCE(ip-172-31-23-68.ec2.internal) MODE(Active)
    INSTANCE(ip-172-31-67-58.ec2.internal) MODE(Standby)

```

### Failover 

```
On IP1
$ endmqm -is PSAP
IBM MQ queue manager 'PSAP' ending.
IBM MQ queue manager 'PSAP' ended, permitting switchover to a standby instance.

On IP2
$ dspmq -x -o standby
QMNAME(PSAP)                                              STANDBY(Permitted)
    INSTANCE(ip-172-31-67-58.ec2.internal) MODE(Active)

On IP1
$ strmqm -x PSAP
The queue manager is associated with installation 'Installation1'.
A standby instance of queue manager 'PSAP' has been started. The active
instance is running elsewhere.

On IP2
$ dspmq -x -o standby
QMNAME(PSAP)                                              STANDBY(Permitted)
    INSTANCE(ip-172-31-67-58.ec2.internal) MODE(Active)
    INSTANCE(ip-172-31-23-68.ec2.internal) MODE(Standby)


```

