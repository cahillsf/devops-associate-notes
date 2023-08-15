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
- EBS volumes do not *need* to be attached to an instance
- Delete on Termination attribute 
	- determines whether the volume is deleted on instance termination
	- by default Root volume is deleted on termination
	- any other attached instances are *not* terminated on deletion by default
- snapshots:
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
- SSD (can be used for boot volumes):
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
	- only SSD can be used for boot disks

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
- uses a POSIX file system that scales automatically (no capacity planning)
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
	- offers snapshot feature to migrate across AZ
	- EBS backups use IO and you shouldn't run them while handling high traffic
	- root EBS volumes of instances get terminated by default along with EC2 instance termination (can be disabled)
- EFS
	- can be mounted to 100s of instances across AZs
	- only for linux (POSIX) instances
	- higher pricepoint that EBS
	- can leverage EFS-IA for cost savings

## ELB and ASG

- scalability: application/system can handle greater loads by adapting
- two kinds of *scalability*:
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
- *high availability*:
	- goes hand in hand with horizontal scaling
	- means running your app/system in at least 2 AZs
	- goal is to survive a DC outage
	- HA can be passive (RDS multi-AZ) or active (horizontal scaling)
	- multi-AZ ASGs and Load balancing enable this

### ELB (managed service)
- spread load across multiple downstream services
- expose a single point of access (DNS) to your app
- allows you to seamlessly handle failures of downstream instances
- accomplished health checks to your instances
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
	- routing based on QueryString or Headers
- great fit for microservices and container-based apps
- has a port mapping feature to redirect to a dynamic port in ECS

- what are target groups? can be any of the following:
	- EC2 instances (can be managed by an ASG) - HTTP
	- ECS tasks (managed by ECS) - HTTP
	- Lambda functions - https request is translated to a JSON event
	- IP addresses (must be private IPs)
	- ALB can route to multiple target groups
	- health checks are at the target group level
- ALBs have a *fixed hostname*
- app servers don't see the IP of the client directly 
	- *true* ip of the client is inserted into header `X-Forwarded-For`
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
- examples: firewalls, intrusion detection, prevention systems, deep packet inspection system, payload manipulation
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
- the cookie used for stickiness has an expiration timestamp that you can control
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
	- cross AZ is *disabled* by default
	- you pay for inter AZ data
- CLB
	- cross AZ is *disabled* by default
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
- SNI solves the problem of loading multiple SSL certs onto 1 web server to serve multiple web sites
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
	- when an alarm is trigerred, then add 2 units
- scheduled actions
	- anticipate a scaling based on a known usage pattern
- predictive scaling
	- continuously forecast load and schedule scaling ahead

- good metrics for scaling:
	- CPUUtilization (avg CPU)
	- RequestCountPerTarget (make sure the number of reqs/ instance is stable)
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
- caveat: you cannot SSH into your instances as they are offered as a *managed service*

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
- replication is ASYNC, so reads are *eventually* consistent
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
- *not* used for scaling
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
- failover is instantaneous (it's HA native)
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
	- if the master is not encryped, the read replicas cannot be encrypted
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
- lazy loading/cache aside is easy to implement and works for many situations as a foundation, esp on the read side
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

##### Pros:
- data in cache is never stale, reads are quick
- write penalty vs read penalty (each write requires 2 calls)
	- could be better from a user perspective (writes are understood to take more time vs a fetch)

##### Cons:
- missing data until it is added/update in the DB
	- mitigation is to implement lazy loading as well
	- but risk of cache churn - a lot of the data written to the cache will never be read

#### Cache Evictions and TTL (time-to-live)
- cache eviction can occur in three ways:
	- you delete the item explicitly in the cache
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
	- you cannot create a CNAME record for the top node of a DNS namespace (Zone Apex)
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
		- if > 18% if the health checkers report the endpoint is healthy, route 53 considers it healthy (otherwise unhealthy)
		- ability to choose which locations you wante route 53 to use
		- health checks pass when the endpoint responds with 2xx and 3xx status codes
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
- would need to specify the AWS nameservers within the domain registrar and create a hosted zone in route 53 

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
	- can have only ALLOW rules
	- rules include IP addresses and other security groups
	- *stateful*: return traffic is automatically allowed
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
	- instead, you must use an external DB as a search index such as DynamoDB
- version ID

### Bucket Policy
- user -based:
	- IAM policies
- Resource-based
	- Bucket Policies - bucket wide rules from the S3 console (allows cross-account)
	- Object ACL (access control list) - finer grain (can be disabled)
	- Bucket ACL - less common (can be disabled)
- *Note:* an IAM principal can access an S3 object if:
	- the user IAM permissions ALLOW it or the resource policy ALLOWS it
	- AND there's no explicit DENY
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
- after you enable replication, only new objects are replicated
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
	- Glacier Instant Retrieval
		- millisecond retrieval 
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
	- Frequent (default), Infrequent (30 days no access), Archive Instant Access (90 days no access), Archive Acess (optional - 90 to 700+ days), Deep Archive Access (optional - 180 to 700+ days)

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
- you can allow for a specific origin or for `*` (all origins)

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
	- to self-implement, use retries on 5xx server errors
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
	2. inlcude it as a query string option (`X-Amz-Signature`)

## Cloudfront (CDN)
- content delivery network that improves read performance by caching content at the edge
- improves user experience
- 216 points of presence globally (edge locations)
- DDoS protection, integration with Shield, AWS WAF

### Origins
- S3 bucket
	- distributing files and caching them at the edge
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
- the cache lives at each CF edge location
- CF identifies each object in the cache using the Cache Key
- you want to maximize the cache hit ratio to minimize requests to the origin
- you can invalidate part of the cache using the `CreateInvalidation` API
- CF cache key = unique identifier for every object in the cache
	- by default consists of  hostname + resource portion of the URL
	- if you have an app that serves content based on user, device, lang, location, etc you can add other elements to the Cache Key using CF Cache policies

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
- create your own policy or use predefined managed policies
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
- you can force an entire or partial cache refresh (bypassing the TTL) by performing a CF invalidation
- you can invalidate all files (`/*`) or a specified path (`/images/*`)

#### Cache Behaviors
- configure different settings for a given URL path pattern
- example: one specific cache behavior to `images/*.jpg` files on your origin web server
- route to different kind of origins/origin groups based on the content type or path pattern
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
- can use CF signed URL / cookie and attach a policy with
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
	- uses the IAM key of the signing IAM principal
	- limited lifetime

### Pricing
- cost of data out per edge location varies
- can reduce the number of edge locations for cost reduction
- 3 price classes:
	- All - most expensive
	- Price class 200 - most regions excluding the most expensive
	- Price class 100 -  only the least expensive regions

### Origin Groups
- to increase HA and use failover, you can use origin groups
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
	- use different roles for different ECS services
	- task role is defined in the task definition

### LB Integrations
- ALB (port 80/443) supported and works for most use cases
- NLB recommended only for high throughout/ high performance use cases or to pair w/ Private link
- ELB (classic) is supported but not recommended (not supported on Fargate / no advanced features)

### ECS Data Volumes (EFS)
- mount EFS file systems onto ECS tasks
- supported for both EC2 and fargate
- tasks running in any AZ will share the same data in the EFS file system
- Fargate + EFS = totally serverless
- use cases: persistent multi-AZ shared storage for your containers
- note: S3 cannot be mounted as a file system

### ECS Autoscaling
- automatically increase/decrease the desired number of ECS tasks
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
	- every 1 hour event is created
	- ECS tasks does some batch processing of objs in S3
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
	3. identify the instances that satisfy the task placement strategies
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
- helps you focus on building apps rather than setting up infra
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

## Elastic Beanstalk (todo)
- developer centric view of deploying an app on AWS
- uses all the infra components but is a managed service
	- automatically handles capacity provisioning, load balancing, scaling, app health monitoring, instance config
	- just the app code is the responsibility of the developer
- still gives you full control over the config
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
- Immutable - spins up new instances in a new ASG, deploys the version to these instances, then swaps all the instances when everything is heathy
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
	- old app version is then terminated


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
	- perform a CNAME swapß or Route 53 update after you confirm the new app is working
	- terminate the old env (RDS wont be deleted)
	- delete the cloudformation stack (as the delete will fail because the RDS db is still around)

## Cloudformation
- declarative way of outlining your AWS infra
- IAC
- free to use:
	- each resource within the stack is tagged with an id so you can easily see how much a stack costs
	- can estimate the costs of your resources using the CF template
- in dev you can destroy and recreate resources overnight
- automated generation of diagram for your templates
- separation of concerns: create many stacks for many apps
- to update a template we cant edit a previous one (have to reupload a new version of the template to AWS)
- can manage templates in the UI (CF designer) OR managed the templates in yaml files and use the CLI to deploy

### Resources
- resources are the core of your CF template
- they represent the different AWS components that will be created and configured
- resources are declared and can reference each other
- AWS figures out creation/updates/deletions of resources for us
- over 224 types of resources
- cannot have a "dynamic" amt of resources
- almost every AWS service is supported as a resource,  for those that aren't you can work around using AWS Lambda Custom Resources
### Parameters
- way to provide inputs to your AWS CF template
- some inputs cannot be determined ahead of time
- Is this likely to change in the future?  If yes, parameter is a good idea
- short hand for ref in yaml is `!Ref` (`Fn::Ref`)
	- can be used to reference parameters and resources
- pseudo parameters can be used at any time and are enabled by default (i.e. `AWS::AccountId`)

### Mappings
- mappings are fixed variables within the CF template
- handy to differentiate bw different envs, regions, AMI types, etc
- all values are hardcoded within the template
- great for when you know in advance all of the values that are available
- accessing mapping values `Fn::FindInMap`
	- `!FindInMap [ MapName, TopLevelKey, SecondLevelKey ]`

### Outputs
- declares *optional* outputs values that we can import into other stacks
- you can also view the outputs in the AWS console or in the CLI
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

## Monitoring and Audit (CW, X-Ray, CloudTrail)
- CloudWatch:
	- metrics
	- logs
	- events
	- alarms
- X-Ray:
	- troubleshooting app performance and errors
	- distributed tracing of microservices
- CloudTrail:
	- internal monitoring of API calls being made to AWS
	- audit changes to AWS resources by your users

### CW Metrics
- AWS CW provides metrics for every service in AWS
- a metric is a variable to monitor
- each metric belongs to a namespace
- a dimention is an attribute of a metric
- up to 30 dimensions per metric
- metrics have timestamps
- can create cloudwatch dashboards of metrics

#### EC2 Metrics
- by default metrics are submitted every 5 mins
- detailed monitoring (extra cost) metrics are submitted every minute
- detailed monitoring is helpful if you want to scale your ASG fatsre
- EC2 memory usage is by default not pushed (must be submitted as a custom metric

#### CW Custom metrics
- use API call `PutMetricData`
- `StorageResolution` API param can be set to 
	- Standard (1 minute)
	- High Res - 1/5/10/30 seconds = higher cost
- the API accepts metric data points two weeks in the past and two hours in the future

### CW Logs
- log groups: arbitrary name, usually representing an app
- log stream: instances within app/ log files/ containers
- can define log expiration policies (never expire, 1 day to 10 years)
- CW logs can send logs to
	- S3 Export
		- Log data can take up to 12 hours to become available for export
		- API is `CreateExportTask`
		- not near real-time or real-time
	- Kinesis Data Streams
	- Kinesis Data Firehose
	- AWS Lambda
	- OpenSearch
- logs are encrypted by default
- can setup KMS-based encryption with your own keys

#### CW Logs Sources
- SDK, CW Logs Agent, CW Unified Agent
- Elasic Beanstalk: collection of logs from an app
- ECS: container logs
- Lambda: serverless logs
- VPC Flow Logs
- API Gateway
- CloudTrail based on filter
- Route53 DNS queries

#### CW Logs Insights
- for searching an analyzing log data within CW logs
- provides a purpose-built query language
	- automatically discovers fields from AWS services and JSON log events
	- fetch desired event fields, filter based on conditions, calculate stats, sort events, limit number of events
	- can save queries and add them to CW dashboards
	- Can query multiple log groups in different AWS account
	- it's a query engine (not a real-time engine)

#### CW Logs Subscriptions
- allow you to get real-time events from CW logs for processing and analysis
- send to Kinesis Data Streams, Kinesis Data Firehose, or Lambda
- Subscription Filter - decide which logs are filtered to your destination

#### Logs Aggregation Multi-account and Multi-region
- allow you to use subscription filters to ship logs from different regions/accounts to Data Streams or Firehose
- Cross-Account Subscription filter
	- in the sender account create a subscription filter
	- in the recipient account:
		- create a Destination Access Policy that allows the sender account `PutSubscriptionFilter` action on the subscription destination 
		- create a cross account IAM Role with a `PutRecord` permission in the Kinesis data stream, and ensure the sender account can assume this role

#### CW Logs for EC2
- by default, no logs from your EC2 machine will go to CW
	- can also be setup on prem
- need to run a CW agent on EC2 to push the logs
- make sure the IAM permissions on the EC2 instance


##### CW Logs Agent and Unified Agent
- CW logs agent
	- "old" agent version
	- only send logs
- CW Unified Agent
	- collect addtl system-level metrics
	- collect logs to send to CW
	- can centralize the config using SSM parameter store

#### CW Logs Metric Filter
- allows you to create metrics from logs based on filters and 3 (optional) dimenstions
- do not retroactively publish metrics (only publish metric data for events *after* filter creation)
- can create an alarm based on the CW Metric

### CW Alarms
- used to trigger notifications on any metrics
- states:
	- OK
	- INSUFFICIENT_DATA
	- ALARM
- Period:
	- length of time in seconds to evaluate the metric
- to test alarms and condition, you can set the alarm state manually using `aws cloudwatch set-alarm-state`
- high res alarms can use 10 or 30 second resolution, regular alarms can be set with any multiple of 60 seconds

#### Alarm Targets
- EC2 (stop, start, terminate an instance)
- Trigger an AutoScaling action
- send notification to SNS (from which you can do pretty much anything)
- SystemManager action

#### Composite Alarms
- monitor the states of multiple other alarms
- AND or OR conditions
- helpful for reducing "alarm noise" by creating complex composite alarms

#### EC2 instance recovery
- status check
	- instance status = check the EC2 VM
	- system status =  check the underlying hardware
- `StatusCheckFailed_System` can trigger an EC2 instance recovery where you move your instance to another host with the same: public/private IP, elastic IP, metadata, placement group
- can trigger an SNS notification as well

### CW Synthetics Canary
- script to monitor APIs, URLs, website
- reproduce what your customers do programmaticlly to find issues proactively
- check the availability and latency of your endpoints
- store load time data and UI screenshots
- integration w/ CW alarms
- scripts written in Node.js or Python
- programmatic access to a headless google chrome browser
- can be scheduled as one-offs or recurring

#### Blueprints
- Heartbeat monitor -  load URL, store screenshot and an HTTP archive file
- API canary - test basic read/write functions of REST APIs
- Broken Link Checker - checks all links inside the URL you are testing
- visual monitoring - compare a screenshot taken during a canary run w/ a baseline screenshot
- Canary Recorder - used with CW synthetics recorder to records your actions on a site and automatically generate a script
- GUI workflow builder - verifies that actions can be taken on your webpage

### EventBridge (formerly CW events)
- schedule: Cron jobs (schedules scripts)
- event pattern: event rules to react to a service doing something
- trigger lambda functions, send SQS/SNS messages
- can filter events
- example destinations
	- Lambda, AWS Batch, ECS Task
	- SQS, SNS, Kinesis SData Streams
	- Step Fnctions, CodePipeline, CodeBuil
	- SSM, EC2 actions
- Eventbridge is the `Default Event Bus` receiving events from AWS services
- can also use `Partner Event Buses` and `Custom Event Buses`
- event buses can be access by other AWS accounts using resource-based policies
- can archive events sent to an event bus (indefinitely or set period) and replay these archived events

#### Schema Registry
- eventbridge can analyze the events in your bus and infer the schema
- the schema registry allows you to generate code for your application that will know in advance how data is structured in the event bus
- schema can be versioned

#### Resource-based policy
- manage permissions for a specific event bus
- example: allow/deny events from another AWS account/region
- allows you to aggregate all events from your AWS org into a single account or region

#### Multi-account aggregation
- create an event rule in the sender account that forwards the events to an event bus in another account
- create a resource policy in the receiver account that allows the other accounts to send events

### X-Ray
- compatiblity:
	- Lambda
	- Beanstalk
	- ECS
	- ELB
	- API gateway
	- EC2s or any app server
- tracing is made of "segments"
	- can define "subsegments" for more details
- annotations can be added to traces to provide extra info
	- key/value pairs used to index traces and use with filters
- metadata on traces: key/value pairs, not indexed or used for searching
- x-ray daemon has a config to send traces cross accounts with the correct IAM permissions that the agent assumes
	- allows you to centralize tracing
- Security:
	- IAM for auth
	- KMS for encrytion at rest

#### Sampling
- sampling rules allow you to control the amount of data you record
- by default, the SDK records the first request each second, and five percent sampling of any additional requests
	- the one traced req/second is the *reservoir* which ensures that at least one trace/second is recorded as long as the service is serving requests
- custom sampling rules allow you to create your own rules with the *reservoir* and *rate*
	- can specify service, resource type, action, etc

#### How to enable?
1. Your code (Java, python, Go, Node.js, .NET)
	- very little code modification required
	- the app SDK then captures
		- calls to AWS services
		- HTTP/HTTPS requests
		- DB calls
		- queue calls
2. Install the x-ray daemon or enable x-ray AWS integration
	- x-ray daemon works as a low level UDP packet interceptor
	- Lambda and other AWS services already run the daemon for you
	- each app must have the IAM rights to write data to x-ray
	- daemon sends the batches to AWS every 1 second

#### Troubleshooting
- EC2
	- ensure the EC2 IAM role has the proper permisions
	- ensure the daemon is running
- Lambda
	- ensure it has IAM execution role with proper policy (`AWSX-RayWriteOnlyAccess`)
	- ensure x-ray is imported in the code
	- enable lambda x-ray active tracing

#### X-Ray Write APIs
- used by the X-Ray Daemon
	- `PutTraceSegments`
	- `PutTelemetryRecords`
		- SegmentsReceivedCount, SegmentsRejectedCount, BackendConnErrs
	- `GetSamplingRules`
	- `GetSamplingTargets` and `GetSamplingStatisticSummaries`
	- x-ray daemon needs to have an IAM policy authorizing the correct API calls to function correctly

#### X-Ray Read APIs
- `GetServiceGraph`: main graph
- `BatchGetTraces`: retrieves a list of traces specified by ID
- `GetTraceSummaries`: retrieves IDs and annotations for traces available for a specified time frame using an optional filter
- `GetTraceGraph`: retrieves a service graph for one or more specific trace IDs

#### X-Ray <-> Beanstalk
- beanstalk includes the x-ray daemon
- can run the daemon by setting an option in the EB console or with a config file
- give your instance profile the correct IAM permissions so the daemon can function properly
- instrument the app code
- Note: x-ray daemon is not provided for multicontainer docker

#### ECS <-> X-Ray

##### ECS on EC2
- run the daemon as a container
- x-ray container as "daemon"
	- one side car per ec2 instance

##### ECS on Fargate
- x-ray container as "sidecar"
	- one side car per app container

#### AWS Distro fo Otel
- secure, prod-ready AWS-supported distribution of the open-source Otel project

### CloudTrail
- provides governance, compliance and audit for AWS
- enabled by default
- get a history of events / API calls made within your AWS account by
	- console
	- SDK
	- CLI
	- AWS services
- can put logs from cloudtrail into CW logs or S3
- a trail can be applied to all regions (default) or a single region

#### Events
- Management events
	- operations that are performed on resource in your AWS account
	- trails are configured to log mgmt events
	- can separate read events from write events
- Data events
	- by default data events are not logged
		- i.e. S3 object-level activity, lambda `Invoke` API
- CloudTrail Insights
	- must be enabled (paid service)
	- used to detect unusual activity in your account
		- inaccurate resource provisioning
		- hitting service limits
		- bursts of AWS IAM actions
		- gaps in periodic maintenance activity
	- analyzes normal mgmt events to create a baseline, then continuously analyzes *write* events to detect unusual patterns
		- anomalies appear in the CloudTrail console
		- event is sent to Amazon s3
		- eventbridge event is generated

#### Event Retention
- stored for 90 days
- for longer storage send to s3 and use athena to analyze

#### EventBridge Integration
- intercept API calls to send to eventbridge to create alerts

## Integration and Messaging: SQS, SNS & Kinesis
- two patterns of app communication: sync and async
- syncronous communication can be problematic if there are sudden spikes in traffic
- better to decouple your applications:
	- SQS: queue model
	- SNS: pub/sub model
	- Kinesis: real-time streaming
- these services can scale independently of the application

### SQS
- producers send messages to the SQS queue
	- use the SDK (`SendMessage` API)
	- message is persisted in SQS until a consumer deletes it
- consumers poll messages from the SQS queue
	- once the message is polled, it is processed, then deleted from the queue using the `DeleteMessage` API
	- consumers can receive and process messages in parallel
	- consumers can be scaled horizontally to improve throughput (i.e. based your ASG logic on the `ApproximateNumberOfMessages` metric)
- can have as many consumers as you like

#### Security
- encryption:
	- in-flight encryption using HTTPS API
	- at-rest encryption using KMS keys
	- client-side encryption if the client wants to perform encryption/decryption itself
- access controls: IAM policies to regulate access to the SQS API
- SQS Access policies
	- useful for cross-account access to SQS queues 
	- useful for allowing other services (SNS, S3) to write to an SQS queue
	- need `ReceiveMessage` to poll and `SendMessage` to put


#### Standard Queue
- oldest offering (10 yrs)
- fully managed service used to decouple applications
- attributes:
	- unlimited throughout, unlimited number of messages in a queue
	- default retention of messages: 4 - 14 days
	- low latency (< 10 ms on publish and receive)
	- limitation of 256KB per message
- can have duplicate messages (at least once delivery)
- can have out of order messages (best effort ordering)

#### Message Visibility Timeout
- after a message is polled by a consumer, it becomes invisible to other consumer (by default set to 30 seconds)
- the message then has 30 seconds to be processed
- after the visibility time is over, the message becomes "visible" in SQS		
- if a message is not processed within the visibility timeout, it will be processed twice
- a consumer could call the `ChangeMessageVisibility` API to get more time
- if timeout is too high and the consumer crashed, re-processing will take a lot of time
- if timeout is too low, the message could be processed many times

#### Dead Letter Queue (DLQ)
- if a consumer fails to process a message within the timeout, the message goes back to the queue
- we can set a threshold of how many times a message can go back to the queuea
- after the `MaximumReceives` threshold is exceeded, the message goes into a DLQ
	- useful for debugging
- DLQ of a FIFO queue must be FIFO (DLQ of a standard queue must be std as well)
- good to set a max retention policy in the DLQ of 14 days to process the messages before they expire

##### Redrive to Source
- feature to help consume messages in the DLQ to understand what is wrong with them
- when our code is fixed, we can redrive the message from the DLQ back into the source queue (or other queue) in batches without writing custom code

#### Delay Queue
- delay a message up to 15 minutes (default is 0)
- can set a default at queue level
- can override the default on send using the `DelaySeconds` parameter

#### Long Polling
- when a consumer requests messages from the queue, it can optionally "wait" for messages to arrive if there are none in the queue
- decreases the number of API calls made to SQS while increasing efficiency and latency
- wait time can be bw 1 sec to 20 secs
- is preferred to short polling
- can be enabled at the queue level or at the API level using `ReceiveMessageWaitTimeSeconds`


#### SQS Extended Client (Java Library)
- uses an S3 bucket to hold the large message
- also sends a small metadata message to the SQS queue which contains a pointer to the file in S3 that the consumer can process

#### SQS API
- `CreateQueue` Set `MessageRetentionPeriod`, `DeleteQueue`
- `PurgeQueue`
- `SendMessage`(`DelaySeconds`), `DeleteMessage`
- `ReceiveMessage` (`MaxNumberOfMessages`):default 1, max 10
- `ReceiveMessageWaitTimeSeconds` = long polling
- `ChangeMessageVisibility`: change the message timeout
- batch APIs for `SendMessage`, `DeleteMessage`, `ChangeMessageVisibility` helps decrease your cost


#### FIFO Queue
- limited throughout (300 msgs/sec without batching 3000 msg/s with batching)
- exactly-once send capability (by removing duplicates)
- messages are processed in order by the consumer

##### Deduplication
- deduplication interval is 5 minutes
- two dedup methods:
	- content-based deduplication: calcs a SHA-256 hash of the message body
	- explicity provide a message deduplication ID

##### Message Grouping
- if you specify the same value of `MessageGroupID` in a SQS FIFO queue, you can only have one consumer and all the messages are in order
- to get ordering for subset of messages, specify different values for `MessageGroupId`:
	- messages that share a common Message Group ID will be in order within the group
	- each Group ID can have a different consumer (parallel processing)
	- ordering across groups is not guaranteed

### SNS
- "event producer" only sends messages to one SNS topic
- as many "event receivers" (subscriptions) as 12,500,000 can listen to the SNS topic notifications
- each subscriber to the topic will get all the messages (note: now you can filter messages)
- 100,000 topics limit
- many AWS services directly integrate with SNS
- supported subscription protocols:
	- KDF
	- Lambda
	- Email
	- Email (json)
	- HTTP
	- HTTPS
	- SMS

#### How to publish

- Topic Publish (using SDK)
	- create a topic
	- create a subscription (or many)
	- publish to the topic

- Direct Publish (mobile apps SDK)
	- create a platform application
	- create a platform endpoint
	- publish to the platform endpoint
	- works with Google GCM, Apple APNS, Amazon ADM

#### Security (same as SQS)
- encryption
	- in flight encryption using HTTPS
	- at rest encryption using KMS keys
	- client-side encryption if the client wants to perform encryption/decryption itself

- access controls: IAM policies to regulate access to SNS API

- SNS access policies
	- useful for cross-account access to SNS topics
	- useful for allowing other services to write to an SNS topic

### SNS - Fifo topic
- strict ordering by message group ID
- deduplication using a dedup ID or content based dedup
- can only have SQS FIFO queues as subscribers
- limited throughput (same as SQS FIFO)
- in case you need fan out + ordering + dedup

### Message filtering
- JSON policy used to filter messages sent to SNS topic's subscriptions
- if a sub doesnt have a filter policy, it receives every message

### SNS/SQS: Fan Out
- push once in SNS, receive in all SQS queues that are subscribers
- fully decoupled, no data loss
- SQS allows for: data persistence, delayed processing and retries of work
- ability to add more SQS subscribers over time
- make sure SQS queue access policy allows for SNS to write
- cross-region delivery: works with SQS queues in other regions

#### Use case: S3 events to multiple queues
- for the same combo of event type and prefix, you can only have one S3 event rule
- if you want to send the same S3 event to many SQS queues, use fan-out

#### Use case: SNS to S3 through firehose
- SNS can send to Kinesis
- Service -> SNS -> Kinesis Data Firehose (from here can go to S3 or any other KDF destination)


### Kinesis
- makes it east to collect, process and analyze streaming data in real-time, such as: app logs, metircs, website clickstreams, IoT telemetry data, etc

#### Kinesis Data Streams
- streams are provisioned with shards
- meant for real-time big data, analytics, and ETL
- retention can be 1 day - 365 days
- ability to reprocess (replay) data
- once data is inserted in Kinesis, it cannot be deleted (immutability)
- data that shares the same partition goes to the same shard (ordering)

##### Capacity modes
- provisioned mode
	- choose the number of shards provisioned, scale manually or using API
	- each shard gets 1MB/s in (or 1000 records per second)
	- each shard gets 2MB/s out (classic or enhanced)
	- pay per shard provisioned per hour
- on-demand mode
	- no need to provision or manage capacity
	- default provisioned capacity (4MB/s in or 4000 records/second)
	- scaled automatically based on observed throughput peak during last 30 days
	- pay per stream per hour & data in/out per GB

##### Security
- control access/auth using IAM
- regional resource
- encryption in flight using HTTPS endpoints
- encryption at rest using KMS
- you can implement encryption/decryption of data on client side
- VPC endpoint available for Kinesis to access within VPC
- monitor API calls using cloudtrail
##### producers
- send data into the data streams via records
- all use the SDK, some common producers are:	
	- apps
	- client
	- AWS SDK, KPL (Kinesis producer library)
	- Kinesis agent -  monitor log files
- records: sequence no, partition key and data blob (up to 1MB)
- throughput is 1MB/sec or 1000msg/sec per shard
- `PutRecord` api
- use batching with `PutRecord` to reduce cost and increase throughput (KPL does this already)
- hash function used to map a partition key to a specific shard
- use highly distributed partition key to avoid "hot partition"
- `ProvisionedThroughputExceeded` error
	- use highly distributed partition
	- retries with exponential backoff
	- increase shards (scaling)

##### Consumers
- consume the records fom the stream 
- you can write your own:
	- KCL - Kinesis consumer library 
	- AWS SDK - classic or enhaced fan-out
- or used managed consumers (K Firehose, K Data Analytics, Lambda)
- shared - classic fan-out
	- throughput is 2MB/sec per shard (all consumers)
	- `GetRecords` api
	- better for low number of consuming apps
	- max 5 `GetRecords` api calls/sec
	- latency ~200 ms
	- returns up to 10MB (then throttle for 5 sec) or up to 10000 records
- enhanced fan-out
	- throughput is 2MB/sec per shard (per consumer)
	- `SubscribeToShard` api
	- multiple consuming applications for the same stream
	- latency ~70 ms
	- more expensive
	- kinesis pushes data to consumers over HTTP/2
	- soft limit of 5 consumer applications (KCL 2.x) per data stream

##### Lambda Consumers
- supports classic and enhanced fan-out consumers
- read records in batches
- can configure batch size and batch window
- if error occurs, lambda retries until success or data expiration
- can process up to `10` batches per shard simultaneously

##### KCL
- a Java lib that helps read record from a KDS with distributed apps sharing the workload
- each shard is to be read by only one KCL instance
- progress is checkpointed into DynamoDB
- KCL can run on EC2, E Beanstalk and on-prem
- records are read in order at the shard level
- versions:
	- KCL 1.x (supports shared consumer)
	- KCL 2.x (supports shared and enhanced fan-out consumer)

##### Shard Splitting
- used to increase the stream capacity
- used to divide a "hot shard"
- old shard is closed and will be deleted once the data is expired
- no automatic scaling in kinesis
- can't split a shard into more that 2 shards in a single operation

##### Merging Shards
- descrease the stream capacity and save costs
- can be used to group two shards with low traffic (cold shards)
- old shards are closed and will be deleted once the data is expired
- can't merge more than 2 shards in a single op


### Kinesis Data Firehose
- fully managed service (no administation, autscoling, serverless)
- pay for data going through firehose
- near real-time
	- 60 seconds min latency for non-full batches
	- or minimum 1MB data at time
- supports many data formats, conversions, transformations, compressoion
- can send failed or all data to a backup S3 bucket
- records (up to 1MB) are sent from source 
	- AWS IOT
	- apps
	- client
	- SDK, KPL
	- Kinesis Agent
	- Amazon CW
	- Data Streams
- once in KDF there is an optional lambda transformation
- KDF will then batch write to any of the valid destinations:
	- AWS Services:	
		- S3
		- Redshift (copy through s3)
		- OpenSearch
	- partner destinations (datadog, splunk, new relic, mongodb...)
	- custom destinations (http endpoints)

#### KDF v KDS
- KDS:
	- streaming service for ingest at scale
	- write custom code
	- you manage scaling
	- supports replay
	- data storage for 1 - 365 days
	- realtime
- KDF:
	- load streaming data
	- fully managed 
	- near real-time
	- auto-scaling
	- no data storage/ no replay

#### Kinesis Data Analytics

##### KDA for SQL
- use SQL statements to analyze data
- sources:
	- KDS
	- KDF
- sinks:	
	- KDS
	- KDF
- can use reference data from an S3 bucket
- fully managed, no servers to provision
- autoscaling
- pay for your data consumption rate
- use cases:
	- time-series analytics
	- realtime dashboards
	- realtime metrics

##### KDA for Apache Flink
- use Flink (java, scala, SQL) to process and analyze streaming data
- sources
	- KDS
	- MSK (managed kafka)
- runs on a managed cluster on AWS
- provisions compute resources, parallel computing, aut scaling
- app backups (checkpoints and snapshots)
- use any Flink programming features
- *does not* read from firehose

#### Ordering Data in Kinesis
- use the parition key to ensure the records are going into the same shards
- partition key is very similar to the group id for SQS

## Lambda
- scaling is automated
- pricing:
	- pay per request and compute time
- increasing RAM will also improve CPU and network
- easy to get more resources (up to 10 GB RAM per function)
- language support:
	- node.js
	- python
	- java (java 8)
	- C# (.NET Core and powershell)
	- Go
	- Ruby
	- Custom runtime API (community supported - Rust)
- lambda container image
	- must implement the lambda runtime API
	- ECS fargate is preferred
- main integrated services:
	- API gateway
	- Kinesis
	- DynamoDB
	- S3
	- Cloudfront
	- EventBridge
	- CW Logs
	- SNS/SQS
	- Cognito

### synchronous invocations (CLI, SDK, API Gateway, ALB)
- result is returned right away
- error handling must happen client side (retries, exponential backoff, etc)
- user invoked:
	- ELB (ALB)
	- API Gateway
	- CF
	- S3 batch
- service invoked:
	- Cognito
	- Step Functions
	- Lex
	- Alexa
	- KDF
		
### asynchronous invocations
- S3, SNS, CW events (eventbridge), CodeCommit, Codepipeline
- the events are placed in an event queue
- the lambda attempts to retry on errors:
	- 3 tries total
	- 1 min wait after first, then 2 minutes wait after 2nd
- make sure the processing is idempotent (in case of retries)
- it the function is retried, you will see duplicate log entries in the CW logs
- can define a DLQ (SNS or SQS) for failed processing
	- this requires `SendMessage` permissions to SQS on the Lambda IAM role
- async invocations allow you to speed up processing if you don't need to wait for the results (i.e. you need 1000 files processed)
 
### Lambda w/ ALB
- to expose a lambda as an http endpoint you can use an ALB
- lambda gets registered in a target group
- HTTP request gets converted to a JSON request payload
- Lambda response JSON then gets converted HTTP for the response

#### Multi-header values
- alb can support multi-header values
- these get converted as arrays with multiple values in json

### Lambda w/ Eventbridge
- CRON or rate eventbridge rule to trigger a lambda
- codepipeline eventbridge rule to trigger on state change

### Lambda w/S3
- S3 event notifications typically deliver events in seconds but sometimes can be a minute plus
- if two writes are made to a single non-versioned object at the same time, it is possible that only a single event notification will be sent
	- if you want to esnure that an event notification is sent for every successful write, enable versioning on your bucket
- automatically generated resource-based policy allows the S3 to invoke the lambda

### Event Source Mapping
- applies to:
	- Kinesis Data Streams
	- SQS and SQS FIFO queue
	- DynamoDB streams
- records need to be polled from the source
- your lambda is invoked synchronously

#### Streams (Kinesis, DynamoDB)
- an event source mapping creates an iterator for each shard, proceses items in order
	- iterator allows it to start with new items, from the beginning of the stream, or from a timestamp
- processed items are not removed from the stream (other consumers can read them)
- for low traffic streams you can use a batch window to accumulate records before processing
- you can process multiple batches in parallel
	- up to 10 batches per shard
	- in-order processing is still guaranteed for each partition key

#### Error handling
- by default, if your function returns an error, the entire batch is reprocessed until the function succeeds or the items in the batch expire
- to ensure in-order processing, processing for the affected shard is paused until the error is resolved
- you can configure the event source mapping to:
	- discard old events
	- restrict the number of retries
	- split the batch on error (to work around Lambda timeout issues)
- discarded events can go to a `Destination`

#### Queues (SQS and SQS FIFO)
- event source mapping will poll SQS (long mapping)
- specify batch size (1-10 msgs for FIFO, 1 -10,000 for std)
- recommended: set the queue visibility timeout to 6x the timeout of your lambda function
- to use a DLQ, configure it on the SQS queue (not lambda)
	- DLQ for lambda is only for async invocations
- lambda also support in-order procesing for FIFO queues, scaling up to the number of active message groups
- for standard queues, items aren't necessarily processed in order
- lambda scales up to process a standard queue as quickly as possible
- when an error occurs, batches are returned to the queue as individual items and might be procesed in a different grouping than the original batch
- occassionally, the event source mapping might receive the same item from the queue twice, even if no function error returned
- lambda deletes the items from the queue after they're processed successfully

#### Scaling
- KDS and DynamoDB
	- one lambda invocation per stream shard
	- if you use parallelization, up to 10 batches processed per shard simultaneously
- SQS standard
	- lambda adds up to 60 more instances per minute to scale up
	- up to 1000 batches of messages processed simultaneously
- SQS FIFO
	- messages with the same GroupID are processed in order
	- the lambda function scales to the number of active message groups

### Event and Context Objects
- Event:
	- JSON doc that contains data for the func to process
	- contains data from the invoking service
	- Lambda runtime converts the event to an object (i.e. dict type in python)
- Context:
	- provides methods and properties that provide info about the invocation, function, and runtime environment
	- passed to your function by lambda at runtime
	- example: `aws_request_id`, `function_name`, etc

### Destinations
- send result of an async invocation or a failed source mapper
- async invocations can define destinations for both a successful or failed event
	- AWS recommends you use destinations over DLQ now for greater flexiblity
- Source mapping: destinations only for discarded event batches
- for SQS you can either use destinations or a DLQ from the source SQS queue

### Lambda Exec Role (IAM)
- grants the lambda func perms to AWS services/resources
- there are also managed policites available for Lambda
- when you use an *event source mapping* to invoke your function, Lambda uses an exec role to read event data
- best practice: create one lambda exec role per func

### Lambda Resource Based policies
- uses resource-based policies to give other accounts and AWS services permission to use your lambda resource
	- (similar to S3 bucket policies)
- an IAM principal can access lambda:
	- if the IAM policy attached to the principal authorizes it
	- OR if the resource-based policy authorizes it

	### Lambda Envvars
	- helpful to store secrets (encrypted by KMS)
	- Lambda service adds its own system envvars
	- allows you to adjust function behavior without updating code

	### Lambda Monitoring and Monitoring
	- CW logs
		- exec logs are stored in CW logs
		- make sure your func has an exec role with an IAM policy that authorizes writes to CW logs
	- CW Metrics
		- lambda functions all emit metric
		- invocations, durations, concurrent execs, error cts, success rates, thrttles
		- async delivery failure
		- iteratore age (Kinesis and DynamoDB streams) 
	- Tracing
		- enable active tracing in the config
		- use the AWS X-Ray SDK in cokde
		- ensure the func has a correct IAM exec role to write to Xray Daemon
	- needs envvars to communicate w/ x-ray

### Lambda @Edge
- edge functions: code your write and attach to CF distributions to minimize latency
- CF provides 2 types: CF Functions and Lambda@Edge
- use cases: 
	- customize CDN content
	- dynamic web app at edge
	- SEO
	- intelligently route across origins and DCs
	- bot mitigation
	- RT image transformation
	- user auth
	- A/B testing
	- user prioritization
	- user tracking + analytics
- only pay for what you use

#### CF Functions
- lightweight funcs written in JS
- high-scale, latency sensitive CDN customizations
- used to change `Viewer` requests and responses
- native feature of CF (code managed entirely within CF)
- *millions* of reqs/sec
- use cases:
	- Cache key normalizations
	- header manipulations
	- URL rewrites or redirects
	- request auth

#### Lambda @ Edge
- written in NodeJS or Python
- scales to 1000s of reqs/second
- used to change CF requests/responses (both `Viewer` and `Origin`)
- author your functions in one region, then CF replicates to its locations
- longer exec time than CF functions
- adjustable resource allocation
- your code depends on 3rd party libraries
- network access to use external services for processing
- file system access or access to the body of HTTP requests

### Lambda by default
- launches outside your own VPC
	- cannot access resources in your VPC
- can be launched in your VPC
	- define the VPC ID, subnets, and security groups
	- lambda will create an ENI in your subnets
	- requires `AWSLambdaVPCAccessExecutionRole`
	- by default has no internet access (regardless of private/public subnet)
		- can deploy a NAT gateway/instance to provide internet access for a lambda in a private subnet
	- can use VPC endpoints to privately access AWS services without a NAT
	- *note*: lambda -CW logs works regardless of VPC endpoint or NAT gateway

### Function config
- RAM (function is allocated CPU proportional to the memory setting):
	- from 128MB to 10GB in 1MB increments
	- the more RAM you add, the more vCPU credits you get
	- at 1792 MB, a func has the equivalent of 1 vCPU
	- after 1792 MB you get more than 1 vCPU and need to use multithreading in your code to benefit
- if your app is CPU bound, increase RAM to improve performance
- timeout: default is 3 seconds, max is 900 seconds (15 mins)

### Exec Content
- temporary runtime env that initializes any external depencies of your lambda code
- great for DB connections, HTTP clients, SDK clients, etc
- exec context is maintained for some time in anticipation of another lambda function invocation
- the next function invocation can "re-use" the context to execution time and save time in initializing connections objects
- exec context includes the `/tmp` directory
- initialize reusable components (i.e. DB connections) *outside of the handler function* so they can be reused in multiple executions
- `/tmp` use cases:
	- download a large file to work across multiple execs
	- disk space ops
	- max space is 10 GB
	- directory content remains hen the exec context is frozens, providing transient cache that can be used for multiple invocations
	- for permanent persistence, you would need to use S3
	- to encrypt `/tmp` you can generate KMS keys to do so

### Lambda Layers
- custom runtimes (i.e. Rust, C++)
- externalize dependencies to re-use them
	- app package is self contained
	- layers represent external dependencies that don't need to be repacked each time
	- layers can be referenced across mutliple functions

### FS Mounting
- funcs can access EFS file systems if they are in a VPC
- configure lambda to mount the EFS systems to a local dir during init
- must leverate EFS access points
- limitations: EFS has connection limits (on function instance = one connection) and connection burst limits

### Concurrency and Throttling
- concurrency limit: up to 1000 concurrent executions of lambdas across your account
- can set a "reserved concurrency" at the function level (= limit)
- each invocation over the concurrency limit will trigger a "throttle"
- throttle behavior:
	- if synchronous = return `ThrottleError - 429`
	- if async = retry automatically for up to 6 hours, if another throttle then go to DLQ
		- retry interval increases exponentionally from 1 second after the first attempt to a max of 5 mins
- higher limits on lambda funcs can be requested through an AWS support ticket
- if you don't set a concurrency limit on your functions:
	- one function can hog all of the concurrent executions resulting in a throttle for other functions

### Cold Starts and Provisioned concurrency
- cold start:
	- code is loaded and the code outside the handler is run (init)
	- if the init is large, this can be time consuming
	- first request served has higher latency than the rest
- provisioned concurrency:
	- concurrency is allocated before the function is invoke (in advance)
	- cold start never happens and all invocations have low latency
	- app autoscaling can managed concurrency (schedule or target utilization)

### Dependencies
- if your func depends on external libs, you need to install the packages alongside your code and zip it together
- upload the zip straight to lambda if less than 50MB, else to S3 first
- native libraries work - they need to be compiled on amazon linux
- AWS SDK comes by default with every lambda

### Lambda <> CF
- can use inline functions for smaller funcs (no dependencies)
- otherwise store the Lambda zip in S3 and reference the S3 func in your CF
- can launch Lambdas from S3 across multiple accounts
	- launch the CF in the destination account
	- add a bucket policy in the source account to allow principal (other AWS account IDs) to access the bucket
	- add an execution role in the destination account to allow `Get` and `List` S3 buckets 

### Lambda Container Images
- deploy lambda function as a container (up to 10GB) from ECR
- pack complex/large dependencies in a container
- base image must implement the `Lambda Runtime API`
- can test the containers locally using the Lambda Runtime Interfaces Emulator

#### Best Practices
- use the AWS-provided base images
	- stable, built on Amazon Linux 2, cached by lambda service
- use multi-stage builds
	- build your code in larger preliminary images, copy only the artifacts you need in your final container image (discard the preliminary steps)
- build from stable to frequently changing in the Dockerfile (top down)
- use a single repo for functions with large layers
	- ECR compares each layer of a container image when it is pushed to avoid uploading and storing duplicates

### Lambda Versions
- `$LATEST` version is mutable
- when the code is stable, you publish a version which becomes immutable
- versions increase in number
- each version is assigned an ARN
- version = code + configuration

### Aliases
- "pointers" to Lambda functions versions
- aliases are mutable
- aliases enable canary deployments by assigning weights to lambda functions
- enable stable config of event triggers/destinations
- aliases also have their own ARNs
- aliases *cannot* reference other aliases

### Lambda <> CodeDeploy
- CodeDeploy can help automate traffic shift for lambda aliases
	- allows you to specify the strategy for traffic shifting
	- i.e `Linear`, `Canary`, `AllAtOnce`
- feature is integrated within the SAM framework
- can create pre and post traffic hooks to check the health of the lambda function

### Function URL
- dedicated HTTP(S) endpoint for your lambda
- a unique URL endpoint is generated
	- `https://<URL_ID>.lambda-url.<REGION>.on.aws` (dual-stack)
- invoke via web browser, curl, postman, etc
- accessible only through public internet
	- doesn't support PrivateLink
- supports resource-based policies and CORS configs
- `AuthType`
	- `NONE` = allow public and unauthenticated access (resource-based policy is ALWAYS in effect - must grant public access)
	- `AWS_IAM`:
		- both principal's identity-based policy AND resource-based policy are evaluated
		- principal must have `lambda:InvokeFunctionUrl` permissions
		- same account = identity-based OR resource-based policy as allow
		- cross account = identity-based AND resource-based policy as allow
- can be applied to any function alias or to `$LATEST`
- create and configure using AWS Console or API
- throttle your function using reserved concurrency

### Lambda <> CodeGuru Profiling
- gain insights into runtime performance of your Lambda funcs using CodeGuru profiler
- CodeGuru creates a Profiler Group for your lambda func
- supported for Java and Python runtimes
- activate from AWS lambda console
- when acitivated, Lambda adds 
	- CodeGuru profiler layer to your function
	- envvars to your function
	- `AmazonCodeGuruProfilerAgentAccess` policy to your function

### Lambda Limits (per region)
- Execution:	
	- memory allocation: 128 MB - 10GB (1 MB increments)
	- max exec time is 15 minutes
	- envvars (4KB)
	- concurrency executions: 1000 (can be increased)
- Deployment:
	- lambda func deployment size (compressed .zip): 50MB
	- size of uncompressed deployment (code + deps): 250 MB
	- can use the /tmp directory to load other files at startup

### Best Practices
- perform heavy-duty work *outside of* function handler:
	- connect to DBs
	- init the AWS SDK 
	- pull in dependencies
- use envvars for:
	- conn strings, S3 Bucket,etc
	- pws, sensitive vals can be encrpyted using KMS
- minimize your deployment package size to its runtime necessities:
	- break down the function if need be
	- remember the AWS lambda limits
	- use layers where necessary
- avoid using recursive code (never have a lambda function call itself)

## DynamoDB
- NoSQL serverless DB

### RDBMS
- vertical scaling (increasing RAM/CPU/IO)
- horizontal scaling (can increase read capability by adding read replicas)
- SQL querying

### NoSQL
- non-relational, distributed databases
- do not support query joins
- all the data is present in one row
- don't perform aggregations such as "SUM", "AVG"
- scale horizationtally

### Basics
- fully managed, highly available with replication across multiple AZs
- scales to massive workloads (millions of reqs/sec, trillions of rows, 100s of TB of storage)
- fast and consistent perf (low latency on retrieval)
- integrated with IAM for security, auth, and administration
- low cost and auto-scaling capabilities
- Standards & Infrequent Access (IA) table classes
- made of `Tables`
- each table has a `Primary Key`
- each table can have an infinite number of items (=rows)
- each item has attributes (can be added over time and `null` is ok)
- maximum size of an item is `400KB`
- support data types:
	- scalar types - string, number, binary, boolean, null
	- document types - list, map
	- set types -  string set, number set, binary set


### Primary Keys
- option 1: Partition Key (HASH)
	- partition key must be unique for each item
	- partition key must be "diverse" so that the data is distributed
	- e.g. "User_ID" for a users table
- option 2: Partition Key + Sort Key (HASH + RANGE)
	- combination must be unique for each item
	- data is grouped by partition key
	- e.g. usersgames table, "User_ID" for partition key and "Game_ID" for sort key

### Read/Write Capacity Modes
- control how you manage your table's capacity (read/write throughput)

#### Provision Mode (default)
- table must have provisioned read and write capacity unites
- RCU - throughput for reads
- WCU - throughput for writes
- option to setup auto-scaling of throughput to meet demands
	- set min/max levels for provisioned RCUs/WCUs
- throughput can be exceeded temporarily using "Burst Capacity"
- if burst capacity has been consumed, you'll get a `ProvisionedThroughputExceededException`
	- at this point it's advised to use an exponential backoff retry

##### WCU (write capacity units)
- 1 WCU represents one write/sec for an item up to 1 KB in size
	- if the items are larget than 1 KB, more WCUs are consumed
	- KBs are rounded to the upper KB

##### RCU (read capacity units)
- 1 RCU represents 1 strongly conistent read per second OR 2 eventually consistent reads per second, for an item up to 4KB in size
	- if the items are larger than 4KB, more RCUs are consumed
- item sizes are rounded up to the next 4KB increment for calcing RCU needs

###### Strongle Consistent Read vs Eventually Consistent Read
- eventually consistent: possibility we'll get stale data because of replication lag
- strongly consistent: consumes twice the RCU, but we'll always get the most recent, correct data

###### Partitions (internal)
- data is stored in paritions
- partition keys go through a hashing algorithm to know which partition they belong to
- to compute the number of partitions:
	- `# partitions by capacity = (RCUs total / 3000) + (WCUs total / 1000)`
	- `# partitions by size = (Total Size)/ 10 GB`
	- `# of partitions = ceil(max(# partitions by capacity, # partitions by size))`
- WCUs and RCUs are spread evenly across partitions

###### Throtting
- if we exceed provisioned RCUs or WCUs, we get `ProvisionedThroughputExceededException`
- Reasons:
	- hot keys - one partition is being read many times
	- hot partitions
	- very large items
- Solutions:
	- exponentions backoff when exception is encountered (already in SDK)
	- distribute partition keys as much as possible
	- if RCU issue, we can use DynamoDB accelerator (DAX)

#### On Demand Mode
- read/writes scale up/down with your workloads
- no capacity planning needed (WCU/RCU)
- unlimited WCU & RCU (no throttle but 2.5x more expensive)
- charged for reads/writes in temrs of RRU and WRU
- RRU (Read Request Units) - throughput for read (same as RCU)
- WRU (Write Requeset Units) - throughput for writes (same as WCU)
- use cases: unknown workloads, unpredictable app traffic

### Writing Data
- `PutItem`
	- creates a new item or fully replace an old item (same primary key)
	- consumes WCU
- `UpdateItem`
	- edits an existing item's attributes or adds a new item if it doesn't ecist
	- can be used to implement atomic counters - a numeric attribute that's unconditionally incremented
- `Conditional Writes`
	- accept a write/update/delete only if conditions are met, otherwise returns an error
	- helps with concurrent access to items
	- no performance impact
	- can specify a Condition expression to determine which items should be modified based on:
		- `attribute_exists`
		- `attribute_not_exists`
		- `attribute_type`
		- `contains` (for strings)
		- `IN` keyword and `BETWEEN` keyword
		- `size` (string length)
### Reading Data
- `GetItem`
	- read based on primary key
	- primary key can be `HASH` or `RANGE + HASH`
	- eventually consistent read (default)
	- option to use Strongly Consistent Reads (more RCU - might increase latency)
	- `ProjectExpression` can be specified to retrieve only certain attributes
- `Query`
	- returns items based on:
		- `KeyConditionExpression`
			- partition key values (must be = operator) - required
			- sort key value (comparator operator OR `Between`, `Begins With`) - optional
		- `FilterExpression`
			- additional filtering after the query operation (before the data is returned)
			- use only with *non-key attributes* (does not allow `HASH` or `RANGE` attributes)
	- Returns:
		- the number of items specified in `Limit`
		- OR up to 1 MB data
	- ability to paginate results
	- can query a table, a local secondary index, or a global secondary index
- `Scan` the entire table and then filter out data (inefficient)
	- returns up to 1MB data (use pagination to keep on reading)
	- consumes a lot of RCU
	- limit impact using `Limit` or reduce the size of the result and pause
	- for faster performance use `Parallel Scan`
		- multiple workers can scan multiple data segments at the same time
		- increases throughput and RCU consumed
	- can use `ProjectionExpression` and `FilterExpression` (no changes to RCU)
- `DeleteItem`
	- delete an individual item
	- ability to perform a conditional delete
- `DeleteTable`
	- delete a whole table and all items (much quicker than calling DeleteItem on all items)

### Batch Operations
- allows you to save in latency by reducing the number of API calls
- operations are done in parallel for better efficiency
- part of a batch can fail, in which case we need to retry for the failed items
- `BatchWriteItem`
	- up to 25 `PutItem` or `DeleteItem` in one call
	- up to 16 MB of data written, up to 400KB of data per item
	- `UnprocessedItems` for failed writes (solution: use exponential backoff or add WCU)
- `BatchGetItem`
	- return items from one or more tables
	- up to 100 items, up to 16 MB of data
	- items retrieved in parallel to minimize latency
	- `UnprocessedKeys`	for failed read operations (solution: use exponential backoff or add RCU)

### PartiQL
- SQL-compatible query language for DynamoDB
- allows you to `select`, `insert`, `update`, and `delete` using SQL
- run queries across multiple DynamoDB tables

### Local Secondary Index (LSI)
- alternative sort key for your table
- sort key consists of one scalar attribute (String, Number or Binary)
- up 5 local secondary indexes per table
- must be defined at table create time
- attribute projections - can contain some or all attributes of the base table (KEYS_ONLY, INCLUDE, ALL)

### Global Secondary Index (GSI)
- alternative primary key from the base table
- speed up queries on non-key attributes
- the index key consists of scalar attributes (String, Binary, or Number)
- attribute projections - can contain some or all attributes of the base table (KEYS_ONLY, INCLUDE, ALL)
- must provision RCUs and WCUs for the index
- can be added/modified after table creation

### Indexes and Throttling
- GSI
	- if the writes are throttled on the GSI, then the main table will be throttled
- LSI
	-	uses the WCUs and RSUs of the main table
	- no special throttling considerations

### Optimistic Locking
- a strategy to ensure an item hasn't changed before you delete/update it
- each item has an attribute that acts as a *version number*

### DynamoDB Accelerator (DAX)
- fully-managed, highly-available, seamless in-memory cache for DynamoDB
- microsecond latency for cached reads and queries
- doesn't require app logic modification (same APIs as dynamo)
- solves the "hot key" problem(too many reads)
- 5 min TTL for cache (default)
- multi-az (3 nodes min recommended for prod)


#### DAX vs ElastiCache
- can be combined for a solution
- DAX for individual objects cache, query+scan cache
- ElastiCache for aggregated results

### DynamoDB Streams
- ordered stream of item-level modifications in a table
- stream records can be:
	- sent to kinesis data streams
	- read by AWS Lambda
	- read by KCL applications
- data retention up to 24 hours
- use cases
	- react to changes in real-time
	- analytics
	- insert into derivative tables
	- insert into opensearch service
	- implement cross-region replication
- ability to choose the info that will be written to the streams
	- `KEYS_ONLY`
	- `NEW_IMAGE` - the entire item as it appears *after* it was modified
	- `OLD_IMAGE` - the entire item as it appears *before* it was modified
	- `NEW_AND_OLD_IMAGES`
- streams are made of shards, just like KDSs (user does not provision shards - this is automated by AWS)
- records are *not retroactively populated in a stream* after enabling it
- Streams <> Lambda
	- need to define an `Event Source Mapping` to read from streams
	- need to ensure the Lambda has appropriate permissions
	- lambda is invoked synchronously

### TTL
- automatically delete items after an expiry date
- doesn't consume any WCUs
- the ttl attribute must be a `Number` with `Unix expoch timestamp` value
- expired items are deleted within 48 hours of expiration
- expired items that haven't been deleted still appear in reads/queries/scans
- a delete op for each expired item enters the DB streams
- use cases:
	- reduce stored data
	- adhere to regulation


### CLI
- `--projection-expression` = select which attributes you would like to have returned
- specify `--max-items` to avoid timeouts

### Transactions
- coordinated all-or-nothing operations (add/update/delete) to multiple items across one or more tables
- provides Atomicity, Consistency, Isolation, and Durability (ACID)
- Read Modes: eventual consistency, strong consistency, transactional
- Write Modes: standard, transactional
- Consumes 2x WCUs or RCUs
	- dynamo performs 2 ops for every item (prepare & commit)
- two ops:
	- `TransactGetItems` & `TransactWriteItems`
- use cases: financial transactions, managing order, multiplayer games

### DynamoDB as a Session State Cache
- common use case
- vs ElasiCache
	- EC is in-mem
	- dynamodb is serverless
	- both are key/value stores
- vs EFS
	- EFS must be attached to EC2s as a network drive
- vs EBS & Instance Store
	- EBS and IS can only be used for local caching (not shared)
- vs S3
	- S3 is higher latency and not meant for small objects

### Write Sharding
- a strategy that allows better distribution of items evenly across paritions is to append a suffix to a partition key value
- two methods:
	- sharding using random suffix
	- sharding using calc'd suffix

### Large Objects pattern
- S3 bucket contains the large object, DynamoDB contains the metadata (i.e. `image_url` attr of where the item lives in s3)
- s3 add can invoke a lambda which then updates the object's metadata in dynamoDB

### table operations
- copying a DB tables
	- option 1 to use AWS data pipeline
	- option 2 backup the dynamo DB table and restore it into a new table (time consuming)
	- option 3 write your own code to use the APIs to copy the table

### Security + Misc
- security:
	- VPC endpoints available to access dynamoDB w/o the internet
	- access fully controlled by IAM
	- encrpytion at rest using AWS KMS and in transit TLS
- backup and restore
	- PITR (point-in-time-recovery) like RDS
	- no performance impact
- global tables
	- multi-region, multi-active, fully replicated, high performance
- AWS DB migration service can be used to migrate to Dynamo from other DB services
- for services that interact with Dynamo directly:
	- can use IdPs to generate temporary AWS creds so users can obtain an IAM role w/ necessary permissions
	- conditions of the IAM role can limit the users access to just the data they control and limit the attributes they can see

## API Gateway
- serverless offering that allows you to create REST APIs that are public/accesible
- Lambda + Gateway = no infra
- support for Websockets
- supports versioning and env tagging
- security (Authnetication and Authorization)
- create API keys to handle request throttling
- swagger/open API import for rapid development
- transform and validate requests/resonses
- generate SDK and API specs
- cache API responses
- max reqest time is 29 seconds

### Integrations
- Lambda
- HTTP
	- expose HTTP endpoints in the backend on an ALB or on prem server
- AWS Services
	- post to AWS step functions, SQS, KDS

### Endpoint Types
- edge-optimized (default): for global clients
	- requests are routed through CF edge locations for improved latency
	- GW still only lives in one region
- Regional
	- for clients within the same region
	- can manually combine w/CF for more control over caching strategies and distribution
- Private
	- can only be accessed from your VPC using an interface VPC endpoint (ENI)
	- use a resource policy to define access

### Security
- User Authentication
	- IAM roles (internal apps)
	- Cognito (identity for external users)
	- Custom Authorizer (lambda - your own logic)
- Custom domain name HTTPS through ACM
	- if using edge-optimized, the cert must be in us-east-1
	- if using regionial, cert must be in the API gateway region
	- must setup CNAME or A-alias record in route 53

### Deployment Stages
- changes to the gateway will not be effective until they are deployed
- each stage has its own config params
- stages can be rolled back (history of deployments is preserved)
- `stage variables` are like envvars for API gateway
	- must manually add permissions to the lambda aliases allowing the API gateway to invoke
- can be used in:
	- lambda function ARN
	- http endpoint
	- parameter mapping templates
- format: `${stageVariables.<VAR_NAME>}`
- at the stage level you can define
	- WAF
	- throttling
	- cache settings
	- client certs
	- monitoring

### Canary Deployment
- choose the % of traffic the canary channel receives
- telemetry is separate for better monitoring
- possibility to override stage vars for canary

### Integration Types
- MOCK: gateway returns a response without sending the req to the backent
- HTTP / AWS
	- must configure the integration request and response
	- setup data mapping using mapping templates for req/resp
- AWS_PROXY (Lambda)
	- incoming request from the client is input to the Lambda
	- function is responsible for the logic of req/rep
	- no mapping template -> headers, query string params are passed as args to the lambda
- HTTP_PROXY
	- no mapping template
	- HTTP req is passed to the backend
	- HTTP res from backend is forwarded by API gateway
	- possibility to add headers if needed

#### Mapping Templates (AWS & HTTP integration)
- mapping templates can be used to modify req/resp
- rename/modify query string params
- modify body content
- add headers
- uses Velocity Template Language (VTL)
- filter output results (remove unnecessary data)
- `Content-Type` can be either `application/json` or `application/xml`

### OpenAPI spec
- common way of defining REST apis, using the api def as code
- can import existing openAPI 3.0 spec to API gateway
- can export current API as openAPI spec
- specs can be written in YAML or JSON
- can use OpenAPI to generate an SDK for your apps
- can configure API gateway to perform basic validation of an API request before proceeding with the integration request
- when the validation fails, API gateway fails the request and returns a `400`
- reduces unnecessary calls to the backend
- checks:
	- required request params are in the URI, query string, and headers of an incoming request are included
	- the applicable req payload adheres to the configured JSON schema request model of the method

### Caching
- reduces # of calls to the backend
- default TTL is 300 seconds (min: 0s, max: 3600s)
- caches are defined per stage
- possible to override cache settings per method
- cache encryption options
- capacity: bw `0.5GB` and `237GB`
- cache is *expensive*
- invalidation:
	- ability to flush the entire cache (invalidate it) immediately
	- clients can invalidate the cache with header: `Cache-Control: max-age=0` (with proper IAM auth)
	- if you don't impose an `InvalidateCache` policy or choose require auth checkbox in the console, any client can invalidate the cache

### Usage Plans & API keys
- to make an API available as an offering ($) for customers
- usage plan:
	- who can access one or more deployed API stages/methods
	- how much and how fast they can access them
	- use API keys to identify clients and meter access

#### How To Deploy
- configure a use plan:	
	1. create one or more apis, require those to require an API key -> deploy the APIs to stages
	2. generate or import API keys to distribute to app developers who will use your API
	3. create the usage plan w/ desired throttle and quota limits
	4. associate API stages and API keys w/ the usage plan
- callers of the API must supply an assigned API key in the `x-api-key` header in their API request

### Logging/ Tracing
- CW Logs
	- contains info about req/resp body
	- enable CW logging at the stage level
	- can override settings on a per API basis
- X-ray can be enabled
- Metrics
	- `CacheHitCount` & `CacheMissCount` - efficiency of the cache
	- `Count` - total # of reqs
	- `IntegrationLatency` - time bw GW relays a req to backend and receives a response
	- `Latency`	- totaly time bw when GW receives a request from the client and returns a response to the client (include `IntegrationLatency` and other GW overhead)
	- `4XXError` (client-site) & `5XXError` (server-side)

### Throttling
- account limit
	- GW throttles reqs at 10,000 rps across ALL apis
	- soft limit (can be increased upon request)
	- will return `429 Too Many Requests` (retryable error)
- can set Stage limit & Method limits to improve performance
- can define usage plans to throttle per customer

### CORS
- CORS must be enabled to receive API calls from another domain
- `OPTIONS` preflight req must contain the following headers
	- `Access-Control-Allow-Methods`
	- `Access-Control-Allow-Headers`
	- `Access-Control-Allow-Origin`
- CORS can be enabled through the console
	- in lambda proxy, the headers will not be automatically added, need to add the `Access-Control-Allow-Origin` header in the lambda function log

### Security
- IAM permissions
	- create an IAM policy auth and attach to user/role
	- good to provide access within AWS
	- leverages `Sig v4` capability where IAM creds are in headers
- Resource Policies (can be combined with IAM)
	- set json policy on the gateway to define access
	- allow for Cross-Account access 
	- allow for specific IPs
	- allow for a VPC endpoint
- Cognito User Pools
	- cognito manages user lifecycle and token expiration
	- api gateway verifies identity automatically from Cognito
	- no custom implementation required
- Lambda Authorizer
	- token-based authorizer (bearer token  i.e. JWT or OAUTH)
	- request parameter based Lambda authorizer (headers, query string, stage var)
	- lambda must return an IAM policy for the user, result policy is cached
	- Authentication (external 3rd party auth system) | Authorizer (lambda func)

### HTTP v REST
- HTTP APIs
	- low-latency, cost-effective AWS lambda proxy, HTTP proxy apis and private integration (no data mapping)
	- support OIDC and Oauth 2.0 authorization
	- built in CORS support
	- no usage plans/API keys
- REST APIs
	- more robust, include all features covered 

### Websocket
- enables stateful app use cases
- two-way interactive communication bw server and browser
- server can push data to the client
- works with AWS services
- choose a Route Selection Expression (i.e. `$request.body.action`) to determine how to route the web socket request from the request attribute

### Architecture
- create a single interface for all microservices within your company using API Gateway
- use API endpoints for various resources

## CI/CD (CodeCommit, CodePipeline, CodeBuild, CodeDeploy)
- continuous integration:
	- devs push code to a code repo
	- testing/build server checks the code as soon as its pushed and provides the result of the tests
	- find bugs early
- continuous delivery
	- software can be released reliably whenevre needed
	- ensure deployments happen often, quickly
	- often uses automated depoyment methods (spinnaker, codedeploy, jenkins CD)

### CodeCommit
- version control using Git
- private git repos
- no size limits
- fully manged, highly available
- code is *only* in AWS Cloud acct
- security
- integrates with other CI tools
- can set up notifications and triggers

#### Security
- standard git cli
- authentication:
	- ssh keys
	- HTTPS
- Authorization:
	- IAM policies to manage users/rols perms to repos
- Encryption:
	- Repos are encrypted at rest using KMS
	- encrypted in transit (can only use SSH or HTTPS to commit)
- Cross-acct access
	- use an IAM in your account and use STS `AssumeRole` API

### CodePipeline
- visual workflow to orchestrate CICD
- Source - CodeCommit, ECR, S3, BitBucket, GH
- Build - CodeBuild, Jenkins, CloudBees, TeamCity
- Test - CodeBuild, AWS Device Farm, 3rd party tools
- Deploy - CodeDeploy, EB, CF, ECS, S3
- Invoke - Lambda, Step functions
- consists of stages
	- each stage can have sequential and/or parallel actions
	- manual approval can be defined at any stage
	- action groups fall within stages (stages can be composed of multiple action groups)

#### Artifacts
- each pipeline stage can create `artifacts`
- artifacts are store in an S3 bucket and passed to the next stage

#### Troubleshooting
- can use events for pipeline failures
- ensure the IAM service role has permissions to perform the actions
- CloudTrail can be used to audit API calls

#### Events v Webhooks v Polling
- events *recommended* (i.e. new commits) trigger a codepipeline build
	- codestart source connection can be used to trigger from GH events
- webhooks
	- your pipeline exposes a webhook which allows it to be triggered from custom scripts
- polling
	- CP polls an event source for changes

### CodeBuild
- Source: CodeCommit, S3 , bitbucket, GH
- build instructions: file `buildspec.yml`
	- set at the root of your code
	- define envvars (can use secrets-manager or parameter-store)
	- phases (install, pre_build, build, post_build)
	- artifacts
	- cache (dependencies)
- output logs can be shipped to s3 or CW logs
- use CW metrics to monitor build stats
- use eventbridge to detect failed builds and trigger notifications
- use CW alarms to notify if you need thresholds for failures
- build projects can be defined within CP or CB
- if necessary you can use S3 to cache intermediate artifacts
- output artifacts to S3
- can run locally w/docker for debugging purposes
- by default, CB containers are launched outside VPC
- can specify a VPC config (ID, subnet, SGs)
	- then your build can access resources within VPC

#### Supported Envs
- Java, Ruby, Python, Go, Node.JS, android, .NET core, PHP
- Docker can be used to extend any env you like

### CodeDeploy
- deployment service that automates app deployment
- supports EC2, on-prem, lambda funcs, ECS services
- automated rollback capability on failed deploys or CW alarms
- gradual deployment controls
- `appspec.yaml` defines how the deployment occurs

#### EC2/ on-prem
- perform in-place deployments or blue/green
- must run the `CodeDeploy agent` on the target instances
- define deployment speed
	- `AllAtOnce`
	- `HalfAtATime`
	- `OneAtATime` - slowest but low impact
	- `Custom` (define your desired %)
- Blue/Green
	- spin up a new ASG, update the ASG to point to the new ASG then takedown the old one
- instances need sufficient permissions to access S3 to retreive the deployment bundles
- can us hooks to verify the deployment after each deployment phase

##### EC2 w/ASG
- in place:
	- updates existing ec2s
	- newly created EC2 instances by an asg will also get automated deployments

#### Lambda
- CD can help you automate traffic shift for Lambda aliases
- feature is integrated within the SAM framework
- different strategies
	- `LambdaLinear` - grow traffic every N mins to 100%
	- `LambdaCanary` - try X percent, then 100%
	- `AllAtOnce` - immediate

#### ECS
- automate deployment of ECS task def
- only blue/green deployments (reuires ALB)
	- `ECSLinear` - grow traffic every N mins to 100%
	- `ECSCanary` - try X percent, then 100%
	- `AllAtOnce` - immediate

#### Redeploy and Rollback
- rollback = redeploy a previously deployed revision of your app
- depoyment rollback triggers
	- automatically via CW Alarms or deployment fail
	- OR you can disable rollback
- when a rollback happens, CD redeploys the last known "good" revision *as a new deployment*

#### Troubleshooting
- `InvalidSignatureException` can be triggered from datetime mismatches from timestamp in CD and the server time
- check codedeploy agent log files to understand deployment issues

### CodeStar
- integrate solutions that groups: GH, CodeCommit, CodeBuild, CodeDeploy, CloudFormation, CodePipeline, CW
- quickly ready "ci/cd ready" projects for EC2, lambda, EB
- support langs: C#, Go, HTML 5, Java, Node.js, PHP, python, Ruby
- issue tracking wth Jira/GH issues
- ability to integrate Cloud9 to obtain a web IDE (not all regions)
- one dashboard to view all components
- free (pay for underlying services)
- limited customization

### CodeArtifact
- storing and retrieving code dependencies is called artifact mgmt
- secure, scalable and cost-effective artifact mgmt system for software dev
- works with common dependency mgmt tools (maven, gradle, npm, yarn, twine, pip, NuGet)
- devs and codebuild can then retrieve deps straight from CodeArtifact
- define domains
	- each domain is composed of repos
	- request is proxied from CA to public artifact repos (or custom sources) and cached within CA
- resourcepolicy can be used to authorize another account to access CA
	- principals are given all or nothing access to packages in repos

#### CA <> EventBridge
- event is created when a package is created, modified or deleted
- can be used to invoke other AWS services, including CodePipeline

#### Upstream Repos
- a repo can have another CA repo as an `upstream` repo
- allows a package manager client to access packages that are contained in more than one repo using a single repo endpoint
- up to 10 upstream repositories
- allows up to 1 external connection as well
	- connection BW CA repo and an external/public repo
	- allows you to fetch packages that are not already present within your CA repo
	- can create many repos if you want many external connections
	- retention: if a requested package version is found in an upstream rpo, a reference to it is retained and it is always available from the downstream repo
		- retained package is not affected by changes to the upstream repo
		- intermediate repos do not keep the package
	
#### Domains
- deduplicated storage - asset only needs to be store once in a domain, even if it's available in many repos
- allows for sharing packages across AWS accounts
- fast copying - only metadata records are updated when you pull packages from an upstream repo into a downstream one
- easy sharing across repos and teams - all assets and metadata in a domain are encrypted with a single KMS key
- apply policy across multiple repos - domain admin can apply policy across the domain such as 
	- restricting which accounts have access to repos in the domain
	- who can configure connections to public repos

### CodeGuru
- ML-powered service for automated code reviews and app performance recommendations
- two functionalities
	- Reviewer: automated code review for static code analysis
		- identify critical issues, security vulnerabilities and hard-to-find bugs
		- supports Java/python
	- Profiler: visbility/recommendations about app performance during runtime
		- identify and remove code inefficienies
		- decrease compute costs
		- provides heap summary and anomaly detection

#### Agent Config
- `MaxStackDepth` - max depth of the stacks represented in the profiler
- `MemoryUsageLimitPercent`
- `MinimumTimeForReportingInMilliseconds`
- `ReportingIntervalInMilliseconds`
- `SamplingIntervalInMilliseconds` - reduce to have a higher sampling rate

### Cloud9
- cloud-basd IDE
- code editor, debugger, terminal in browser
- prepacked with tools for popular programming langs
- share dev env with your team
- fully integrated with SAM and Lambda

## SAM (Serverless Application Model)
- package and deploy
	- `aws cloudformation package` / `sam package`
	- `aws cloudformation deploy` / `sam deploy`
- allows you to build and deploy locally for testing and debugging
- build on CloudFormation
- requires the `Transform` and `Resources` section
- fully integrated with code deploy
- use SAM policy templates for easy IAM policy definition

### SAM Policy Templates
- list of templates to apply permissions to your lambda functions
- important examples:
	- `S3ReadPolicy` - gives read only perms to objects in SQS
	- `SQSPollerPolicy`
	- `DyanmoDBCrudPolicy`

### SAM CodeDeploy
- SAM natively uses codedeploy to update lambda functions
- traffic shifting feature
- pre and post traffic hooks to validate deployment (before traffic shift starts and after it ends)
- easy and automated rollback using CW alarms
- `AutoPublishAlias`
	- detects when new code is being deployed
	- creates and publishes an updated version of that function with the latest code
	- points the alias to the updated version of the Lambda func
- `DeploymentPreferece` = Canary, Linear, AllAtOnce
- `Alarms` = can trigger a rollback
- `Hooks` = pre and post traffic shifting lambda funcs to test your deployment

### Local Capabilities
- locally start AWS lambda
	- `sam local start-lambda`
	- starts a local endpoint that emulates AWS lambda
	- automated tests can be run against the endpoint
- locally invoke lambda func
	- `sam local invoke`
	- helpful for testing
	- if the func is calling AWS, make sure the proper credentials are provided
- locally start an API gateway
	- `sam local start-api`
	- starts a local HTTP server thats hosts your functions
	- supports hot-reloading
- generate AWS events for lambda funcs
	- `sam local generate-events`
	- generate sample payload for event sources

### Serverless Application Repository (SAR)
- managed repo for serverless apps
- apps are packaged with SAM
- build and publish apps that can be re-used by organizations
	- can share publicly
	- can share with specific AWS accounts
- prevents duplicate work
- app settings and behavior can be customized using envvars

## AWS CDK
- define your cloud infra using a familiar lang (JS, python, java, .NET)
- contains high level components called constructs
- code is compiled into a CF template
- you can deploy infra and app runtime code together
	- great for Lambda funcs
	- great for docker containers in ECS/EKS

### CDK v SAM
- SAM
	- serverless focused
	- declarative template
	- leverages CF
- CDK
	- ALL AWS services
	- write infra in a programming lang
	- leverages CF
- SAM + CDK
	- can use sam CLI to locally test CDK apps
	- use `cdk synth`

### Constructs
- `CDK Donstruct` is a component that encapsulates everything CDK needs to create the final CF stack
- can represent a single AWS resources (i.e. S3 Bucket) or multiple relates resouces (i.e. SQS queue with compute)
- AWS Construct library:
	- a collection of constructs included in AWS CDK which contains `Constructs` for every AWS resources
	- contains 3 different levels of construct (L1, L2, L3)
- `Construct Hub` - contains addtl `Constructs` from AWS, 3rd parties, and open-source CDK

#### L1
- CFN resources - all resources directly available in CF
- construct names start with `Cfn`
- you must explicitly configure all resource properties

#### L2
- resources with a higher level (intent-based API)
- adds convenient defaults, boilerplate and helper methods that make it simpler to work with the resource

#### L3
- `Patterns` which represent multiple related resources
- helps you complete common tasks n AWS
- i.e.
	- `aws-apigateway.LambdaRestApi` represents an API gateway backed by a lambda function

### Bootstrapping
- necessary resources must be generated per Account/Region before you can deploy CDK apps into an env
- CF Stack called `CDKToolkit` is created and contains:
	- S3 bucket
	- IAM Roles

### Testing
- use CDK assertions module combined with common test frameworks (i.e. Jest or Pytest)
- verify we have specific resources, rules, conditions, params
- two types of tests
	- fine-grained assertions
	- snapshot tests - compare synth'd CF template against a previously stored baseline
- to import a template
	- `Template.fromStack(MyStack)` -> stack build in CDK
	- `Template.fromString(MyString)` -> stack build outside CDK

## Cognito
- give users an identity to interact with web/mobile app

### cognito user pools (for authentication = idenatity vertification):
- sign in functionality for app users
- integrate with API gateway and ALB
	- ALB + Listeners & Rules allows for users to authenticate against CUP and access the target group
- serverless database
- simple login (username + pw)
- MFA
- email and phone number verification
- federated identities through 3rd party IdPs: users from FB, Google, SAML
- Feature: block users if their credentials are compromised eleswhere
- login sends back a JWT
- built in support for snychronous lambda triggers on User Lifecycle hooks
- Cognito has a hosted auth UI you can add to your app which allows for customizations (logo+css)
- `Hosted UI Custom Domain` -> must create the ACM cert in `us-east-1`
	- defined in the `App Integrations` section
- Adapative Authentication
	- block sign ins or req MFA if the login is suspicious
	- risk factor is calcd base don different factors such as device, location, IP
	- users are prompted for a second MFA when risk is detected
	- integration with CW logs for # sign-in attempts, risk score, failed challenges
- CUP Issues JWT tokens (Base64 encoded)
	- Header
	- Payload
	- Signature
- Signature must be verified to ensure the JWT can be trusted (libraries can help w this)
- Payload contains the User info (sub UUID, given_name, email, phone_number, expiry)

#### ALB <> CUP
- offload the work of authenticating users to the ALB
- auth users through
	- IdP, OpenID Connect (OIDC)
	- Cognito User Pools
- must use an HTTPS listener to set `authenticate-oidc` and `authenticate-cognito` rules
- `OnUnauthenticatedRequest` - authenticate (default), deny or allow

### cognito identity pools (federated identity for authorization = access control):
- provide AWS credentials to users so they can access AWS resources directly
- integrate with Cognito User Pools as an IdP
- identity pool can include:
	- public providers (Google, Apple, Amazon)
	- users in an Amazon Cognito user pool
	- OIDC and SAML IdPs
	- developer authenticated identities (custom login server)
	- can also allows for *unauthenticated guest access*
- users can then access AWS services directly or via API gateway
	- IAM policies applied to the credentials are defined in Cognito
	- can be customized based on the `user_id` for fine grained control
- once the token has been obtained from the IdP, it can be exchanged for temporary AWS credentials using STS
- can define a default IAM role for authenticated/guest users
- can partition your users' access using policy variables

## Other Serverless
### Step Functions
- model your workflow as state machines
- written in JSON
- visualize the workflow, its execution and its history
- workflow can be kicked off with SDK, API, CW Event

#### Task State
- work in your state machine
- can invoke 1 AWS service or run 1 activity
- `Choice` - test for a condition to sent to a branch
- `Fail` or `Succeed` - stop execution with failure or success
- `Pass` - pass the input to its output or inject some fixed data
- `Wait` - provide a delay for a fixed interval or until a specified time
- `Map` - dynamically iterate steps
- `Parallel` - begin parallel branches of execution

#### Error Handling
- any state can encounter runtime errors
	- state machine definition issues
	- task failures
	- transient issues
- use `Retry` and `Catch` in the state machine to handle errors instead of inside the app code
- predefined error codes
	- `States.ALL` - matches any error name
	- `States.Timeout` - task ran longer than `TimeoutSeconds` or no heartbeat received
	- `States.TaskFailed`
	- `States.Permissions`
- the state may report its own errors
- conditions in `Catch` and `Retry` are evaluated from top to bottom
- `ResultPath` is used to append the result to the output using the `$` as to object root

#### Wait for Task Token
- allows you to pause setep functions during a task until a task token is returned
- task might wait for other AWS services, human approval, 3rd party integration, etc
- append `.waitForTaskToken` to the `Resource` field to tell the step functions to wait for the task token
- task will pause until it received the token back with a `SendTaskSuccess` or `SendTaskFailure` API call

#### Activity Tasks
- enables you to have task work performed by an `Activity Worker`
- `Activity Worker` apps can be running on EC2, Lambda, Mobile
- workers poll for a task using `GetActivityTask` API
- after worker completes its work, it send a response either `SendTaskSuccess` or `SendTaskFailure`
- to keep the `Task` active:
	- confure how long a task can wait by setting `TimeoutSeconds`
	- periodically send a heartbeat from your Activity worker using `SendTaskHeartBeat` within the time you set in `HeartBeatSeconds`
	- task can wait up to a full year

#### Standard vs Express
- see note in presentation

### AppSync
- managed services that uses GraphQL
- combines data from one or more sources
	- NoSQL data stores, RDMS, HTTP APIs
	- integrates with DynamoDB, Aurora, OpenSearch, etc
	- custom sources with Lambda
- retrieve data in realtime with WebSocket or MQTT on WebSOcket
- for mobile apps: local data access and data sync (replaces cognitosync)

#### Security
- four ways to auth apps to interact w/AppSync
	- API_KEY
	- AWS_IAM
	- OPENID_CONNECT
	- AMAZON_COGNITO_USER_POOLS
- for custom domains and HTTPS, use CF in front of 

## Amplify
- create mobile and web apps (EB for mobile + web)
- authentication (OOTB)
	- leverages Cognito
	- user registration, auth, accountrecovery, etc
	- prebuilt UI components
	- fine grained authorization
- datastore
	- leverages appsync and dynamodb
	- work with local data and automatically sync to the cloud
	- powered by graphql
	- offline and real-time capabilities
	- visual data modeling w/ Amplify studio
- hosting
	- build and host modern web apps
	- CI/CD
	- PR previews
	- custom domains
	- monitoring
	- redirect + custom headers
	- PW protection
- E2E testing and unit testing
	- integrated w/Cypress
	- generate a UI report for your tests

## AWS STS - Security Token Serveice
- allows to grant limited and temporary access to AWS resources (up to 1hr)
- `AssumeRole`: assume roles within your accound or cross account
	- define an IAM role within an account
	- define which principals can access the role
	- use STS tto retrieve cred and impersonate the IAM role
	- temporary creds valid bw 15mins and 1hr
- `AssumeRoleWithSAML`: return credentials for users logged in with SAML
- `AssumeRoleWithWebIdentity`: not recommeded - use cognito identity pools instead
- `GetSessionToken`: from MFA, from a user or AWS account root user
	- access ID
	- secret key
	- session token
	- expiration date
- `GetFederationToken`: temp creds for a federated user
- `GetCallerIdentity`: details about the IAM user or role used in the API call
- `DecodeAuthorizationMessage`: decode error msg when an API call is denied

### IAM policies (simplified)
- if there's an explicit DENY -> DENY
- if there's an ALLOW -> ALLOW
- else -> DENY

### IAM Policies <> S3 Buckets
- what's evaluated is the `UNION` of the two policies

### Dynamic policies with IAM
- special policy var `${aws:username}`

### Inline v Managed
- managed policies
	- maintained and updated by AWS
	- good for power users and admin
- customer managed
	- best practice, re-usable, can be applied to many principals
	- version-controlled + rollback, central change mgmt
- inline
	- 1:1 policy:principal
	- policy is deleted if you delete the IAM principal

### Pass a role to a service
- many services require an IAM role to the service (usually happens during setup)
- the service assumes the role to perform the related actions
- to do this you need `iam:PassRole` permission
	- often comes with `iam:GetRole` to view the role being passed
- roles can only be passed to what their *trust* allows

## AWS Directory Services
- Managed Microsoft AD
	- create your own AD in AWS, manage users locally, supports MFA
	- establish "trust" connections with your on-prem AD
- AD Connector
	- directory gateway (proxy) to redirect to on-prem AD, supports MFA
	- users are managed only on the on-prem AD
- Simple AD
	- AD-compatible managed directory on AWS
	- cannot be joined with on-prem AD

## Security & Encryption

### KMS (Key Mgmt Service)
- AWS manages encryption keys
- fully integrated with IAM for auth
- easy way to control access to your data
- audit KMS key usage using cloudtrail
- integrated into most AWS services
- never store your secrets in plaintext (esp in code)
- AWS owned keys (free): SSE-S3, SSE-SQS, SSE-DDB (default key)
- AWS managed keys (free): (`aws/<SERVICE_NAME>`)
- Customer managed keys ($1/mo) - must be symmetric
	- pay for api calls to KMS as well ($0.03/10000 calls)
- Automatic Key rotation:
	- aws-managed (automatic every year)
	- customer managed (automatic every year)
	- imported KMS (only manual rotatoin possible using alias)
- keys are scoped per region
- KMS encrypt API call has a limit of 4 KB
	- if you want to encrypt > 4 KB, we need to use `Envelope Encryption`
- `Encrypt` and `Decrypt` main API calls
- `GenerateRandom` returns a random byte string

#### Envelope Encryption
- main API for this is `GenerateDataKey` which returns a plaintext data key (DEK) and the encrypted DEK
- then you encrypt the file with the DEK and add the encrypted DEK (these two components make up the final file)
- to decrypt the file, use the Decrypt API to decrypt the DEK, then use the plaintext DEK to decrypt the file
- AWS provides an Encryption SDK
	- data key caching:
		- re-use data keys instead of creating new ones for each encryption
		- helps reduce the API calls (with a security tradeoff)
		- use `LocalCryptoMaterialsCache`
- `GenerateDataKeyWithoutPlaintext` - generate an encrypted DEK


#### Key Types
- symmetric (AES-256)
	- single key used to encrypt and decrypt data
	- aws services that are integrated with KMS use symmetric keys
	- never get access to the KMS key unencrypted
- asymmetric (RSA & ECC key pairs)
	- public (encrypt) and private (decrypt) key pair
	- public key is downloadable, but you cant access the private key unencrypted
	- use case: encryption outside of AWS by users who can call the KMS API

#### Key Policies
- control access to KMS keys
- difference: without a policy you cannot control access (no one can access)
- *default* kms key policy:
	- created if you don't provide a key policy
	- anyone in the account can acces the key
- custom kms key policy:
	- define users, roles that can access the KMS key
	- define who can administer the key
	- useful for cross-account access of your KMS key

#### Request Quotas
- when you exceed the quota, you get a `ThrottlingException`
- for cryptographic ops, all api calls share a quota (including requests made by AWS on your behalf)
- solution:
	- request a quota increase
	- use key caching for DEK

#### S3 Bucket Key for SSE-KMS
- add an S3 bucket key for envelope encryption to decrease number of API calls to KMS by 99%
- leverages Data keys

### Principal Options in IAM Policies
- AWS account
- IAM Roles
- IAM Role Sessions
- IAM Users
- Federated User Sessions
- AWS Services

### CloudHSM
- AWS provisions encryption hardware
- dedicated hardware (HSM = Hardware security module)
- manage your own encryption keys
- HSM device is tamper resistant, FIPS 140-2 Level 3 compliance
- supports both symmetric and asymmetric encryption
- must use the CloudHSM client software
- support for Redshift
- good option to use with SSE-C encryption
- spread across multiple AZs for HA
- integration through KMS
	- define a customer key store in KMS to connect to CloudHSM
	- benefit of auditing w/ cloudtrail
- single tenant

### SSM Parameter Store
- secure storage for configuration and secrets
- option encryption using KMS
- serverless, scalable, durable
- version tracking of configs/secrets
- security through IAM
- notifications w/ Eventbridge
- integrate w/CloudFormation
- allows for hierarchy
- can create a cloudwatch event rule to invoke a lambda every 30 days that will manually change the RDS password and update the value in the param store

#### Parameter Policies (advanced parameters)
- allow or assign a TTL to a param to force updating/deleting sensitive dat
- can assign multiple policies at onces
- events generated in eventbridge to notify based on policies in place

### Secrets Manager (primarily RDS)
- newer service, meant for storing secrets
- capability to force rotation of secrets every `X` days
- automate generation of secrets on rotation (uses lambda)
- integration w/ RDS
- secrets are encrypted w/ KMS
- multi-region secrets
	- repicate secretes across regions
	- secrets manager keeps read replicas in sync with the primary secret
	- ability to promote a read replica secret to a standalone secret

#### CF integration (RDS and Aurora)
- `ManageMasterUserPassword` - creates admin secret implicitly
- can also create the secret, reference the secret in the RDB DB instance, and link the secret as an attchment to the RDS DB

### CW Logs Encryption
- can encrypt CW logs w/KMS keys
- can enable at log group level by associating with a CMK
	- must use API, cannot be done with console

### CodeBuild Security
- use envvars to reference either param store secrets OR secrets manager secrets

### Nitro Enclaves
- process highly sensitive data in an isolated compute environment
	- PII, financial, healthcare
- fully isolated VMs, hardened and highly constrained
	- not a container, not persisent storage, no interactive access, not external networking
- helps reduce the attach surface for sensitive data processing apps
	- cryptographic attestation - only authorized code can be running in your enclave
	- only enclaves can access sensitive data (integrated with KMS)

## Quick overview of other services

### SES
- simple email service
- send emails using SMTP or AWS SDK
- integrates with S3, SNS, Lambda

### OpenSearch (successor to ElasticSearch)
- allows you to search any field, even partial matches
- commonly usd an a complement to another DB
- two modes: managed cluster or serverless cluster
- no native SQL support (can be enabled via a plugin)
- ingestion from KDF, IoT, Lambda and CW logs (subscription filter)
- security w/ Cognito, IAM, KMS encryption, TLS
- comes w/ OpenSearch dashboards

### Athena
- serverless query service to analyze data stored in amazon s3
- uses standard sql language to query the files (built on Presto)
- $5.00 per TB of data scanned
- commonly used w/Amazon Quicksight

#### Performance improvement
- use *columnar data* for cost-savings (less scan)
	- apache parquet or ORC is recommended
	- big perf improvement
	- use glue to convert your data to Parquet or ORC
- compress data for smaller retrievals
- partition data sets in S3 for easy querying on virtual columns
- use larger files (>128 MB) to minimize overhead

#### Federated Query
- allows you to run SQL queries across data stored in relational, non-relational, object and custom data source
- uses data source connectors that run on Lambda to run federated queries (CW logs, DynamoDB, RDS)
- store the results back in S3

### MSK (managed streaming for apache kafka)
- alternative to kinesis
- fully managed Kafka on AWS
	- allows you to create, update, delete clusters
	- MSK creates and manages Kafka broker nodes and Zookeeper nodes for you
	- deploy the MSK cluster in your VPC (up to 3 for HA)
	- automatic recovery from common Kafka failures
	- data is stored on EBS volumes for as long as you require
- MSK serverless
	- run Kafka without managing capacity
	- MSK automatically provisions resources and scales compute and storage

#### Consumers for MSK
- KDA for Flink
- Glue (streaming ETL powered by apache spark streaming)
- Lambda
- Custom apps

### ACM (AWS Certificate Manager)
- easily provision, manage and deploy SSL/TLS certs
- used to provide in-flight encryption for websites
- supports private and public TLS
- automatic TLS renewal
- integrations with: 
	- ELB
	- CF
	- APIs on API gateway

#### Private CA
- create a private CA, including root and subordinaries
- can issue and deploy end-entity X.509 certs
- certs are trusted only by your org
- works with AWS service integrated with APM
- use cases:
	- encrypted TLS communication, cryptographically signing code
	- authenticate users, computers, API endpoints, and IoT devices
	- enterprise customers building a PKI (public key infrastructure)

### Macie
- fully managed data security and data privacy service that uses machine learning and pattern matching to discover and protect sensitive data in AWS
- helps to identify PII and send events to eventbridge

### AppConfig
- configure, validate and deploy dynamic configs independent of your code deployments
- feature flags, app tuning, allow/block listing
- use with apps on EC2, lambda, ECS, EKS
- gradually deploy config changes and rollback if issues occur
- validate config changes before deploy using:
	- json schema (syntax)
	- lambda function (semantic)

## Exam prep
- review lambda + API Gateway auth stuff (https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-control-access-to-api.html)
- CF 
	- keys: https://a.cl.ly/Jrueg8bJ 
	- caching policies
- You must create the Lambda function from the same account as the container registry in Amazon ECR - You can package your Lambda function code and dependencies as a container image, using tools such as the Docker CLI. You can then upload the image to your container registry hosted on Amazon Elastic Container Registry (Amazon ECR). Note that you must create the Lambda function from the same account as the container registry in Amazon ECR.
- IAM with RDS: "IAM database authentication works with MariaDB, MySQL, and PostgreSQL." (https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.html)
- S3 access question (https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.html): query string auth == presigned URL
- IAM: permissions boundary comes up often
- parameter store:  You cannot use a resource-based policy with a parameter in the Parameter Store. (secrets manager for cross-account access)
- EBS stuff:
	- https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/general-purpose.html#gp2-performance
	- https://a.cl.ly/d5uDpnEy
	- The maximum ratio of provisioned IOPS to requested volume size (in GiB) is 50:1. So, for a 200 GiB volume size, max IOPS possible is 200*50 = 10000 IOPS. (https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/provisioned-iops.html)
- what is the correct answer? https://a.cl.ly/DOuKo1by 
- Elasticache config options (cluster mode, replication, etc)
- encryption headers for S3:
	- https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingKMSEncryption.html
- invalide CF cache -- use a HEADER: https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-caching.html#invalidate-method-caching
- dynamodb secondary index types: https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-indexes-general.html
- ports 1024 - 65535 are for user apps
- CodeDeploy lifecycle hooks: https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file-structure-hooks.html#reference-appspec-file-structure-hooks-run-order
- A gateway endpoint is a gateway that you specify as a target for a route in your route table for traffic destined to a supported AWS service. The following AWS services are supported: `Amazon S3`, `DynamoDB`
	- You should note that S3 now supports both gateway endpoints as well as the interface endpoints.
- ELB troubleshooting: https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-troubleshooting.html 
- Macie Sensitive data findings: https://docs.aws.amazon.com/macie/latest/user/findings-types.html
- how to migrate beanstalk env b/w AWS accounts: https://a.cl.ly/jkuRLl7G
- encryption by default for EBS volumes is REGION-SPECIFIC
- SQS has up to 2 week retention
- `Export` for CF to export vars 
- IAM can be used for SSL certs: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_server-certs.html
- IAM access analyzer review: https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html
- ASGs are region-based (can contain instances across AZs)
- reserved instances (https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/reserved-instances-scope.html)  - only Zonal reserved instances provide capacity reservations
- IAM policy evaluation: https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html 
- `ec2 monitor-instances` - enable detailed monitoring for running instances
- The Lambda function invocation is asynchronous - When an asynchronous invocation event exceeds the maximum age or fails all retry attempts, Lambda discards it. Or sends it to dead-letter queue if you have configured one.
- lambda does support websocket
- CodeBuild -> cache dependencies in S3 for faster build times


