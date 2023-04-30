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




