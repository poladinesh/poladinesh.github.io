# AWS Solutions Architect Associate
#AWS - Solutions Architect
￼

￼


80 minutes
60 Questions or more
70% average cutoff

AWS Global Infrastructure: Physical infra of AWS & its associated terminology
Regions - is nothing but a geographical area; consists of collection of independent computing resources in defined geography
Availability Zones(AZ) - is nothing but a data center 
Edge Locations are CDN End Points for CloudFront

Network & Content Delivery:
VPC stands for Virtual Private Cloud (A virtual data center)
Route53 - DNS Service for Amazon (its called 53 because the DNS default port is 53)
CloudFront - Consists of  several Edge Locations 
DirectConnect: uses a dedicated line/network from you local network to AWS

Compute:
EC2: stands for Elastic Compute Cloud
All virtual machines in the Amazon cloud

EC2 Container Service: Run docker in EC2 
Elastic Beanstalk: upload code to beanstalk, which will take care of underlying infrastructure to produce results
Lambda: AWS service allows you to run code without having to worry about provisioning any underlying resources (such as virtual machines, databases etc) {Serverless service of AWS, upload the code to lambda & it will respond to the events (e.g.: Echo, Alexa)}
Lightsail: out of the box cloud, ligtsail will deploy wordpress or zoomla or other sites for you.


Storage: S3(simple storage service): used to store objects in cloud
Glacier: is used to archive files (takes few hours to access them)
EFS(Elastic File Service): store blocks(block based storage) 
Storage Gateway: Used to connect S3 to your on-premise datacenter

Databases:
RDS: Relational DB like SQL, Oracle, PostgreSQL DynamoDB: non relational DB
Redshift: Datawarehousing solution of Amazon
Elasticache: Caching mechanism of data in cloud (relieves data load from the db that are frequently accessed)

Migration: Snowball: used to import & export date into & from the cloud
DMS(Database Migration Services): Allows to migrate databases from on-premises data center to cloud.
SMS (Server Migration Services): Server Migration from VMware VMs to AWS VMs

Analytics: Athena: used to search flat files using SQL queries
EMR(Elastic Mapreduce): Used to Big Data processing
Cloud Search: Amazon search used for application or website Elastic Search: uses open source framework
Kinesis: used for collating large amounts of data streamed from multiple resources; streaming and analyzing real time data
Data Pipeline: Move data from one place to another (e.g: move data from  S3 to DynamoDB)
Quick Sight: Business Analytics tools helps to create visualizations, dashboards

Security & Identity:
IAM (Identity Access Management):
Inspector: an agent installed on VMs which is used for security reporting
Certificate manager: Gives free SSL Certificates for website & applications
Directory Service: connecting Active Directory to AWS
WAF (Web Application Firewall): application level protection to website 
Artifacts: Documentation such as  ISO standards and others are stored here. AWS>Compliance reports>AWS Artifact
 
Management Tools:
CloudWatch: Monitor AWS environment 
Cloud Formation: Infrastructure as code, a document describes was environment visa son script
Cloud Trail: Auditing AWS Resources
Opsworks: Automating deployments using chef
Config Manager: Monitors the configuration and sends alerts when a config is about to break
Service Catalog: 
Trusted Advisor: Gives recommendations, tips about your AWS environment

Application Services:
Step Functions: Used to check what micro services or what is inside your application actually looks like
SWF (Simple Work Flow): Coordinating automated & human tasks
 API Gateway: Allows to create, modify API ; doorway to accessing backend servers with AWS
AppStream: its a way of streaming Desktop applications to users
Elastic Transcoder: changes the format of video to suit different devices

Developer Tools
CodeCommit: It is Github, store your code in AWS
CodeBuild: way f compiling your code
CodeDeploy: Deploys your code to EC2 instances
Code Pipeline:  keeping track of different versions of your code

Messaging:
SNS (Simple Notification Service): Send notification via email or text 
SQS (Simple Queue Service): Queue System SES (Simple Email Service): sending  and receiving emails from AWS



Complete List of Services in AWS: 
￼


IAM:
IAM doesn’t have a region, it is global
IAM consists of following
users: 	
groups: group of users
roles: see below
policies: policy document written in json (javascript object notation)


Role: Its a way of allowing one AWS service to access another AWS service

Root account is the one that is created when we setup a AWS account. It has full Admin access by default.
New Users have no permission by default. They are assigned Access Key ID & Secret Access keys which can only used
programmatically (APIs & Commandline) and are not replace for passwords. They cannot be used for logging in into AWS console.
These are not available for second time, the first they are seen should be downloaded and stored. If gone, we need to regenerate again.

Power user: gives access to all AWS services except the management of groups and users within IAM

S3: Simple Storage Service

S3 is a Object based storage (to store files, videos, photos and other documents)
Block based storage (used to store OS or run a database etc)
S3 is a universal namespace ,which means that the bucket names must be unique globally
0 bytes to 5 TB
Unlimited storage
Files are stored in buckets (bucket is like a folder)

S3 link breakdown
https://s3-REGION_NAME.amazonaws.com/BUCKETNAME
Example:
https://s3-eu-west-1.amazonaws.com/cloudguru

A bucket that has static webhosting enabled on it will always have the format; https://<bucket-name>.s3-website-<AWS-region>.amazonaws.com

Amazon’s SNS has the following subscribers; Lambda, SQS, HTTPS, Email, SMS

Data Consistency model for S3
a) Read after write consistency for PUTS of new object
If a new object is put into S3, we can immediately read it
b) Eventual consistency for overwrite PUTS and DELETES (can take some time to propagate)
If we a updating or deleting an object it can take some time
Updates to S3 are atomic, either we get the new data or the old data. But we never get partial or corrupted data.

S3 is object based, so the object has the following: Key -  name of the object
Value - data of the object
VersionID - version
Metadata- data about data
Subresources: ACL & Torrent

shows successful upload gives 200 status code

Storage Tiers/Classes:
S3 - 99.99% availability and 99.999999999% durability (design to sustain loss of 2 facilities concurrently)
S3 - IA — infrequently accessed information - accessed less frequently but requires rapid access when needed (a retrieval fee is charged)
S3-RRS - Reduced redundancy storage - 99.99% availability and 99.99% durability
Glacier: cheapest of all, used for archival only, takes 3-5 hours of time to restore
￼

￼
S3 - Charges
Charged for:
Storage
Requests (requests made to objects)
Storage Management Pricing - per tag pricing
Data Transfer Pricing (moving data within regions/AZs)
Transfer Acceleration 

Read S3 FAQ before taking Exam

Once enable, you cannot disable versioning, only suspended.
Versioning can also use Multifactor authentication delete capability on buckets and objects
Cross region replication needs versioning to be enabled on the source bucket

Lifecycle Management
Can be used along with versioning 
can be applied to current and previous versions
	can be transitioned to Standard IA after 30 days of creation date (min 128 kb)
	can be archived to Glacier storage, after 30 days from Standard - IA 
	delete the objects/buckets permanently (deletes from not only S3 but from Glacier too)

CloudFront:
CDN is a content delivery network is a system of distributed servers that delivers web pages and other web content to the user based on the geographical location of the users, the origin of the web page and a CDN server.

Edge location: this is the location where the content will be cached(its not a region or a AZ)
Origin: Its the origin of all the files that CDN will distribute. (it can be a S3, EC3, Route 53 or an ELB) (an origin can also be non-AWS)
Distribution: this is the name given to the CDN which consists of a collection of edge locations

Web Distribution: used for websites
RTMP: used from streaming

Edge location are not read only, they can also be written
objects are cached for the life of TTL (Time To Live) - objects are expired after the TTL
You can clear the cached objects, but you will be charged

S3-Security
Securing the buckets:
All new buckets are private
we can setup access control using: Bucket Policies and Access Control lists.
S3 buckets can be setup to write access logs for the requests made to the buckets.

Encryptions:
In Transit: SSL/TLS -> sending data in and out of the bucket
At Rest: 
Server side encryption: 
	a) S3 Managed Keys - SSE-S3  ——> mostly used
	b) AWS Key Management Service , Managed Keys - SSE-KMS (additional features such as envelope key which protects encryption key and order trail of the key usage and misuses, when and where the key was used etc.)
	c) SSE with Customer provided keys - SSE-C
Client side encryption

Storage Gateway:
It is a Service that connects on-premises software appliance with cloud based storage to provide seamless and secure integration between an on-premise IT environment and AWS infrastructure.

The AWS Storage Gateway appliance is available for download as VM Image that can be installed on a host in a datacenter. It support a VMWare ESXi or Microsoft Hyper-V. Once installed, we can activate it and use AWS Management console to create a storage gateway option that is needed.

4 types of Storage Gateways:
File Gateway (NFS): store flat files directly on S3
Volume Gateway(iSCSI): Block based storage: 
	Stored volumes  : Entire dataset is stored onsite and is asynchronously backed up to S3
	Cached volumes:  Entire dataset is stored on S3 and only frequently accessed data is stored onsite
Tape Gateway (VTL): Virtual Tape Library:
Used for Backup and uses popular backup applications such as NetBackup, Backup Exec, Veam etc


Snowball:
Earlier it used to be import/export where the customer need to send their own disk to amazon for data transfer.
Snowball - Onboard storage - uptp 80 TB
Snowball Edge - Onboard storage and compute capabilities - unto 100 TB
Snowball Mobile - Exabyte scale transfer - provided with a semi truck attached to a shipping container

Snowball can: import from S3 and export to S3. If its glacier, data needs to be moved to S3 and then onto Snowball

S3 Transfer Acceleration: Instead of uploading the files to S3, we can transfer to the edge location directly. Which then will be transferred to S3 later.
We will get a distinct url to upload to.
We can enable this from the properties of the bucket from S3

We can load files to S3 much faster when multi-part upload.

EC2: Elastic Compute Cloud:

It is a webservice that allows a resizable compute capacity in cloud.

EC2 Options:
Demand: allows you to pay a fixed rate by the hour with no commitment
Reserved: provides a capacity reservation.  (1-3 year terms contract) gets discount 
Spot pricing: Instances are spin up when the bid price is higher than spot price
Dedicated Hosts: These are you physical ec2 dedicated for your use. We can use the existing software licensing. (pay by hour with no long term commitment)

If the spot instance is terminated by Amazon, then the usage for partial hour is not charged. however, if the spot instance is terminated by the user, he is charged for the usage even it is partial.

Dr Mc GIFT Px
Density, RAM, Main choice for general purpose, Compute, Graphics, IOPS, FPGA, T - Cheap compute, P - Graphics Pix, Extreme memory

IOPS - Input Output Operations Per Second
 EBS: Elastic Block Storage
EBS allows us to create storage volumes and attach them to EC2 instances. Unlike S3 which is a object based storage, EBS is block based storage which allows us to run a database, create a file system or even install an OS etc.

EBS - Elastic Block Storage
3 Types:

Magnetic (standard/ throughput optimized HDD and cold HDD)
General Purpose SSD / Provisioned IOPS


1. Magnetic - 40-90 MiB/s with 1 Gb - 1 Tb
	Averages 100 IOPS, can burst unto hundreds of IOPS

2. General Purpose SSD 160 MiB/s with 1 GiB-16 TiB
	3 IOPS per GiB with a burst of 3000 IOPS unto 10,000 Max IOPS possible

	for 1 GiB, its 3 X 1 = 3 IOPS, total = 3000 - 3 = 2997 IOPS Burst performance achieved
	for 10 GiB, its 3 X 10 = 30 IOPS, total = 3000 - 30 = 2970 IOPS Burst performance achieved
	for 100 Gib, its 3 X 100 = 300 IOPS, total = 3000 - 300 = 2700 IOPS Burst performance achieved 

3. Provisioned IOPS SSD 320 MiB/s with 4 GiB-16 TiB
	20,000 IOPS Maximum and is consistently performed

We cannot mount 1 EBS volume to multiple EC2 instance, instead we can use EFS (Elastic File System)

Pre-Warming EBS Volumes: When a new EBS volume is create or restored (from a snapshot), all the blocks are allocated immediately. However, whenever it is accessed for the first time  it takes a performance hit on I/O of 5 to 50% loss. To avoid this we need to do something called a Pre-Warming which will avoid the 5- 50% I/O Loss.

(EBS) Volume Status Checks:
condition = status
degraded or severely degraded = warning 
stalled or not available = impaired

One Subnet equals one AZ, a subnet cannot span across multiple AZs.
Volume Type - Root specifies that this volume is going to be the boot volume.

￼

chkconfig httpd start —> this will make sure that the apache gets started every time the VM is rebooted

Security Groups:
Security Groups are nothing but a virtual firewall.
All inbound traffic is blocked by default
All outbound traffic is allowed by default
Changes to Security Group is effective immediately
We can have multiple security groups for a single EC2 instance
We can have multiple instances in a single Security Group
Security Groups are stateful, which means inbound rules automatically add outbound rules.
Any inbound rule specified also implicitly define the corresponding outbound rule. We need not specify outbound rule explicitly for the same rule matching with the inbound rule.
We
We cannot block specific IP Address using security groups. We can do that using Network ACLs.
In VPC, for Network Access Control Lists - we need to add inbound as well as outbound rules because they are stateless.
We can allow rules, but not deny rules with Security Groups


default ports:
RDP - 3389 
MySQL/Aurora - 3306

Volumes & Snapshots:

lsblk -> shows mount and partition of the existing volume attached to the EC2 instance
file -s /dev/xvdf —> if this command returns data then the volume has no data in it.
mkfs -t ext4 /dev/xvdf —> this will format the volume and provide with a filesystemm
mount /dev/xvdf /fileserver —> mounting the volume to /fileserver

Volumes exist in EBS. They are nothing but virtual hard disks
Snapshots are stored on S3
Snapshots are point in time copies of volumes
Snapshots are incremental, only the blocks that are changed since your last snapshot are stored on S3 not the entire volume
Snapshots may take some time for the first time, but actually get faster as we go on

snapshots can be shared only if they are unencrypted.
to create a snapshot, instance should be stopped before taking a snapshot.

Why RAID?
Disk I/O can be improved by creating a RAID.
AWS doesn’t recommend putting RAID 5’s on EBS.

Problem: How to take snapshot of RAID array?
Puzzle: Snapshot excludes the caches held by application and OS. 
Solution: take a application consistent snapshot.

stop the application from writing to the disk.
Flush all the application caches

how to do this?
freeze the file system
Unmount the RAID array
shutdown the EC2 instance

AMIs are regional. AMIs can only be launched from the region they are created. However, we can copy the AMIs from one region to another region using Amazon API, console or command line.
AWS - ELB
two types: Classic Loadbalancer and Application LoadBalancer
 Classic LoadBalancer: Works at layer 4 - TCP/IP Layer
Application LoadBalancer: Works at layer 7 - Application Layer (introduced in mid 2016)

ELBs have DNS name. They are not given IP Addresses. Amazon manages it.

CloudWatch: 
Dashboard, Alarms, Events(respond to state changes of resources), Logs
Standard Monitoring: 5 Minutes
Detailed Monitoring : 1 Minute

Note:
CloudWatch is Monitoring & Logging Whereas CloudTrail is for Auditing

Dashboards, Alarms, Events, Logs

Roles: Roles are more secure than storing your AWS access key and secret access key on individual EC2 instances.
They are easy to manage.
They can only be assigned during the creation of an EC2 instance(cannot  be added to existing ec2 instances)
Roles are universal, they can be used in any region.
Changes can be made to roles and the changes are effective immediately.

Instance Meta-data: used to get information about an instance
use the following url for meta-data about the instance
curl http://169.254.269.254/latest/meta-data
there is nothing such as user-data

EFS: Elastic File System
supports Network FileSystem version 4 protocol
only required to pay for the storage u use (no need to pre-provisioning)
scales upto petabytes
supports thousands of concurrent NFS connections
data is stored across multiple AZs within a region
read after write consistency
We can mount multiple EC2 instances to a single EFS

EC2 Placement Groups: An EC2 Placement group is a logical grouping of ec2 instances within a single availability zone. It is recommended for applications for low latency, high network throughput or both. These ec2 instances are placed in a low latency, 10 gbps network connection.

They can’t be spanned across multiple availability zones.
Name of the placement group should be unique.
Only certain type of EC2 instances can be part of EC2 placement groups
AWS recommends to have homogenous instances to be part of a placement group
can’t merge placement groups and cannot existing instances in to placement group

AWS Lambda:
￼
Languages supported by Lambda: Node.js, java, c#, python
Lambda Pricing: Request based and Duration based


Route 53: DNS port number is 53, thats why it is called Route53

SOA record: Start of Authority Record
NS record: Name Server Record


Two default records are create for each hosted zone. They are NS record and SOA record

Route53 routing policies:
Simple - default routing policy, best for single resource 
Weighted - splits your traffic based on different weights assigned
Latency - allows to route your traffic to lowest network latency
Failover - allows to create a active/passive setup. when the active fails, it failover to passive setup
Geolocation - allows routing the traffic based on geographic location of the users

Max Domain names by default are 50. But this can be changed by contacting AWS support
AWS Suport Level: Basic, Developer, Business, Enterprise.

Databases:
RDS: Relational Database service: SQL, MySQL, Oracle, MariaDB, PostgreSQL, Aurora——> tables, rows and columns
Mainly for OLTP - Online Transaction Processing

DynamoDB: NoSQLService: CouchDB, MongoDB are nosql examples ——> collections, documents, key pairs


Elasticache: In-memory cache cloud service : Memcache, Redis (available in AWS)

Redshift: data-ware housing —> Used for business intelligence. Tools like cogno, jaspersoft, sql server reporting services etc. These are used to pull up large data sets. Mainly for OLAP - Online Analytics Processing.

DMS: Database Migration service


RDS: Two types of backups—> 
1) Automated Backups 
Enabled by default
Default retention period is 7 days. Retention period is 1 - 35 days max.
backups are stored in S3

2) Database Snapshots
are manual backups ie., they are user initiated
will exist even after the res instances are deleted (unlike the automated backups)

Restoration: after restoring the backups (both types) the result will a new RDS instance with a new end point.

Encryption  at rest is supported for MySQL, Oracle, SQLServer, PostgreSQL & Maria DB.
Encryption is done using AWS - KMS.
Once a RDS instance is encrypted its underlying storage, snapshots, backups, and read replicas are also encrypted.
We cannot encrypt a existing DB, it has to be enabled when and RDS instance being created

Multi-AZ: Multi-AZ has exact copy of prod db in another availability zone. AWS handles the replication & the failover is automatic Used only for disaster recovery only. For better performance use read replicas.
Achieved using synchronous replication. 
Multi-AZs are available for MySQL, Oracle, SQLServer, PostgreSQL & Maria DB.

Read-replicas: read only copies of your database. We can also have read-replica of a read-replica.
Achieved using asynchronous replication.
Used for scaling. Not for DR.
Must have the automated backups turned on to deploy a read replica.
Max upto 5 copies of read-replicas are allowed
MySQL, MariaDB and PostgreSQL are available for Read-Replicas feature.
Each read-replica has its own end-point. 
Cannot have Read-replicas that have multi-AZ. But we can read-replicas of multi-AZ databases.
Read-replicas can be promoted to be their own databases. this breaks replication.
We can do a read-replica in a second region for MySQL and MariaDB but not for a PostgreSQL.


DynamoDB: offers push button scaling which means we can scale the db on fly without downtime

RDS: scaling with RDS is not so easy. Usually we need to go for a bigger size or add a read replica to scale.
also we can scale in terms of reads not writes

Dynamo DB: NoSQL DB Service for applications support both document and key-value data models.
stored only on SSDs
spread across 3 geographically distinct data centers
eventual consistent reads (default) - read consistency after a second - can wait scenario
strongly consistent reads - read consistency after all written before a read ——can’t wait scenario

DynamoDB pricing: 
provisioned throughput pricing:
write throughput of 0.0065 dollars for every 10 units
read throughput of 0.0065 dollars for every 50 units

Storage cost of $ 0.25 gb per month

20 on demand and 20 reserved instances  are allowed per region
Only 5 elastic IPs per region are allowed
Cloudwatch metrics are aggregated at 1-minute intervals 
Can see metrics for deleted instances for up to 2 weeks
Instances will be deleted if the auto scaling group is deleted
Automatic backups should be enabled for read replicas - RDS
Aurora read replicas - 15
RDS read replicas - 5
RDS supports cross region replicas
Aurora has 6 copies of your db in 3 azs
During a failover, CNAME record is updated to a healthy db copy

Synchronously replicated -RDS 
We can Force a failover 


Steps to allow traffic:

Create a VPC
Create a internet gateway and attach to the vpc
Create public and private subnets
Enable auto assign options address for public subnet
Create custom route table and allow traffic from world to internet gateway we just created
associate the public subnet to the custom route table
Create ec2 instances in public and private subnets


Create a NAT gateway 
Add this to the default route table. 0.0.0.0 to the nat gateway
