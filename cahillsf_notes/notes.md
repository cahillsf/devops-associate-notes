## hack
- https://a.cl.ly/5zuj9GlE

## regions
- broken into AZs -- each region is broken into 3 (up to 6) 
- points of presence (edge location) - 205 edge locations


### how to choose a region:

- complicance (data location)
- proximity (latency for users)
- AWS service availability
- pricing (varies per region)




## IAM

- principle of least privilege
- MFA options:
	- Virtual MFA Device
	- U2F (Universal 2nd factor security key)
	- Hardware key fob MFA device
	- GovCloud (US) specific Hardware key fob MFA device
- Three ways to access AWS:
	- AWS mgmt console (password + MFA)
	- CLI (access keys)
	- SDK (access keys)

- access keys are generated through the console
- users manage their own access keys

### IAM Roles for Services:
- can create roles for AWS services (common roles=ec2/lambda)

### IAM Security Tools (Audit):

- IAM Credentials Report (account-level)
- IAM Access Advisor (user-level)

### Groups
- only contain users (do not contain other groups)

### Policies
- JSON doc that outlines permissions for users or groups


## EC2 

### Naming conventions:
- <instance_class><generation>.<size_within_class> (e.g m5.2xlarge)
- max instance size: `12.3 TB, 448 vCPUs`
- smalles: `0.5G RAM, 1 vCPU`

### Types:
- general purpose:
	- great for diversity of workloads (web servers or code repos)
	- balance b/w compute/mem/networking
- compute optimized:
	- compute intensive tasks that require high performance processors
		- batch processing workloads
		- media transcoding
		- high perf web servier
		- high perf computing (HPC)
		- scientific modeling / ML
		- dedicated gaming servers
- memory optimized: 
	- fast performance for workloads that process large data sets in memory
		- high perf relational/non-relational DBs
		- distributed web scale cache stores 
		- in-mem DBs optimized for business intelligence (BI)
		- applications performing real-time processing of big unstructured data
- storage optimized: 
	- great for sorage-intensitve tasks that require high, sequential read and write access to large data sets on local storage
		- high frequency online transaction processing systems (OTLP)
		- relational and NoSQL DBs
		- cache for in-mem DBs (i.e. Redis)
		- data warehousing appls
		- distributed file systems

### Security Groups
- fundamental of network security in AWS
- control how traffic is allowed into or out of EC2s
- security groups contain only `allow` rules
- SG rules can reference by IP or security group
- act as a "firewall" and they regulate:
	- access to ports
	- authorized IP ranges (v4 and v6)
	- control of inbound network (from other to instance)
	- control of outbound network (from instance to other)
- SGs can be attached to multiple instance (instance can also have multiple SGs)
- SGs are region/VPC specific
- good to maintain one separate SG for SSH access
- connection refused indicates app error - timeout is often a SG issue
- all inbound traffic is `blocked` by default
- all outbound traffic is `authorized` by default
- SGs can reference other SGs

#### Common ports
- 22 = SSH
- 21 = FTP 
- 22 = SFTP
- 80 =  HTTP
- 443 = HTTPS
- 3389 = RDP

#### EC2 instance purchasing options

- on-demand - short workload, predictable pricing, pay by second
- Reserved instances (1 & 3 yrs) - steady state usage apps
	- reserved instances - long workloads
	- convertible reserved instance - long workloads with flexible instances
- Savings plans (1 & 3 yrs) -  commitment to amt of usage (long workload)
	- locked to specific instance family & AWS region
- Spot instance  - short workloads, cheap, can lose instances (less reliable)
	- up to 90% discount
	- use cases should be resilient to failure
- Dedicated Hosts - book an entire physical server, control instance placement
	- allows you to address compliance requirements and use existing server-bound software licenses
	- Purchasing: on-demand or reserved 
	- the MOST EXPENSIVE option
- Dedicated instances -  no other customers will share your hardware
	- may share hardware with instances in the same account
	- no control over instance placement
- Capacity reservations  - reserve capacity in a specific AZ for any duration
	- reserve on-demand instance in a specific AZ for any duration
	- no time commitment, no billing discounts
	- you're charged at on-demand ware whether you run instances or not
	- suitable for short-term uninterrupted workloads that need to be in a specific AZ


### EBS (Elastic Block Store)
- network drive (not a physical drive) you can attach to your instances while they run
- allows data persistence even after instance termination
- bound to a specific AZ
	- can move a volume across AZs per snapshot
- need a provisioned capacity
- instances connect to EBS via network connections
- EBS volumes do not ~need~ to be attached to an instance
- Delete on Termination attribute 
	- determines whether the volume is deleted on instance termination
	- by default Root volume is deleted on termination
	- any other attached instances are ~not~ terminated on deletion by default
- spanshots:
	- backup your volume at a point in time
	- not necessary to detach to snapshot, but recomended
	- can be copied across AZ or region
	- Snapshots can be moved to `snapshot archives` for cheaper storage (24 - 72 hours for restoring)
	- Receycle bins can be setup (rules to retain deleted snapshots for 1 day - 1 yr)
	- FSR (fast snapshot restore) - paid feature to force full initialization of snapshot to have 0 latency on first use


### AMIs
- AMIS are region specific (can be copied across regions)
- 3 categories:
	- public AMI (AWS provided)
	- custom AMI (user built and maintained)
	- marketplace AMI 

### EC2 Instance Store
- high performances hardware disk 
- better I/O performance
- lose their storage if they're stopped
- good for buffer/cache/scratch/temp content
- risk of data loss if hardware fails
- backups and replication are user responsibility

### Types of EBS volumes
- SDD (can be used for boot volumes):
	- gp2/3: gen purpose SSD vol that balance price/performance (1GiB - 16 TiB)
		- gp3:
			- baseline of 3,000 IOPS and throughoutput of 125MiB/s
			- can increase iOPS up to 16,000 and throughput up to 1000 MiB/s (independently)
		- gp2:
			- burst IOPS to 3,000
			- size of the volume and IOPS are linked, max IOPS is 16,000
			- 3 IOPS / GB -- at 5,334 GB you have maxed IOPS
	- io1/2: highest performance SSD for mission crit low/high throughput workload (Provisioned IOPS SSD)
		- great for database workloads
		- apps that need more than 16,000 IOPS
		- 4 GiB - 16TiB
		- max PIOPS 64,000 for Nitro EC2 (32,000 for other)
		- can increase PIOPS independently from storage size
		- io2 block express (4 GiB - 64 TiB)
			- sub millisecond latency
			- max PIOPS: 256000 with and IOPS:GiB ration of 1,000:1
		- support multi-attach
- HDD options
	- 125 GiB - 16 TiB
	- st1 (throughput optimized HDD): low cost HDD for frequently accessed, throughput intensive workloads
		- max throughput 500 MiB/s - max IOPS 500
	- sc1 (Cold HDD): lowest cost HDD for less frequently accessed workloads
		- max throughput 250 MiB/s - max IOPS 250
	- only SDD can be used for boot disks

### EBS Multi-attach
- available for io1/io2
- attach the same EBS volume to multiple EC2 instances in the same AZ
- each instance has full read/write permissions to the high-performance volume
- use case:
	- achieve higher application availability in clustered linux applications (eg: Teradata)
	- apps must manage concurrent write operations
- up to 16 EC2 instances can be attached at a time
- must use a file system that's cluster-aware (not XFS, EXT4)

### EFS (elastic file system)
- managed NFS that can be mounted on many EC2
- EFS works with Ec2 instances in multiple AZs
- highly available, scalable, expensive, pay-per-use
- use cases: content mgmt, web serving, data sharing, wordpress
- uses NFSv4.1
- uses security group to control access to EFS
- compatible with linux (not windows)
- encryption at rest using KBM
- uses a POSIX fil system that scales automatically (no capacity planning)
- Perfomance tiers:
	- EFS Scale:
		- 1000s of concurrent NFS clients, 10GB+/s throughout
		- grow to petabyte-scale network file system
	- Performance mode (set at EFS creation time)
		- General purpose (default)
		- Max I/O - highly parallel, higher latency
	- Throughoput mode
		- Bursting: 1 TB  =  50 MiB/s + burst of up to 100 MiB/s
		- Provisioned - set your throughout regradless of storage size
		- Elastic - atuomatically sclaes throughput up or down based on your workfload
			- up to 3 GiB/s for reads and 1 GiB/s for writes
			- best for unpredicatble workloads
- Storage Classes:
	- Storage Tiers (lifecycle mgmt feature)
		- Standard (frequently accessed)
		- Infrequent access
			- cost to retrieve files but lower price to store
			- enable EFS_IA with a lifecycle polcicy
	- Availability and durability
		- standard: multi-az (prod)
		- one zone (one AZ): great for dev, backup enabled by default, compatible with IA (EFS One Zone - IA)

### EBS v EFS Highlights
- EBS:
	- one instance (except multi-attach io1/io2)
	- AZ specific
	- gp2: IO increases along with disk size
	- io1: can increase IO independently 
	- offers snapshot feature to migrate acrozz AZ
	- EBS backups use IO and you shouldn't run them while handling high traffic
	- root EBS volumes of instances get terminated by default along with EC2 instance termination (can be disabled)
- EFS
	- can be mounted to 100s of instances across AZs
	- only for linux (POSIX) instances
	- higher pricepoint that EBS
	- can leverage EFS-IA for cost savings

## ELB and ASG

- scalability: application/system can handle greater loads by adapting
- two kinds of ~scalability~:
	- vertical
		- increasing the size of the instance
		- common for non-distributed systems (i.e. DEB)
		- inherent limit to how much you can vertically scale (hardware limit)
	- horizontal (elasticity)
		- increasing the number of instances/systems for your application
		- implies distributed systems
		- common for web apps/modern apps
		- easy to horizontally scale with cloud providers
		- ASGs and Load balancing enable this
- scability is related to, but distinct from HA
- ~high availability~:
	- goes hand in hand with horizontal scaling
	- means running your app/system in at least 2 AZs
	- goal is to survive a DC outage
	- HA can be passive (RDS multi-AZ) or active (horizontal scaling)
	- multi-AZ ASGs and Load balancing enable this

### ELB (managed service)
- spread load across multiple downstream services
- expose a single point of access (DNS) to your ap
- allows you to seamlessly handle failures of downstream instances
- accomplished helath checks to your instances
- provide SSL termination for website
- enforce stickiness with cookies
- HA across zones
- separate public traffic from private
- AWS responsibilties
	- guaranteed uptime
	- no upgrades, maintenance required from user
	- minimal configuration
- integrated with many AWs Offerings 
	- EC2, ASGs, ECS
	- AWS Cert Managers (ACM), Cloudwatch
	- Route 53, AWS WAF, AWS Global Accelerator
- 4 types of LBs:
	- Classic Load Balancer (v1 old gen 2009 - CLB)
		- HTTP, HTTPS, TCP, SSL (secure TCP)
	- Application Load Balancer (v2 new gen 2016 - ALB)
		- HTTP, HTTPS, WebSocket
	- NLB (v2- new generation 2017 - NLB)
		- TCP, TLS (secure TCP), UDP
	- Gateway Load Balancer (2020 - GWLB)
		- operates at layer 3 (network layer) - IP protocol
- can be set up as internal (private) OR external (public)


#### ALB
- layer 7 protocol (HTTP)
- support load balancing to multiple HTTP applications across machines
- supports load balancing to mutiple applications on the same machine
- support for HTTP/2 and WebSocket
- support redirects (i.e. HTTP to HTTPS)
- support routing tables to different target groups
	- routing based on URL path
	- routing based on HOSTNAME in url
	- roubting based on QueryString or Headers
- great fit for microservices and container-based apps
- has a port mapping feature to redirect to a dynamic por tin ECS

- what are target groups? can be any of the following:
	- EC2 instances (can be managed by an ASG) - HTTP
	- ECS tasks (managed by ECS) - HTTP
	- Lambda functions - https request is translated to a JSON event
	- IP addresses (must be private IPs)
	- ALB can route to multiple target groups)
	- health checks are at the target group level
- ALBs have a ~fixed hostname~
- app servers don't see the IP of the client directly 
	- ~true~ ip of the client is inserted into header `X-Forwarded-For`
	- we can also get port (`X-Forwarded-Port`) and proto (`X-Forwarded-Proto`)


#### NLB (v2)
- NLBs operate on Layer 4:
	- forward TCP/UDP traffic 
	- can handle millions of reqs/sec
	- low latency (~100ms) vs 400ms for ALB
- has 1 static IP per AZ (supports assiging Elastic IP)
- target groups can be:
	- EC2 instances
	- IP addresses (must be private IPs)
	- ALB
	- health checks support TCP/HTTP/HTTPS protocol

#### Gateway Load Balancer
- Layer 3 protocol (network layer - IP packets)
- deploy, scale and manage a fleet of 3rd party network virtual applicances in AWS
- examples: firewalls, intrusion detection, prevention systems, deep packet inspection system, parload manupulation
- combines the following functions:
	- Transparent Network Gateway (single entry/exit point for all traffic)
	- Load Balancer - distributes traffic to your virtual appliances
- uses the GENEVE protocol on port 6081
- target groups:
	- EC2 instances
	- IP addresses (private IPs)

#### Sticky Sessions (Session Affinity)
- it is possible to implement stickiness so that the same client is always redirected to the same instance behind a load balancer
- works for CLB, ALB, NLB
- the cookie used for stickiness has an expirate timestamp that your can control
- use case: make sure the user doesn't lose session data
- enabling stickiness may bring imbalance to the load over the backend ec2 instances
- cookie names:
	- app-based cookie:
		- custom cookie
			- generated by the target
			- can include any custom attributes required by the application
			 - cookie name must be specified individually for each target group
			 - AWSALB, AWSALBAPP, AWSALBTG cannot be used as cookie names
		- application cookie
			- generated by the load balancer
			- cookie name is AWSALBAPP
	- duration-based cookies:
		- cookie is generated by the load balancer
		- cookie name is AWSALB (alb) or AWSELB (clb)

#### Cross-Zone Load Balancing
- each LB instance distributes evenly across all registered instances in all AZ
- (w/o CZ LB) - requests are distributed in the instances of the node of the ELB
- ALB 
	- cross AZ is enabled by default (can be disabled at TG level)
	- no charges for inter AZ data
- NLB/GWLB
	- cross AZ is ~disabled~ by default
	- you pay for inter AZ data
- CLB
	- cross AZ is ~disabled~ by default
	- no charges for inter AZ data

#### SSL/TLS Basics
- SSL cert allows traffic b/w your clients and your LB to be encrypted in transit (in-flight encryption)
- SSL = secure sockets layer
	- TLS = transport layer security (newer version of SSL)

#### Certs and Load Balancer
- LB uses an X.509 cert (SSL/TLS server cert)
- you can manager certs using ACM or upload your own certs
- HTTPS listener
	- must specify a default cert
	- can add an option list of certs to support multiple domains
	- clients can use SNI (server name indication) to specify the hostname they reach
	- ability to specify a security policy to support older versions of SSL/TLS (legacy clients)
- SNI solves the problem of loading multple SSL certs onto 1 web server to serve multiple web sites
- it's a newer protocol that requires the client to indicate the hostname of the target server in the initial SSL handshake
- server will then find the correct cert, OR fallback to the default
- works for ALB & NLB (newer gen) and Cloudfront
	- not supported in CLB

#### Connection Draining
- feature naming:
	- Connection Draining (CLB)
	- Deregistration Delay (ALB/NLB)
- time to complete "in-flight requests" while the instance is de-registering or unhealthy
- stops sending new request to the ec2 instances which is de-registering
- can be set to b/w 1-3600 seconds (default=300s)
- can be disabled (set to 0)
- set to a low val if your requests are short

#### ASG
- features:
	- scale out to match an increased load
	- scale in to match a decreased load
	- ensure a max/min number of instances running
	- automatically register new instances to an LB
	- recreate an instance if a previous one is terminated (i.e. deemed unhealthy)
	- free to use (you pay for the underlying compute resources)
- attributes:
	- launch template:
		- AMI/instance type
		- User data
		- EBS vols
		- Security groups
		- SSH key pair
		- IAM role
		- networking + subnets
		- LB info
	- min size/ max size/ initial capacity
	- scaling policies
	- can scale based on cloudwatch metrics

##### Dynamic Scaling Policies
- target tracking scaling
	- simplest and easiest to setup 
	- i.e. avg CPU should = 40%
- simple/step scaling
	- when an alarm is trigerred, then add 2 unites
- scheduled actions
	- anticipate a scaling based on a known usage pattern
- predictive scaling
	- continuously forecast load and schedule scaling ahead

- good metrics for scaling:
	- CPUUtilization (avg CPU)
	- RequestCountPerTarget (make sure the number of reqs/ instance is stable
	- Average Network In/Out (if your app is network bound)
	- or any custom metric
- Scaling Cooldowns:
	- after a scaling activity happens, you are in a cooldown period (default 300 seconds)
	- advice: use a ready-to-use AMI to reduce config time to serve reqs faster and reduce the cooldown period
- Instance refresh:
	- goal: update launch template and then recreate all EC2s
	- set a min of healthy percentage and instances will be cycled 
	- set a warm-up time as well

## RDS
- relational database service
- managed DB service (SQL as query language)
- allows you to create DBs in the cloud that are managed by AWS.  Offers the following flavors:
	- postgres
	- mysql
	- MariaDB
	- Oracle
	- Microsoft SQLServer
	- Aurora (prioprietary AWS db)
- advantages:
	- managed service (automated provisioning & OS patching)
	- coninuous backups and restore to specific timestamp
	- monitoring dashboards
	- read replicas for improved read performance
	- multi-AZ available for DR
	- maintenance windows for upgrades
	- horizontal and vertical scaling capabilities
	- storage backed by EBS (gp2 or io1)
- caveat: you cannot SSH into your instances as they are offered as a ~managed service~

### Storage Auto Scaling
- helps increase your storage on RBS DB dynamically
- when RDS detects you are running out of free DB storage, it scales automatically (you can avoid manually scaling your DB storage)
- you have to set a Maximum Storage Threshold
- automatically modify storage if:
	- free storage < 10% allocated storage
	- low-storage lasts at least 5 mins
	- 6 hours have passed since last modification
- useful for apps with unpredictable workloads
- support all RDS db engines

### Read Replicas and Multi-AZ

#### Read Replicas
- can created up to 15 read replicas for read scalability (within AZ, cross AZ, or cross region)
- replication is ASYNC, so reads are ~eventually~ consistent
- replicas can be promoted to their own DB
- applications need to update the connection string to leverage read replicas
- read replicas are used for SELECT (read) operations only

##### Network Cost of Read Replicas
- in AWS there's generally a network cost when data goes from one AZ to another
- for RDS read replicattions:
	- same region = NO FEE
	- cross region = CHARGED NETWORK COSTS

#### RDS Multi AZ (DR)
- SYNC replication
- one DNS name (automatic app failover to standby)
- increase availability
- failover in case of loss of AZ, loss of network, instance or storage failure
- no manual intervention in apps
- ~not~ used for scaling
- NOTE: read replicas can be setup as multi AZ for DR
- common q: how to go from single-AZ to multi-AZ
	- this is a zero downtime operation
	- just click on modify for the DB and enable multi-AZ
	- behind the scenes:
		- snapshot of original DB is taken
		- new DB is restored from the snapshot in a new AZ
		- synchronization is established bw the two DBS

- master
