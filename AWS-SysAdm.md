# AWS Sysops Administrator

#AWS - Sysops Administrator

Deployment & Provisioning:

Services with root access to OS: Elastic Beanstalk, Elastic Mapreduce, Opsworks, EC2 
Its good to remember that we don’t have access to RDS, DynamoDB, S3 or Glacier.


ElasticLoadBalancer (ELB): ELB cannot span across regions or different VPCs

Two types of ELBs:
External ELBs with external DNS names (public dos name)
Internal ELBs with internal DNS names 

By default, ELB routes the traffic to the instance with lowest load

Two types of Sticky Sessions: Duration Based session stickiness (ELB generates the cookie)
Application controlled session stickiness (application generates the cookie)

Pre-warming ELB: needed to perform any load tests or flash traffic
AWS staff does it when a ticket is created  assigned to them with ticket details such as start and end dates of the load test, expected request rate per second, total request/repsonse size. 

High Availability:

Elasticity vs Scalability:

Elasticity is used for short time period such as hours or days. (stretch & retract back your infrastructure)
Scalability is used for longer time periods such as days, months or even years

Scale up vs Scale Out:
Scale up: vertical - increase ram, or instance type from t1.micro to t1.medium/large
Scale Out: horizontal - add another ec2 instance

Scaling out, you can scale back easily. But scaling up is easy, scaling down is not easy.

Comment: Amazon says that recently every EC2 instance has good network performance, however the old tables from amazon was says that the network performance depends on the ec2 instance type e.g.: t1.micro has low, t1.medium has moderate and t1.larger has good network performances.

Note: We can only have one (1) Internet Gateway. 

RDS: SQL Server: has  synchronous logical replication which using SQL-native mirroring technology
Others (Oracle, MySQL, PostgreSQL etc): has synchronous Physical replication
With Multi-AZ deployments: 
In the time of failover, we have configure connection string - RDS endpoint as the DNS name for failover to have it automatically (this is controlled by amazon). Amazon redirects traffic from primary to secondary in multi-AZs during a failover.
 Secondary server(backup server) is always in the same region within another AZ.

RDS Multi-AZ Advantages:
High Availability
Backup & Restores are take from secondary so that the primary server doesn’t take any performance hit.

We can force a failover from one AZ to another by rebooting the instance. This can be done using the AWS console or RebootDBInstance API Call.  RDS Multi-AZ Failover is not a scaling solution.

We do ReadReplicas as a scaling solution for RDS.

RDS & ReadReplicas:
What is a read-replica: read-only copies of your database.

Supported Versions: MySQL of 5.6 or higher & PostgreSQL of 9.3.5 or higher
Creating read Replicas:
If Multi AZs enabled: AWS takes a snapshot of primary db, brief I/O suspension about 1 minute on primary db
If Multi AZs not enabled: AWS takes a snapshot of secondary db, no I/O suspensions on primary db

Separate DNS End point address are created when a read replica is created which can be used with applications.

Read replicas can be promoted to a stand-alone db. this will break the replication between primary and secondary.

A Max of 5 read replicas can be created for MySQL & PostgreSQL.
Read Replicas can be spanned over different regions only for MySQL.
Replication is asynchronous only.
You an have multiple read replicas in different AZs but you can’t have read replicas both in AZ1 & AZ 2.
Can have read replicas of read replicas, but only for MySQL. (but this will increase latency)
DB Snapshots and automated backups of read replicas is not possible. It can only be done for primary or secondary dbs.
REPLICA LAG is a key metric to look at.


Without backups enabled or an available db snapshot, read replicas cannot be created.

Bastion host acts like a gateway between you and your EC2 instances.

Troubleshooting Autoscaling: Things to look for are: Associated key pair doesn’t exist
Security group doesn’t exist
Autoscaling config is not working properly
Autoscaling group is not found
Instance type specified is not supported in AZ
AZ itself is not supported
Invalid EBS device mapping
Autoscaling is not enabled in your account
Attempting to attach an EBS block device to an instance-store AMI

Read replicas can’t have multiple AZ

RTO - Recovery Time Objective

SECTION: 1
Monitoring, Metrics & Analysis : CloudWatch

Monitoring EC2: 4 default Metrics
CPU, Disk, Network & Status Checks - default metrics of AWS CloudWatch

Eg:- Ram Utilization is a custom metric and not a default metric

If we need to enable specific metrics that is not available in the default metric, we can setup a custom metric

Default Metrics are enabled at 5 minute interval unless detailed monitoring is enabled which makes 1 minute intervals

Retention Policy: 2 Weeks only, but can be requested longer than 2 Weeks using GetMetricStatistics API or third party tools offered by AWS partners

Also, we can receive the cloud watch metrics only upto 2 weeks after the termination of an EC2 instance

Custom Metrics - Min Granularity is 1 Min, it can be 3 or 5 or higher depending on the requirement

Create cloud watch alarms to monitor metrics and take appropriate action based on the alarm triggered.

EC2 Status Checks

System Status Checks - Checks Physical Host

Example Issues: Loss of power, Loss of network connectivity, SW/HW issues on Physical Host

Handling issues: Stop & Start the VM, so that if that VM is having issues it can move to another physical machine to get started.

Instance Status Checks - Checks the VM
Example Issues: failed system checks, exhausted memory, corrupted file system, incompatible kernel, misconfiguration
Handling issues: Reboot the instance/VM

Setting Up custom metrics:
To setup custom metrics on your EC2 machine, do the following steps:
a) update the repo manager such as apt-get or yum
b) install the perl packages 
c) download the monitoring scripts provided by amazon
d) run the scripts with proper arguments to get the specified custom metric data. This will send the custom metric data to the cloud watch. After sending the custom metric data to cloudwatch,  we can see the data in the cloud watch.

IF there is a problem with sending the custom metric data to the cloud watch - then the issue might be that the EC2 instance might not be assigned a IAM role. In order to have the role, we have to delete the EC2 instance and recreate it and assign the role during its creation.

Eg:
./mon-put-instance-data.pl —mem-util —verify -verbose
the above command will verify the details but will not post the data to cloud watch

./mon-put-instance-data.pl —mem-util —mem-used —mem-avail
the above command will post the data to the cloud watch

To get the scripts: Search for Amazon Cloud Watching Scripts for Linux


Note:
It is not possible to add a role to an existing EC2 instance. A Role can only be added to EC2 instance during its creation.


EBS - Elastic Block Storage
3 Types:

Magnetic / General Purpose SSD / Provisioned IOPS


1. Magnetic - 40-90 MiB/s with 1 Gb - 1 Tb
	Averages 100 IOPS, can burst unto hundreds of IOPS

2. General Purpose SSD 160 MiB/s with 1 GiB-16 TiB
	3 IOPS per GiB with a burst of 3000 IOPS unto 10,000 Max IOPS possible

	for 1 GiB, its 3 X 1 = 3 IOPS, total = 3000 - 3 = 2997 IOPS Burst performance achieved
	for 10 GiB, its 3 X 10 = 30 IOPS, total = 3000 - 30 = 2970 IOPS Burst performance achieved
	for 100 Gib, its 3 X 100 = 300 IOPS, total = 3000 - 300 = 2700 IOPS Burst performance achieved 

3. Provisioned IOPS SSD 320 MiB/s with 4 GiB-16 TiB
	20,000 IOPS Maximum and is consistently performed


Pre-Warming EBS Volumes: When a new EBS volume is create or restored (from a snapshot), all the blocks are allocated immediately. However, whenever it is accessed for the first time  it takes a performance hit on I/O of 5 to 50% loss. To avoid this we need to do something called a Pre-Warming which will avoid the 5- 50% I/O Loss.

(EBS) Volume Status Checks:
condition = status
degraded or severely degraded = warning 
stalled or not available = impaired

Monitoring RDS:
In Cloudwatch, we can monitor my RDS by metrics
IN RDS itself, we can monitor RDS by events, 

RDS Metrics:
Database Connections: The number of database connections in use.
DiskQueueDepth: The number of outstanding IOs (read/write requests) waiting to access the disk(optimum value is 0)
FreeStorageSpace: The amount of available storage space.
ReplicaLag(Seconds): The amount of time a Read Replica DB instance lags behind the source DB instance. Applies to MySQL, MariaDB, and PostgreSQL Read Replicas. 

Issue: when the users access the replication, data appears to be outdated because the replica lag is more than 60 seconds which should not be.
Read IOPS / Write IOPS: The average number of disk I/O operations per second.
Read Latency/ Write Latency: The average amount of time taken per disk I/O operation


Monitoring ELB:
Monitored every 60 Seconds (provided there is traffic)
 ELB Metrics:
HealthyHostCount: count of healthy instances in the AZ
HealthyHostCount: count of unhealthy instances in the AZ Latency: Measures the time elapsed in seconds after the request leaves the ELB until the response is received
HTTPCode_ELB_4XX
HTTPCode_ELB_5XX
Above two codes are generated by Load Balancer
HTTPCode_Backend_2XX/3XX/4XX/5XX - HTTP Codes generated by back end instances
BackendConnectionErrors: No. of connection that are successful established between the ELB and registered instances
SurgeQueueLength: total number of instances that are pending submission to a registered instance
SpillOverCount: total number of requests rejected due queue being full

Monitoring ElasticCache:
ElasticCache has 2 engines:
 Memcached: Multithreaded
Reddis: Not Multithreaded
4 Important  Things to look at:

a) CPU Utilization:
If Memcached hits 90% usage, use additional nodes
Reddis is not multi-threads to to calculate CPU util, divide the 90/no.of cores for the instance

b) Swap Usage
Memchached: should not exceed 50 Mb,if exceeded use memcached_connections_overhead to increase (this parameters tell the amount of memory reserved for memchached connections and other misc overhead)
Reddis: No Swap file, instead use reserved-memory   
c) Evictions: It occurs when a new is added due to which an old item is removed due to lack of free  space in the system 
Memcached: scale up (add more memory) or scale out (add more nodes)
Reddis: scale out (add read replicas)

d) Concurrent Connections
If there is a huge spike in concurrent connection there can be two thing either your have huge traffic or your application is not releasing connections properly. To avoid this, we can set an alarm on the number of concurrent connection for elasticache

Cloudwatch is not an in depth monitoring tool, to get a a very granular level detail analysis of the health and other resource conditions of a VM, servers we need to go for centralized monitoring tools such as pluck, ibm tivoli, HP operation manager etc.

Note: Subnets cannot span across availability zones, where as security groups can.

Need to allow ICMP/specific ports to a specific IP address/range of IPs in order to make communication possible between the monitoring server and other EC2 instances.

Cost Optimization:

EC2 Instance Types:
1. On demand 
2. Reserved
3. Spot Price
