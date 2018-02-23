## IAM
* Universal across regions
 * New users have 0 permissions

### Critical terms
* Users: End Users
* Groups: A collection of users under on set of permissions
* Roles: Created and assigned to AWS resources
* Policies - Document that defines one or more permissions

### Roles
* A way to grant permissions to entities you trust.

## AWS Storage and CDN - S3, Glacier and CloudFront
### S3
Simple Storage Service - secure, durable, highly-scalable object storage.
* 0 - 5 TB file size
* Unlimited storage
* Buckets
* Universal namespace, must be unique
* 200 HTTP Status code on successful upload
* Host static sites/content
* *Data consistency model*
  * *Read after write consistency for PUTS of new Object*
  * *Eventual Consistency for overwrite for PUTS and DELETES* (can take some time to propagate)
* 99.99% availability  99.9% SLA
* 11X9’s durability (99.999999999%)
* Designed to survive the loss of 2 facilities concurrently

#### Simple key value store
  * *Key*: file name
  * *Value*: object data
  * *Version ID*
  * *Metadata*
  * Subresources: *Access Control Lists*, Torrent

#### Encryption
* Client Side
* Server Side
  * SSE S3
  * SSE-KMS
  * SSE-C

#### Cross Region Replication
* Only applies to new or modified files
* Use command lines to transfer
* Versioning must be enabled
* Regions must be unique
* No Daisy chaining
* Delete markers are replicated
* Deleting individual versions or delete markers are not replicated

#### S3 Options
* S3 -IA Infrequently Accessed (lower fee)
* S3 Reduced Redundancy Storage (slightly less durability)

#### Charges
  * Storage
  * Requests
  * Storage Management Pricing (tags)
  * Data Transfer Pricing   
  * Transfer Acceleration: Uses CloudFront to move data long distances

#### Versioning
* Allows multi factor authentication deletes
* Once enabled, can only be suspended not removed
= 

#### LifeCycle Management
* Can be used w/ versioning
* Can be applied to current and previous versions
* Actions:
  * Transition to IA Storage (min 128Kb and min 30 days)
  * Archive to Glacier (min 60 days)
  * Permanently delete

#### Security and Encryption
Buckets are PRIVATE by default
Use bucket policies and access control lists to control access
S3 buckets can be configured to log access
*Types of encryption
  * *In Transit: SSL/TLS*
  * At Rest
    * *Server Side*:
      * S3 Managed Keys - *SSE-S3*
      * AWS Key Management Service, Manage Keys - *SSE-KMS*
      * Server Side Encryption with Customer Provided Keys - *SSE-C*
    * *Client Side Encryption*

#### Transfer Acceleration
Utilizes edge location to accelerate uploads to S3. 
Custom URL

#### Tips
Read the S3 FAQ

### Glacier 
(Archival), $0.01 per gb, retrieval = 3-5 hours, 90 minimum storage time

### CloudFront
Content Delivery Network

#### Edge Location
Content spread out to edge locations 
* Not just read only
* Objects are cached for TTL (Time To Live) 24hr default
* You can clear the cache, but it costs
* Separate from AWS Regions/AZs

#### Origin
This is where the files are stored that are to be distributed by the CDN
* S3, EC2, ELB, Route53…

#### Distribution
The collection of edge locations
* Web Distribution: Websites
* RTMP: Serving media content

### Storage Gateway
On premise data storage to AWS cloud (S3)
VM on premise server
* File Gateway (NFS) 
  * Files in S3
  * Nothing on premise
* Volumes Gateway (iSCSI) SQL server
  * Virtual hardiest (EBS snapshots)
  * Stored volumes
    * Asynchronous backups to S3 in an EBS Snapshot (1GB - 16TB)
    * Entire dataset on premise
  * Cached volumes
    * Complete data on S3 (1GB -32TB)
    * Recently read data on premise
    * Do not need as much storage on premise
* Tape Gateway (VTL)
  * Virtual tape
  * NetBackup, Backup Exec, Veeam…

### Snowball
* Snowball
  * Petabyte scale data transport solution 80TB
  * Trusted Platform Module
  * 256-bit encryption
  * Tamper resistant enclosures
  * 1/5th the cost of data transfer
* Snowball Edge
  * 100TB data transfer
  * Has compute capacity
  * Can perform Lambda functions
  * Datacenter in a box
* Snowmobile
  * Petabyte or Exabytes of data
  * Shipping container 100PB

## EC2 - Elastic Compute Cloud

Allows developers the ability to pay for only the capacity you use/need.

### Options

* On Demand - allow you to pay a fixed rate by the hour (or by the second)
* Reserved 1 to 3 year contract
  * Standard
  * Convertible
  * Scheduled
* Spot - set bid a price for instance capacity
* Dedicated Hosts - Physical EC2 server dedicated to your use.

### EC2 Instance Types

| Family | Speciality | Use case |
| --- | --- | --- |
| D2 | Dense storage | Fileservers/Data Warehousing/Hadoop |
| R4 | Memory Optimized | Memory Intesive Apps/DBs |
| M4 | General Purpose | Application servers |
| C4 | Compute optimized | CPU intesive Apps/DBs |
| G2 | Graphics intesive | Video encoding/ 3D application streaming
| I2 | High speed storage | NoSQL DBs, data warehousing
| F1 | Field programmable gate array | Hardware accelleration for your code |
| T2 | Lowest Cose, General purpose | Web servers, small DBs
| P2 | Graphics/General purpose GPU | Machine leaerning, mining |
| X1 | Memory optimized | SAP HANA, Apache Spark |

#### DR MC GIFT PX

* **D**: Density
* **R**: RAM
* **M**: Main general purpose
* **C**: Compute
* **G**: Graphics
* **I**: IOPS (Databases) 
* **F**: FPGA
* **T**: Tiny, general purpose
* **P**: Pictures, moving pictures, general GPU
* **X**: X-treme memory optimized

### EBS - Elastic Block Store

Block based storage disks that you can attach to your EC2 instances.
Automatically replicated for you mitigating hardware failure. (In same AZ)

* General Purpose SSD (GP2)
  * Balance of price and performance
  * 3 IOPS per GB up to 10,000 IOPS w/ bursts up to 3000 IOPS
* Provisioned IOPS SSD (IO1)
  * Designed for I/O intensive applications/databases
  * Only needed if you require over 10,000 IOPS
  * Up to 20,000 IOPS
* Throughput Optimized HDD (ST1)
  * Big data, data warehousing, log processing
  * Cannot be a boot volume
* Cold HDD (SC1)
  * Lowest cost for infrequently accessed data
  * File server
  * Cannot be a boot volume
* Magnetic (Standard)
  * Lowest storage cost per GB of all EBS volume types that are bootable

### Need to Know

* The difference between:
  * On demand
  * Spot
  * Reserved
  * Dedicated Hosts
* Sport instances:
  * If you terminate, you pay for the hour.
  * If AWS terminates the spot instance, you get the hour it was terminated for
    free.
* EBS types and applications
  * You can not mount 1 EBS volume to multiple EC2 instances, use EFS instead
    for shared storage.

### EC2 Lab Tips

* Use tags to reduce costs!
