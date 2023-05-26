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

### Aurora
- proprietary tech from AWS (not open source)
- PG and MySQL are both supported as Aurora DB (your drivers will work as if aurora was a PG or MySQLdb)
- AWS cloud optimized and claims 5x performance improvement over MySQL on RDS, over 3x performance of Postgres on RDS
- storage grows automatically in increments of 10GB up to 128 TB
- can have up to 15 replicas and replication is faster than MySQL (sub 10ms lag)
- failover is instantaneous (its HA native)
- costs 20% more than RDS (20% more) but is more efficient
- automatic fail-over
- backup and recovery
- isolation and security
- industry compliance
- push-button autoscaling
- automated patching with zero downtime
- advanced monitoring
- routinge maintenance
- backtrack: restore data at any point of time without using backups

##### Aurora HA and Read Scaling
- 6 copies of your data across 3 AZ
	- 4 copies out of 6 needed for writes
	- 3 copies out of 6 needed for reads
	- self healing with peer-to-peer replications
	- storage is striped across 100s of volumes
- one Aurora instance takes writes (master)
- automated failover for master in less than 30 seconds
- master + up to 15 aurora read replicas serve reads
- support for cross region replication

##### Aurora DB Cluster
- Writer Endpoint (pointing to master)
- can configure autoscaling for read replicas
	- Reader Endpoint (Connection load balancing to read replicas)

### RDS & Aurora Security
- at-rest encryption:
	- DB masters and replicas encryption using AWSK KMS must be defined at launch time
	- if the master is not encryped, the read replicas canot be encrypted
	- to encrypt an un-encrypted DB, use a DB snapshot and restore as encrypted
- in-flight encryption:
	- TLS_ready by default, use the AWS TLS root certificates client-side
- IAM authentication: can be used instead of username/pw
- Security Groups control access to RDS/Aurora DB
- no SSH available (except on RDS custom)
- audit logs can be enabled and sent to CW logs for longer retention

### Amazon RDS Proxy
- fully managed DB proxy for RDS
- allows apps to pool and share DB connections established with the database
- improving DB efficiency by reducing the stress on DB resources and minimize connections/timeouts
- serverless, autoscaling, highly available (multi-AZ)
- reduced RDS and Aurora failover time by 66%
- support MySQL, PostgreSQL, MariaDB, SQLServer and Aurora (MySQL, PostgreSQL)
- no code changes required from most apps
- enforce IAM auth for DB and securely store credentials in AWS secrets manager
- RDS proxy is *never* publicly accessibly (must be accessed from VPC)

## Elasticache
- managed Redis or Memcached
- in-mem databases with high performance, low latency
- helps reduce load off of DBs for read intensive workloads
- helps make your app stateless
- aws takes care of OS maintenance, patching, optimizations, setup, config, monitoring, failure recovery and backups
- using elasticache involves *heavy app code changoses*

### Solution Architecture
- use case 1: relieve RDS load
	- app queries Elasticache:
		- if not available, get from RDS and store in Elasticache (cache miss):
			- Read from DB
			- then write to the cache
		- if available, return it (cache hit)
		- cache must have an invalidation strategy to ensure only the most current data is cached
- use case 2: user session store
	- user logs into any of the apps
	- app writes session data into elasticache
	- user can hit another instance of the app, and the instance retrieves the cached sessions data and is already logged in

### Redis v Memcached
- Redis:
	- multi AZ with auto-failover
	- read replicas to scale reads and have HA
	- data durability using AOF (append only file) persistence
	- backup and restore features
	- supports sets and sorted sets
- Memcached
	- multi-node for paritioning of data (sharding)
	- no HA 
	- non-persistent
	- no backup and restore functionality
	- multi-threaded architecture

### Implementation considerations:
- https://aws.amazon.com/caching/best-practices/
- is it safe to cache data?
	- data may be out of data, but will eventually reach consistency
- is caching effective?
	- pattern: data changing slowly, few keys are frequently needed
	- anti-patterns: data changing rapidly, all large key space are frequently needed
- is data well structured for caching?
	- example: key value caching, or caching of aggregation results
- lacy loading/cache aside is easy to implement and works for many situations as a foundation, esp on the read side
- write through is usually combined with lazy loading if the use case is appropriate
- setting a TTL is usually not a bad idea (except when using write-through)
	- specifics depend on your use case
- only cache the data that makes sense

#### Lazy-loading/cache-aside/ lazy population
- check if the ID is cached
- if not, run the DB query, then cache the data

##### Pros:
- only requested data is cached
- node failures are not fatal (just increased latency to warm up the cache)

##### Cons:
- cache miss penalty that results in 3 round trips resulting in noticeable delay for the cache
- stale data: data can be updated in the db, and outdated in the cache

#### Write Through 
- add or update cache when DB is updated
- 

##### Pros:
	- data in cache is never stale, reads are quick
	- write penalty vs read penalty (each write requires 2 calls)
		- could be better from a user perspective (writes are understood to take more time vs a fetch)

##### Cons:
- missing data until is is added/update in the DB
	- mitigation is to implement lazy loading as well
	- but risk of cache churn - a lot of the data written to the cache will never be read

#### Cache Evictions and TTL (time-to-live)
- cache eviction can occur in three ways:
	- you delte the item explicitly in the cache
	- item is evicted because the memory is full and its not recently used (Least Recently Used)
	- you set an item TTL
- TTL are helpful for any kind of data
	- leaderboards 
	- comments
	- activity streams
- TTL can range from few seconds to hours or days
- if too many evictions are happening due to memory occupation, you should scale up or out

### Amazon MemoryDB for Redis
- Redis-compatible, durable and in-mem database service
- ultra-fast performance with over 160 million reqs/sec
- durable in-mem data torage with multi-AZ transactional log
- scale seamlessly from 10s GBs to 100s of TBS in storage
- use cases: web/movile apps, online gaming, media streaming

## Route 53

### DNS Background
- DNS = domain name syste which translate human friendly hostnames into the machine IP address
- domain registrar: Route 53, GoDaddy,
- DNS record types: A, AAAA, CNAME, NS
- name server: resolves DNS queries (authoritative or non-authoritative)
- Top Level Domain (TLD): .com, .us, .in
- Second Level Domain (SLD): amazon.com, google.com
- request structure:
	- Local DNS Server: (assigned or managed by your company or assigned by your ISP)
		- If not found here, goes up to the ROOT
	- Root DNS Server: (managed by ICANN) directs to the TLD
	- TLD DNS Server (managed by IANA) directs to the SLD
	- SLD DNS Server (managed by Domain Registrar)	
		- returns the Record type and the IP
	- Local Server then caches the result

### Product Overview
- a highly available, scalable, fully managed and authoritative DNS
	- authoritative = the user (me) can update the DNS records
- also a Domain Registrar
- ability to check the health of your resources
- only AWS service that provides 100% availability SLA

### Records
- how you want to route traffic for a domain
- each record contains:
	- Domain/subdomiain name (example.com)
	- record type (A, AAAA)
	- value (IP address)
	- routing policy (how route 53 responds to queries)
	- TTL - amount of time the record is cached at DNS resolvers
		- High TTL= less traffic on route 53 & possibly outdated records
		- Low TTL = higher traffic on route 53, records are outdated for less time, easy to change records
		- TTL is mandator for all records (except ALIAS)
- Route 53 supports the following record types
	- essential: A/ AAAA/ CNAME/ NS
	- advanced: CAA/ DS / MX/ NAPTR

#### Record Types
- A - maps a hostname to an IPv4 address
- AAAA - maps a host name to an IPv6 address
- CNAME - maps a hostname toa nother hostname
	- the target is a domain name which must have an A or AAAA record
	- you cannot create a CNAME record for the top node aof a DNS namespace (Zone Apex)
		- i.e., you can't create for `example.com`, but you can create for `www.example.com`
- NS - name servers for the Hosted Zone 
	- control how traffic is routed for a domain

#### Hosted Zones
- a container for records that define how to youre traffic to a domain and its subdomains
- public hosted zones - contains record that specify how to route traffic on the internet (public domain names)
- private hosted zones - contains records that specify how your route traffic within one or more VPCs (private domain names)
- cost: $0.50 per month per hosted zone

#### CNAME vs Alias
- AWS resources expose an AWS hostname
- CNAME points a hostname to any other hostname
- Alias
	- maps a hostname to an AWS resource
	- works for root domain (zone apex) and non-root comain
	- free of charge
	- always type A / AAAA
	- alias records targets: ELB, CloudFront, API gateways, S3, Elastic Beanstalk, VPC interface endpoints, Global Accelerator, Route 53 record (same hosted zone)
		- cannot set an ALIAS record to an EC2 DNS name

### Routing Policies
- define how route 53 responds to DNS queries
- supported routing policies in route 53:
	- simple:
		- route traffic to a single resource
		- can specify multiple values in the same record
			- if multiple values are returned, a random one is chosen by the client
		- with Alias enabled, only 1 AWS resource can be chosen
		- cannot be associated with health checks
	- weighted:
		- control the % of the requests that go to each specific resource
		- assign each record a relative weight
		- traffic % = (weight for a specific record) / (sum of all weights for all records) 
		- DNS records must have the same name and type
		- can be associated with health checks
		- use cases: load balancing b/w regions, testing new app versions, etc
		- weight of 0 = no traffic
		- if all records have 0, then all records are returned equally
	- failover:
		- add a mandatory health check for the *primary* EC2
		- if that fails, the traffic is directed to the *secondary* isntance
	- latency based:
		- redirect to the resoure that has the least latency (closest to us)
		- helpful when latency for users is high priority
		- latency is based on traffic bw users and AWS regions
		- Germany users may be directed to US (if it's the lowest latency)
		- can be associated with health checks (failover capacity)
	- geolocation:
		- routing is based on user location
		- specify location by continent, country, or US State (if there's overlap, most prcise location is selected)
		- should create a *Default* record in case there is no match on location
		- use cases: website localizations, retrict content distribution, load balancing
		- can be associated with health checks
	- multi-value answer:
		- use when routing traffic to multiple resources
		- route 53 returns multiple values/resources
		- can be associated with Health Checks (return only values for healthy resources)
		- up to 8 healthy records are returned for each multi-value query
		- multi-value is *not* a substitute for an ELB (client side decision for routing)
	- geoproximity (route 53 traffic flow - Traffic Policy record):
		- route traffic to resources based on the geographic location of users and resources
		- ability to shift more traffic to resources based on the defined bias
		- to change the size of the geographic region, specify *bias* valus:
			- to expand (1 to 99) - more traffic to the resource
			- to shrink (-1 to -99) - less traffic to the resource
		- resources can be:
			- AWS resources (specify AWS region)
			- non-AWS resources (specify latitude and longitude)
		- must use route 53 traffic flow in order to leverage this feature
	- IP based routing:
		- routing is based on clients IP addresses
		- you provide a list of CIDRs for your client and the corresponding endpoints/locations (user-ip-to-endpoint mappings)
		- use case: optimize performance, reduce network costs, etcs
		- e.g. route users from a particular ISP to a specific endpoint

### Health Checks
- http health checks are only for public resources
- healthcheck can tie in with Automated DNS Failover:
	- health checks that monitor an endpoint (app, server, or other AWS resource)
		- 15 global health checkers will check the endpoint health
		- healthy / unhealthy threshold (3 by default)
		- interval = 30 seconds (can set to 10 sec == higher cost)
		- supported protocol: TCP, HTTP, HTTPS
		- if > 18% if the health checkers report the endpoint is helathy, route 53 considers it healthy (otherwise unhealthy)
		- ability to choose which locations you wante route 53 to use
		- healtch checks pass when the endpoint responds with 2xx and 3xx status codes
		- health checks can be setup to pass/fail based on the text in the first 5120 bytes of the repsonse
		- you must configure your router/firewall to allow incoming requests from route 53 health checkers
	- health checks that monitor other health checks (calculated health checks)
		- combine the results of multiple health checks into a single health check using boolean logic
		- can monitor up to 256 child health checks
		- specify how many of the health checks needs to pass to make the parent pas
		- use case: perform maintenance to your site/servers without causing all the health checks to fail
	- health checks that monitor cloudwatch alarms (e.g. throttles on DynamoDB, alarms on RDS, custom resources, etc)
		- can help with monitoring the health of private resources through CW alarms
- health checks are integrated with cloudwatch metrics

### Traffic Flow
- simplify the process of creating and maintaining records in large and complex configuratinons
- visual editor to manage complex routing decision trees
- configurations can be saved as `Traffic Flow Policies`
	- can be applied to different route53 hosted zones
	- supports versioning

### Domain Registrar vs DNS
- registrar typically will provide a DNS service to manage the records
- but you can use another DNS service to manage your DNS records (i.e. buy a domain from GoDaddy but use Route 53)
- would need to specify the AWS nameservers within the domain registrar and reate a hosted zone in route 53 

## VPC (private network)
- regional resource
- subnets allow you to partition your network inside your VPC
	- tied to specific AZs
- a *public subnet* is accessible from the internet
- a *private subnet* is not accessible from the internet
- to define access to the internet and b/w subnets we use *Route Tables*

### IGWs and NAT
- internet gateways allow our VPC instances to connect with the internet
	- public subnets have a route to the internet gateway
- NAT Gateways (AWS-managed) & NAT Instances (self-managed) allow your instances in your private subnets to access the internet while remaining private
	- we would create a NAT gateway within the public subnet, then create a route from the private subnet -> NAT gateway

### Network ACL & Security Groups
- NACL (network ACL)
	- a firewall which controls traffic from/to a subnet
	- can have ALLOW and DENY rules
	- are attached at the *subnet* level
	- rules only include IP addresses
	- stateless: return traffic must be explicitly allowed by rules
	- rules are evaluated in number order when deciding to allow traffic
	- inherited by all resources in the subnet
- SGs (security groups)
	- control traffic to and from an ENI/EC2 instance
	- can have onyl ALLOW rules
	- rules include IP addresses and other security groups
	- stateful: return traffic is automatically allows
	- all rules are evaluated before deciding to allow traffic
	- must be explicity associated with resources

### VPC Flow Logs
- capture information about IP traffic going into your interfaces
	- VPC flow logs
	- subnet flow lgos
	- ENI flow logs
- helps to monitor and troubleshoot connectivity issues
- captures network information from AWS managed interfaces as well (ELBs, Elasticache, etc)
- logs can be directed to S3, CW logs, Kinesis data firehose

### VPC Peering
- connect two VPCs, privately using AWS's network to make them behave as if they were in the same network
- must not have overlapping CIDRs
- peering is *not transitive* (must be established for each VPC that need to communicate with one another)

### VPC Endpoints
- allow you to connect to AWS services using a private network instead of the public www network
- enhanced security and lower latency to access AWS services
- VPC endpoint gateways (S3 and DynamoDB)
- VPC Endpint Interfaces (all other services)
- only used within your VPC

### Site to Site VPN & Direct Connect
- site to site VPN
	- connect an on-prem VPN to AWS
	- connection is encrypted
	- goes over the public internet
- Direct Connect (DX)
	- establish a physical connection b/w on-prem and AWS
	- connection is private, secure and fast
	- goes over a private network
	- takes a month+ to setup

### 3 tier architecture
- public subnet (ELB)
- private subnet (compute instances across 3 AZs)
- data subnet (RDS/Elasticache)
- LAMP (linux/apache/mysql/php) 
	- can add caching (Elasticache)
	- EBS for local storage

## S3
- uses:
	- backup and storage
	- disaster recovery
	- archive
	- hybrid cloud storage
	- app hosting
	- media hosting
	- data lakes and big data analytics
- object store in buckets (directories)
- buckets must have a globally unique name (across all regions/account)
- buckets are defined at the region level
- buckets must adhere to a naming convention
	- no uppercase/underscore
	- 3-63 letters
	- not an IP
	- must start w/ lowercase letter or number
	- must not start with prefix `xn--`
	- must not end with suffic `-s3alias`

### Objects
- objects have a Key which is the FULL path
- key is composed of prefix + object name
- no concept of "directories" within buckets, but the UI will make it seem like there are
- object values are the content of the body (max object size is 5TB)
	- more than 5GB must be a "multi-part" upload
	- recommended multi-part for files > 100 MB
	- helps to parallelize uploads
- objects have metadata (list of text key/value pairs)
	- user-defined metadata must begin with "x-amz-meta"
	- user-defined metadata keys are stored in lowercase
- tags
	- key/value pairs for object in S3
	- useful for finegrained permissions 
	- useful for analytics purposes
- YOU CANNOT SEARCH OBJECT METADATA OR OBJECT TAGS
	- instad, you must use an external DB as a search index such as DynamoDB
- version ID

### Bucket Policy
- user -based:
	- IAM policies
- Resource-based
	- Bucket Policies - bucket wide rules from the S3 console (allows cross-account)
	- Object ACL (access control list) - finer grain (can be disabled)
	- Bucket ACL - less common (can be disabled)
- Note: an IAM principal can access an S3 bobject if:
	- the user IAM permissions ALLOW it or the resource policy ALLOWS it
	- AND there's no explicit DENI
- Encryption : encrypt object in S3 using encryption keys
- JSON based policies
	- Resources
	- Effect (Allow/Deny)
	- Actions: set of APIs to allow/deny
	- Principal: the account/user to apply the policy to
- can use bucket settings for `Block Public Access` (can be set at the account level)

### Static Website Hosting
- s3 can host static websites for internet access

### Versioning
- protect against unintended delete
- easy to roll back to previous version
- any file that is not versioned prior to enabling versioning will have version `null`

### Replication (CRR & SRR)
- must enable versioning in source and destination buckets
- Cross-region replication (CRR)
- Same-region replication (SRR)
- buckets can be in different AWS accounts
- copying is async
- must give proper permissions to S3 
- use cases:
	- CRR - compliance, lower latency access, replication across accounts
	- SRR - log aggregation, live replication between prod and test accounts
- after you enable replciation, only new objects are replicated
- to replicate existing objects, use S3 Batch Replication
- for DELETE operations
	- can replicate delete markers from source to target (optional setting)
	- deletions with a version ID are not replicated (to avoid malicious deletes)
- there is not "chaining" of replication
	- if bucket 1 has replication into bucket 2, which has replication into bucket 3
	- objects created in bucket 1 are not replicated to bucket 3

### Storage Classes
- General Purpose
	- 99.99% availability
	- low latency and high throughput
	- can sustain 2 concurrent failures
	- use cases: big data analytics, mobile and gaming apps, content distro
- IA
	- requires rapid access when needed
	- lower cost (but comes with retrieval costs)
	- *Standard* IA (infrequent access)
		- 99.9% avail
		- use cases: disaster recovery, backups
		- min storage duration 30 days
	- *One Zone* - IA
		- high durability in single AZ; data is lost if AZ is destroyed
		- 99.5% avail
		- use cases: secondary copies of on-prem data, or data you can recreate
		- min storage duration 30 days
- Glacier
	- low-cost for archiving/backup
	- price for storage + retrieval 
	- *Glacier Instant Retrieval*
		- millisecnond retrieval 
		- great for quarterly accessed data
		- min storage duration of 90 days
	- Glacier Flexible Retrieval
		- min storage duration of 90 days
		- Expedited (1 - 5 mins), Standard (3 to 5 hrs), Bulk -free- (5 to 12 hrs) 
	- Glacier Deep Archive
		- min storage duration of 180 days
		- standard (12 hrs), bulk (48 hrs)
- Intelligent Tiering
	- small monthly monitoring and auto-tiering fee
	- objects move automatically bw access tiers based on usage
	- no retrieval costs
	- Frequent (default), Infrequent (30 days no access), Archive Instant Access (90 days no access), Archive Acess (optional - 90 to 700+ days), Deep Archive Access (option - 180 to 700_ days)

- Can move between classes manually or use Lifecycle configs

### Lifecycle Rules
* Transitions Actions - configure objects to transition to another storage class
	- move objects to standard IA class 60 days after creation
	- move to Glacier for archiving after 6 mos

* Expiration Actions - configure object to expire (delete) after some time
	- delete access logs after 365 days
	- delete old versions of files
	- delete incomplete multi-part uploads

* rules can be creates for a certain prefix OR certain object tags

### S3 Analytics
- help you decide when to transtion objets to the right storage class
- recommendations for Standard and Standard IA (not supported in One-Zone IA or Glacier)
- report is updated daily
- 24-48 hrs to start seeing data analysis
- good first step to put together lifecycle rules (or improve them) 

### S3 Event Notifications
- create notifications based on actions (S3:ObjectCreated, ObjectRemoved, etc)
	- notifications can be delivered to another service (i.e SNS or SQS or Lambda)
	- IAM policies need to be put in place to allow S3 to invoke events
	- event can also be delivered to EventBridge which allows extensible functionality
- object name filtering possible
- can create as many events as desired
- notifications typically deliver events in seconds but can sometimes take a minute or longer

### S3 - Baseline performance
- S3 automatically scales to high request rates with latency of 100-200 ms
- app can achieve at least 3500 PUT/COPY/POST/DELETE or 5500 GET/HEAD requests per second per prefix in a bucket
- no limits to the number of prefixes in a bucket


### S3 Transfer Acceleration
- increase transfer speed by transferring file to an AWS edge location, which will forward the data to the S3 bucket in the target region 
	- compatible with multi-part upload

### Byte-range fetches
- can parallelize GETs by requesting specific byte-ranges
- better resilience in case of failures
- can be used to speed up downloads
- can be used to retrieve only partial data (i.e. header)


### Durability and Availability
- Durability
	- high durability (11 9s) of objects across multiple AZs
	- same for all storage classes
- Availability
	- varies depending on storage class
	- standard has 99.99% availability

### S3 Select and Glacier Select
- retrieve less data using SQL by performing server-side filtering
- can filter by rows and columns (simply SQL statements)
- less network transfer, less CPU cost client-side

### Object Encryption
- 4 encryption methods:
	1. SSE (2x server-side encryption)
		- S3 managed keys (SSE-S3) - enabled by default
			- keys are handled, managed, and owned by AWS
			- encryption type is AES-256
			- must set the header "x-amx-server-side-encryption": "AES256" when uploading to the bucket
		- KMS keys (SSE-KMS)
			- create a manage your own key using KSM
			- audit key usage w/ CloudTrail
			- must set the header "x-amx-server-side-encryption": "aws:kms" when uploading to the bucket
			- limitation: you may be impacted by the KMS limits for the calls to `GenerateKeyData` (on upload) and `Decrypt` call on download
	2. SSE-C
		- SSE using keys fully managed by the customer outside of AWS
		- AWS does not store the encryption key
		- HTTPS must be used
		- encryption key must be provided in HTTP headers for every request made
		- only supported in CLI or SDK
	3. Client-side encryption
		- use client libraries such as S3 client side encryption lib
		- clients encrypt the data themselves prior to shipping to S3
		- clients must decrypt the data themselves when retrieving from S3
		- customer fully manages the keys and encryption cycle

- bucket policies can be used to force encryption by refusing API calls to PUT an object without encryption headers
- bucket policies are evaluated *before* default encryption


#### Encryption in transit
- in flight encryption (SSL/TLS)
- Amazon exposes 2 endpoints - HTTP/HTTPS
- HTTPS is recommended generall and mandatory for SSE-C
- can enforce encryption in transit by using a bucket policy to enforce `aws:SecureTransport = true`

### S3 CORS
- CORS (cross-origin resource sharing)
- Origin = scheme(protocol) + host (domain) + port
- browser based mechanism to allow requests to other origins while visiting the main origin
- the request wont be fulfilled unless the other origin allows for the requests using CORS header (e.g. `Access-Control-Allow-Origin`)
- if a client makes a cross-origin request on our S3 bucket, we need to enable the correct CORS headers
- you can allows for a specific origin or for `*` (all origins)

### MFA Delete
- force users to generate a code on an MFA device before completing important operations
- versioning must be enabled on the bucket
- only bucket owner (root account) can enable/disable MFA delete
- MFA will then be required to:
	1. permanently delete an object version
	2. suspend versioning on the bucket

### S3 Access Logs
- for audit purpose, you can enable access log collection for S3 buckets
- all requests will then be logged
- the target logging bucket must be in the same aws region
- do not set your logging bucket to be the monitored bucket (creates an infinite loop)

### S3 - Pre-signed URLs
- generate pre-signed URLS using s3 console, CLI or SDK
- URL expiration
	- S3 console (1 min - 12 hrs)
	- CLI - configure with `--expires-in` parameter in seconds (default 3600, max 168 hrs)
- users given a pre-signed URL inherit the permissions of the user that generated the URL for GET/PUT

### S3 Access Points
- can grant R/W permissions to bucket prefixes
- simplify security mgmt for S3 buckets at scale
- each access point has
	- its own DNS name (internet origin or VPC origin)
		- VPC origin - must create a VPC endpoint to access the acess point
		- VPC Endpoint policy must allow access to the target bucket and access point
	- an access point policy (similar to bucket policy)

### S3 Object Lambda
- use Lambda functions to modify the object before it is retrieved by the caller app
- create an S3 access point on top of the S3 bucket, then a lambda can be run in bw the S3 Access point and the S3 Object Lambda Access point
- use cases:
	- redact/enrich data being retrieve from S3 buckets
	- converting data formats
	- resize/watermark images on the fly using caller-specific details



## CLI/SDK/API
- use `--dry-run` to check command output and authorizations without actually running the command
- `aws sts decode-authorization-message --message <MESSAGE>` will allow you to get additional information about auth errors

### IMDS 
- AWS EC2 Instance Metadata Service allows instances to pull information about themselves
- url is: http://169.254.169.254/latest/meta-data
- you can retrieve the IAM role name from the metadata (but not the IAM policy)
- IMDSv1 accesses the URL directly
- IMDSv2:
	- get session token
	- use session token  in IMDSv2 calls using headers

### CLI with MFA
- `aws sts get-session-token --serial-number <ARN_OF_DEVICE> --token-code <CODE_FROM_DEVICE> --duration-seconds 3600`
- add the access key, secret access key, and token into `~/.aws/credentials`

### SDK
- import SDK into your application code to run API calls against AWS
- default region is `us-east-1` if not specified

### API Limits
- API Rate limits
- for intermittent errors: implement exponential backoff (built in w/ the AWS SDK)
	- to self-implement, use retires on 5xx server errors
	- do not implement on 4xx errors
- for consistent errors: request an API throttling limit increase

### Service Quotes
- built in limits to how many resources you can run in your AWS account (i.e. 1152 vCPUs for on-demand instances)

### CLI Credentials Provider Chain
- cli looks for credentials in this order:
	1. CLI options
	2. envvars
	3. CLI credentials file
	4. CLI configuration file
	5. container credentials
	6. instance profile credentials
- SDK:
	1. Java system properties
	2. envvars
	3. default credentials file `~/.aws/credentials`
	4. container credentials
	5. instance profile credentials
- never store credentials in your code
- if working within AWS, use IAM roles
- outside of AWS, use envvars/named profiles

### Signature v4 signing
- when you call the API, you sign the request so that AWS can identify you (using your credentials)
- you should sign an AWS HTTP request with SigV4
	1. include it in the `Authorization` header of the request
	2. inlude it as a query string option (`X-Amz-Signature`)

## Cloudfront (CDN)
- content delivery network that improves read performance by caching content at the edge
- improves user experience
- 216 points of presence globally (edge locations)
- DDoS protection, integration with Shield, AWS WAF

### Origins
- S3 bucket
	- distributing files and cachin them at the edge
	- enhanced security with Cloudfront OAC (origin access control)
		- can be used in combination w/ S3 bucket policy
	- OAC is replacing Origin Access Identity (OAI)
	- can be used as ingress (to upload to S3)
- Custom Origin (HTTP)
	- ALB
		- ALB must be public and allow public IP of edge locations ingress in SG
		- EC2s behind the ALB can be private (ec2 SG must allow LB ingress)
	- EC2 instance
		- must be `Public`
		- allow public IP of edge locations ingress in EC2 SG
	- S3 website (enable bucket as static website)
	- any http backend

### CloudFront vs Cross Region Replication
- CF
	- Global edge netowrk
	- files are cached for a TTL
	- great for static content that must be available everywhere
- CR Repl
	- must be setup in each region you want replication
	- files are updated in near real-time
	- read-only
	- great for dynamic content that needs to be available at low latency in a few regions

### Caching
- the cache lives at each CF edge lcoation
- CF identifies each object in the cache using the Cache Key
- you want to maximize the cache hit ration to minimize requests to the origin
- you can invalidate part of the cache using the `CreateInvalidation` API
- CF cache key = unique identifier for every objec tin the cache
	- by default consists of  hostname + resource portion of the URL
	- if you have an app the servers content based on user, device, lang, location, etc you can add other elements to the Cache Key using CF Cache policies

#### Cache policies
- Cache based on:
	- HTTP Headers: none - whitelist
		- None
			- don't include any headers in the Cache Key (except default)
			- headers are not forwarded (except default)
			- best caching performance
		- Whitelist
			- only specified headers included in the Cache key
			- specified headers are also forwarded to origin
	- Cookies: none - whitelist - include all-except - all
	- Query Strings: none - whitelist - include all-except - all
		- None
			- don't include any query strings in the cache key
			- query strings are not forwarded
		- whitelist
			- only specified query strings included in the cache key
			- only specified query strings are forwarded
		- include all-except
			- include all query-strings in the cache key except the specified list
			- same behavior for forwarding
		- all
			- all query strings included in cache key
			- all are forwarded
			- worst caching performance
- control the TTL (0 seconds to 1 yr) can be set by the origin using `Cache-Control` header, `Expires` header
- create your own policy or use predefined manaed policies
- all http headers, cookies, and query strings that you include in the cache key are automatically included in origin requests

#### Origin Request Policy 
- allows you to specify values that you want to include in origin requests without including them in the cache key
- you can inlude:
	- HTTP Headers: none - whitelist - all viewer headers options
	- Cookies: none- whitelist - all
	- Query Strings: none - whitelist - all
- ability to add CF http headers and custom headers to an origin request that were not included in the viewer request
- create your own or use predefined managed policies

#### Cache Invalidations
- in case you update the backend origin, CF doesn't know and will only get the refreshed content after the TTL has expired
- you can force an entire or partial cache refresh (bypassing the TTL) by performing a CF invlaidation
- you can invalidate all files (`/*`) or a specified path (`/images/*`)

#### Cache Behaviors
- configure different settings for a given URL path pattern
- example: one specific cache behavior to `images/*.jpg` files on your origin web server
- route ot different kind of origins/origin groups based on the content type or path pattern
	- /images/*
	- /api/*
	- /* (default cache behavior)
- when adding addtl cache behaviors the default cache behavior is always processed last

### Geo-restriction
- you can restrict who can access your distribution
	- allowlist: allow your users to access your content only if they're in one of the approved countries
	- blocklist: prevent users from accessing yor content if they're in one of the countries on a list of banned countries
- the country is determine used a 3rd part geo-ip DB
- Use case: copyright laws to control access to content

### CF Signed URL / Signed Cookies
- you want to distribute paid shared content to premium users
- can use CF signed URL / cookie and a attach a policy with
	- URL expiration
	- IP ranges to acces the data from
	- trusted signed (which AWS accounts can create signed URLs)
- how long should the url be valid for
	- shared content (movie, music) make it short (a few mins)
	- private content can be given years of validity
- signed URL  = access to individual files (one signed URL per file)
- signed cookies = access to multiple files (one signed cookie for many files)
- two types of signers
	- Trusted Key Group (recommended)
		- can leverage APIs to create and rotate keys (and IAM for API security)
	- CF Key pair
		- need to manage the keys using the root acct and the AWS console
		- *not recommended* because of required root access
- you should create one or more trusted key groups in your CF distribute (composed of public keys)
	- generate public/private key (2048 bits) to add to your key group
		- private key is used by your applications to sign URLs
		- public key is used by CF to verify the urls

#### CF Signed URL vs S3 Pre-signed URL
- CF signed URL
	- allow access to the path, no matter the origin
	- account wide key-pair (only the root can manage it)
	- can filter by IP, path, date. expiration
	- can leverage caching features
- S3 Pre-signed URL
	- issue a request as the person who pre-signed the url
	- uses the IAM key of the signing IAM principat
	- limited lifetime

### Pricing
- cost of data out per edge location varies
- can reduce the number of edge locations for cost reduction
- 3 price classes:
	- All - most expensive
	- Price class 200 - most regions excluding the most expensive
	- Price class 100 -  only the least expensive regions

### Origin Groups
- to increase HA and use failover, you can user origin groups
- origin group: one primary and one secondary
- if the primary origin fails, the second one is used
- if using S3 origins, this works well with multi-region bucket replication

### Field-level Encryption
- protect user sensitive information through app stack
- adds an additional layer of security along with HTTPS
- sensitive info is encrypted at the edge close to the user
- leverages asymmetric encryption
- usage:
	- specify set of fields in POST requests that you want to be encrypted (up to 10 fields)
	- specify the public key to encrypt them

### Real-time logs
- get real-time requests received by CF sent to kinesis data streams
- monitor, analyze, and take actions based on content delivery performance
- can be delivered to Data Firehose (near real-time)/lambda(real-time)
- allows you to choose:
	- sampling rate - percentage of requests captured with logs
	- specific fields and cache behaviors (path patterns)

## ECS

### IAM roles for ECS 

- EC2 instance profile:
	- used by the ECS agent
	- makes API calls to the ECS service
	- send container logs to CW logs
	- pull docker images from ECR
	- reference sensitive data in Secrets manager or SSM Param store
- ECS Task role:
	- allows each task to have a specific role
	- use diffrent roles for different ECS services
	- task role is defined in the task definition

### LB Integrations
- ALB (port 80/443) supported and works for most use cases
- NLB recommended only for high throughout/ high performance use cases or to pair w/ Private link
- ELB (classic) is support but not recommended (not supported on Fargate / no advanced features)

### ECS Data Volumes (EFS)
- mount EFS file systems onto ECS tasks
- supported for both EC2 and farga
- tasks running in any AZ will share the same data in the EFS file system
- Fargate + EFS = totally serverless
- use cases: persisten multi-AZ shared storage for your containers
- note: S3 cannot be mounted as a file system

### ECS Autoscaling
- automatically increase/decreate the desired number of ECS tasks
- uses AWS App auto autoscaling which can be based on 3 metrics:
	- ECS service avg CPU utilization
	- ECS service avg memory utilization (RAM)
	- ALB request count per target
- Target tracking - scale based on a target value for a cloudwatch metric 
- Step scaling - scale based on a specified CW alarm
- Scheduled scaling - scale based ona s specified date/time (predictable changes)
- FG autoscaling is easier to setup bc it is serverless (ECS service autoscaling at the task level != EC2 auto scaling)

### EC2 Launch Type Autoscaling
- accomodate ECS service scaling by adding ec2 instances
- Autoscaling group scaling
	- Scale your ASG based on CPU utilization
	- Add EC2s over time
- ECS cluster capacity providers
	- automatically provisions and scales the infra for ECS tasks
	- capacity provider paired with an ASG
	- adds EC2s when youre missing capacity

### ECS Rolling Updates
- can control how many tasks can be started and stopped and in which order by specifing Min healhy tasks and Max running tasks

### Architecture Ideas
- ECS tasks invoked by event EventBridge
	- User uploads an object to an S3 which triggers an event
	- event kicks off an ECS tasks which gets the obj from S3, does some processing and saves the result to DynamoDB (task role allows access to other AWS services)
- ECS Tasks invoked by eventbridge schedule
	- every 1 hour event is creates
	- ECS tasks does some batch processing of objs in S#
- ECS tasks polling sqs
	- use autoscaling to see if more capacity is needed to process the messages
- Intercept Stopped tasks using eventbridge
	- Container exited in ECS
	- triggers an event, to send an SNS message alerting Admins of the event

### Task Def
- JSON object
- contains:
	- image name
	- port binding 
	- resources (mem/cpu)
	- envvars
		- can load envvars in bulk via S3
		- can fetch from SSM or Secrets Manager
	- networking information
	- IAM role
	- logging config
- can define up to 10 containers per task def
- in EC2, your can get dynamic host port mapping if you only assign the container port in the task def
	- ALB is smart enough to discover those dynamic ports (does not work with classic LB)
	- in this case, you must have the EC2 SH allowing access on *any* port from the ALB's security group
- in ECS, each task has a unique private IP (only the container port is applicable)
- only 1 IAM role assigned per task def (defined at the task level - not the service level)
- bind mounts allow you to share data bw multiple containers in the same task def
- works for both FG and Ec2
	- in ec2 uses ec2 instance storage
	- in FG they're tied to the containers (20GiB[default] - 200GiB)

### ECS Tasks Placement
- when an EC2 type task is launched, ECS must determine where to place it with resource constraints and available ports (same for scaling down)
- to help guide this process, you can define a task placement strategy and task placement constraints
- task placement strategies are best effort
- process follows these steps:
	1. identify the instances that satisfy the CPU/mem/port reqs in the task def
	2. identify the instances that satisfy the task placement constraints
	3. identify the instnaces that satisfy the task placement strategies
	4. select the instances for task placement
- strategy types - can be mixed together
	- binpack
		- place tasks on the least available amt of cpu/mem
		- minimizes the number of instances in use
	- random (self-explanatory)
	- spread
		- place the task evenly based on the specified value (can be AZ, instance ID, etc)
- placement constraints
	- distinctInstance - place each task on a different container instance
	- memberOf - places tasks on instances that satisfy an expression
		- uses cluster query language (can place tasks only on instances of a specific type)

## ECR
- elastic container registry
- store and manage docker images on AWS either privately or publicly
- fully integrated with ECS (backed by S3)
- access is controlled through IAM
- support image vulnerability scanning, versioning, image tags, image lifecycle
- using the CLI you need to login
	- aws cli v2: `aws ecr get-login-password -- region | docker login -- username AWS -- password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.<RERGION>.amazonaws.com`
- push: `docker push <AWS_ACCOUNT_ID>.dkr.ecr.<RERGION>.amazonaws.com/<IMAGE_NAME>:<TAG>`

## Copilot
- CLI tool used to build, release and operate prod-ready containerized apps
- run your apps on AppRunner, ECS, and Fargate
- helps you focust on building apps rather than setting up infra
- provisions all required infra for containerized apps (ECS, VPC, ELB, ECR)
- automated dpeloyments using one command with CodePipeline
- deploy to multiple environment
- troubleshooting with logs, and health status builtin

## EKS
- elastic kubernetes cluster
- way to launch managed kubernetes clusters on AWS
- kubernetes is open source and supported across multiple cloud providers
- EKS supports EC2 if you want to deploy worker nodes or Fargate to deploy serverless containers

### Node Types
- managed node groups
	- creates and manages nodes for you
	- nodes are part of an ASG managed by EKS
	- supports on demand or spot instances
- self-managed nodes
	- nodes created by you and registered to the EKS cluster (also managed by an ASG)
	- you can use the prebuilt AMI - Amazon EKS optimized AMI
	- supports on-demand or spot instances

### Data Volumes
- need to specify a StorageClass manifest on your EKS cluster
- leverages a CSI (container storage interface) compliant driver
- support for:
	- EBS
	- EFS (fargate)
	- FSx for Lustre
	- FSx for NetApp ONTAP

## Elastic Beanstalk
- developer centric view of deploying an app on AWS
- uses all the infra components but is a managed service
	- automatically handles capacity provisioning, load balancing, scaling, app health monitoring, instance config
	- just the app code is the responsibility of the developer
- still gives you full control over the ocnfig
- free service (you pay for the underlying deployed services)
- there's an EB CLI that helps working with beanstalk esp for deployment pipelines



### Components
- applications: collection of EB components (envs, versions, configs)
- app version: an iteration of your app code
- env:	
	- collection of AWS resources running an app version (only one app version at a time)
	- tiers: web server tier and worker env tier
	- can create multiple envs

### Supported Platforms
- Go, Java SE, Java with Tomcat, .NET core on linux, .NET on windows, Node.js, PHP, python, Ruby, Packer builder, single container docker, multi-container docker, preconfig docker
- if not supported, you can also write a custom platform

### Deployment Modes
- single instance (dev)
- HA with LB (prod)
	- supports RDS

### Deployment modes for updates
- All at once (fastest - has downtime - no addtl cost)
- Rolling - update a few instances at a time, then move onto the next bucket once the first bucket is healthy (no addtl cost)
	- loss of capacity at times
- Rolling with additional batches -  like rolling, but spins up new instances to move the batch (so that the old app is still available)
	- small addtl cost
	- additional batch is removed at the end
	- always running at capacity (sometimes over)
- Immutable - spins up new instances in a new ASG, deploys the version to these instances, then swaps all the instances when everythin is heathy
	- high cost (double capacity)
	- longest deployment
	- new code is deployed to new instances on a temp ASG
	- then the new instances are moved into the existing ASG
	- temp ASG is terminated
- Blue Green - create a new env and switch ver when ready
	- not a "direct feature" of EB
	- zero downtime
	- create a new "stage" env and deploy v2 there
	- route 53 can be setup with weighted policies to redirect some of the traffic to stage
	- using beanstalk, swap URLs when you are happy with the new env
- Traffic splitting - canary testing to send a small % of traffic to the new deployment
	- new app version is deployed to a temp ASG with the same capacity (double capacity)
	- traffic is split for a configurable amt of time
	- deployment health is monitored
	- if there's a deployment failure, this triggers an automated rollback
	- no app downtime
	- new instances are migrated from the temporary to the original ASG
	- old appp version is then terminated


### Deployment Process
- describe dependencies (requirements.txt for python)
- package code as zip
- upload the zip file in Console (or create new app version in CLI)
- EB deploys the zip on each EC2, resolve deps, and start the app

### Beanstalk lifecycle policy
- max 1000 app versions can be stored
- if you don't remove old verions, you cannot deploy anymore
- to phase out old apps, use a lifecycle policy (policies will not delete app versions in use or being created)
	- based on time (old versions are removed)
	- based on space (when you have too many versions)
- option not to delete the source bundle in S3 to prevent data loss

### Beanstalk extensions
- requirements:
	- in the `.ebextensions/` diretory of source code
	- yaml/json format
	- `.config` extensions
	- you are able to modify some default setting using:`option_settings`
	- ability to add resources such as RDS, elasticache, dynamoDB, etc
- resources managed by `.ebextensions` get deleted if the environment goes away

### EB under the hood
- uses cloudformation as the basis of the operations to provision other AWS services
- with ebextension you can configure many other services

### EB cloning
- allows you to clone an env with the exact same configuration
- useful for prototyping and testing

### Migration
- once you create the env, you cannot change the load balancer type
- to migrate:
	- create a new env with the same config (except LB - cannot clone)
	- deploy app in the new env
	- perform a CNAME swap or Route 53 update to route the traffic to the updated env

### RDS
- can provision with EB, but then it is tied to the env
- best practice is to seperately create an RDS DB and give the EB app the connection string
- to decouple an existing DB:
	- snapshot the RDB DB
	- go to RDS console and protect the RDS 
	- create a new EB, without RDS, point your APP to existing RDS
	- perform a CNAME sway or Route 53 update after you confirm the new app is working
	- terminate the old env (RDS wont be deleted)
	- delete the cloudformation stack (as the delete will fail because the RDS db is still around)

## Cloudformation
- declarative way of outlining your AWS infra
- IAC
- free to use:
	- each resource within the stack is tagged with an id so you can easily see how much a stack costs
	- can estimate the costs of your resources using the CF template
- in dev you can destroy and recreate resources overnight
- automated geneation of diagram for your templates
- separation of concerns: create many stacks for many apps
- to update a template we cant edit a previous one (have to reupload a new version of the template to AWS)
- can manage templates in the UI (CF designer) OR managed the templates in yaml files and use the CLI to deploy

### Resources
- resources are the core of your CF template
- they represent the different AWS componenet that will be created and configured
- resources are declared and can reference each other
- AWS figures out creation/updates/deletions of resources for us
- over 224 types of resources
- cannot have a "dynamic" amt of resources
- almost every AWS service is supportes as resources,  for those that aren't you can work around using AWS Lambda Custom Resources
### Parameters
- way to provide inputs to your AWS CF template
- some inputs cannot be determined ahead of time
- Is this likely to change in the future?  If yes, parameter is a good idea
- short hand for ref in yaml is `!Ref` (`Fn::Ref`)
	- can be used to reference parameters and resources
- pseudo parameters can be used at any time and are enabled by default (i.e. `AWS::AccountId`)

### Mappings
- mappings are fixes variables within the CF template
- handy to differentiate bw different envs, regions, AMI types, etc
- all values are hardcoded within the template
- great for when you know in advance all of the values that are available
- accessing mapping values `Fn::FindInMap`
	- `!FindInMap [ MapName, TopLevelKey, SecondLevelKey ]`

### Outputs
- declares *optional* outputs values that we can import into other stacks
- you can also view the outputs int he AWS console or in the CLI
- enables cross-stack collaboration 
- you cannot delete a CF stack if its outputs are being used in another stack
- cross-stack reference `!ImportValue` (`Fn::ImportValue`)

### Conditions
- conditions are used to control the creation of resources or outputs based on a condition
- can use intrinsic logical functions
- added at the same level as `Type`

### Intrinsic Functions
- `Fn::Ref`
- `Fn::FindInMap`
- `Fn::ImportValue`
- `Fn:GetAtt` (can be attached to any resources you create)
- `Fn::Join` (join values with a delimiter)
	- `!Join [ delimiter, [comma-delimited list of values]]`
- `Fn::Sub` shorthand for substitute to sub variables from text
- `Fn::And, Fn::Equals, Fn::If, Fn::Not, Fn::Or`

### CF Rollbacks
- creation fails
	- default: everything rolls back
	- option to disable rollback in order to troubleshoot
- stack update fails
	- the stack automatically rolls back to the previous known working state
	- ability to see in the log what happened (and error msgs)

### CF Stack notifications
- can send stack events to SNS topic (email, lambda)
- enable SNS integration using stack options

### Changesets
- won't say if the update is successful, but allows you to know what changes will happen with greater confidence

### Nested Stacks
- stacks are part of other stacks
- allow you to isolate repeated patterns/common components in separate stacks and call them from other stacks
- ex LB config that is re-used, SG that is re-used
- nested stacks are considered best practice
- to update a nested stack, always update the parent (root)

### Cross stack 
	- helpful when stacks have different lifecycles (use outputs export and import value)
	- helpful when you need to pass export values to many stacks

### Stack Sets
- create, update, or delete stacks across multiple accounts and regions with a single operation
- admin acct to create StackSets
- trusted accts to create, update, delete stack instances from stackset
- when you update a stackset, all associated stack instances are updates throughout all accounts and regions

### Drift
- manual config changes are not prevented by CF
- we can use drift to detect when changes have been made to CF managed resources

### Stack Policies
- dring an update, all update actions are allowed on all resources by default
- a stack policy is json doc that defines the udpates actions that are allowed on specific resources during stack updates
- help you protect resources from unintentional updates
- when you set a stack policy, all resources in the stack are protected by default
- specify an explicit AALOW for the resources you want to be allowed to be updated
## Yaml
- key value pairs
- nested objects
- supports arrays
- supports multiline
