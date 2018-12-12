# multi-instance-qmanager
Instructions for multi-instance qmanager

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
