## 13. IAM Introduction

- IAM is a global service
- three main types:
  - Users
  - Groups
  - Roles
- Users are put into Groups
- Groups are assigned permissions
- Roles are assigned to machines and given permissions
- permissions are JSON object
- should use the "least privilege principle" . give user the minimum amount of access to get their job done
- can use IAM Federation for big enterprises. Users can use their current Active Directory to log into AWS
- Brain Dump
  - one IAM user per person
  - one IAM role per application
  - never use the root account, create once and don't use again

#

## 14. IAM Hands-on

- creating user acct
- create admin group, apply admin policy to group
- add created user to group
- change alias to company name
- log in as new user
- voila!

#

## 15. changes to the EC2 console

- nothing here, doc showing new changes to look of EC2 console

#

## 16. EC2 introduction

- capable of:
  - renting virtual machines (EC2)
  - storing data on virtual drives (EBS)
  - Distributing load across machines (ELB)
  - Scaling the services using an auto-scaling group (ASG)

#

## 17. SSH Overview

- summary table, which OSs support SSH or putty.
- can also use instance connect

#

## 18. How to SSH using Linux or Mac

- what is SSH?
  - allows you to control a remote machine, using the command line
  - ssh into instance
  - chmod 0400 the .pem file
  - run ssh -i nameofkeyfile.pem ec2-user@ipaddress

#

## 19. How to SSH using Windows

- can use putty

#

## 20 How to SSH using Windows 10

- how to use powershell/or putty if you dont have SSH

#

## 21 SSH Troubleshooting

- list of common problems and how to fix

#

## 22 SSH EC2 Instance Connect

- browser based SSH connection
- needs to recent with AMI

#

## 23. Introduction to Security Groups

- fundamental part of network security in AWS
- control how traffic is allowed in/out of our EC2 machines
- fundamental skill to learn to troubleshoot networking issues
- allow, inbound, outbound ports
- edit security groups
  - must allow ports
  - implicit deny?

#

## 24. Security Groups Deep Dive

- SGs act as a "firewall" on EC2 instances
- they regulate:
  - access to ports
  - authorized IP ranges - v4 and v6
  - control of inbound network traffic( outside world -> instance)
  - control of outbound network traffic( instance -> outside world)
- good to know:
  - SGs can be attached to multiple instances
  - locked down to a region/VPC combo
  - live outside of the EC2 instance - if traffic is blocked, the EC2 instance won't see it
  - good to maintain one separate SG for SSH access (Maarek opinion)
  - timeout errors - SG issue
  - connection refused errors - application error
  - **all inbound traffic is blocked by default**
  - **all outbound traffic is authorized by default**

#

## 25. Private vs Public vs Elastic IP

- networking has 2 types of IP addresses: v4 and v6
- v4 allows for 3.7 billion diff addresses in public space
- public IP
  - machine can identified on the internet
  - must be unique across the web
  - can be geolocated easily
- Private IP
  - machine can only be identified on private network
  - IP must be unique across the private network
  - but 2 diff private networks can have the same IP
  - machines connect to WWW via internet gateway
  - only a specified range of IPs can be used as private
- Elastic IP
  - when you start/stop an EC2 instance, it can change it's public IP address
  - if you need a fixed public IP for your instance, you can use an Elastic IP
  - an elastic IP is a public v4 IP address as long as you don't delete it
  - you can remap from one instance to another
  - you can only have 5 Elastic IPs in your acct
  - you can ask AWS to increase that amt
  - overall, try to avoid using Elastic IPs
    - reflect poor architectural decisions
    - instead use a random public IP and register a DNS name to it
    - or use a load balancer and dont use public IP
- by default EC2 instances come with:
  - a private IP for internal AWS network
  - public IP for WWW
  - when we SSH into instance:
    - cant use private IP because not on same network
    - only use public
  - if your machine is stopped, then re-started:
  - **the public ip can change**
  #

## 26. Private vs Public vs Elastic IP Hands On

- stop/start instances will assign you a new IP address
- Elastic IPs will not change if you stop/start an instance

#

## 27. Install Apache on EC2

- install Apache web server to display a web page
- create an index.html page
- ran sudo to give full privileges
- yum updates to machine
- installed httpd -> yum install -y httpd.x86_64
- started the httpd service and remain across reboots
- ran curl on localhost:80
- change the SG to allow port 80
- updated the index.html at /var/www/html

#

## 28. EC2 User Data

- possible to bootstrap EC2 instances using EC2 User Data script
- bootstrapping means launching commands when an instance starts
- script runs once at the start of instance
- use EC2 user data to automate tasks such as:
  - installing updates
  - installing software
  - download any common files from internet
- script runs with root user

- there's a file w/the commands in the course resources

#

## 29. EC2 Instances Launch Types

- on demand instances: short workload, predictable pricing (what we've been using for this course)
- reserved(minimum 1 year):
  - reserved instances: long workloads
  - convertible reserved instances: long workloads w/flexible instances(can change the instance hardware type)
  - scheduled reserved instances: ex -> every Thu between 3-6pm
- Spot instances: short workloads, for cheap, can lose instances (less reliable)
- dedicated instances: no other customers will share your hardware
- dedicated hosts: book an entire physical server, control instance placement

1. on-demand

- pay for what you use
- highest cost, no upfront payment
- no long term commitment
- recommended for short term and un-interrupted workloads, where you can't predict how the application will behave

2. Reserved Instances

- up to 75% discount
- pay upfront for what you use with long term commitment
- reservation period can be 1 or 3 years
- reserve a specific instance type
- recommended for steady state usage applications (think database)

3. Convertible Reserved Instance

- can change the EC2 instance type
- up to 54% off

4. Scheduled Reserved Instances

- launch within time window you reserve
- when you require a fraction of day/week/month

5. Spot Instances

- can get up to 90% discount
- can be lost at any point of time
- cost efficient
- useful for workloads that are resilient to failure
- recommended for batch jobs, data analysis, image processing, etc
- not great for critical jobs or databases
- great combo: reserved instances for baseline + on demand and spot for peaks

6. Dedicated Hosts

- physical dedicated EC2 server for your use
- full control of EC2 instance placement
- visibility into underlying sockets/physical cores of the hardware
- allocated for your account for 3 year period
- more expensive
- used for: software that has a complicated licensing model - BYOL
- or for regulatory/compliance needs

7. Dedicated Instances

- instances running on hardware dedicated to you
- may share hardware with other instances in the same account
- no control over instance placement - can move hardware after start/stop

#

## 30. Spot Instances & Spot Fleet

- can get a discount of up to 90% compared to ondemand
- you define max spot price and get the instance while current spot price < max
- the hourly spot price varies based on offer and capacity
- if current spot price > your max price, you can choose to stop/terminate your instance within a 2 minute grace period
- option: spot block -> you spot an instance for a specified period of time (1-6 hours) without interruptions
- rare situations, your instance can be reclaimed
- used for batch jobs, data analysis, workloads resilient to failures
- not great for critical jobs or databases
- terminate spot instances by first canceling the spot request, then terminating the instance
- otherwise, terminating the instance first will just generate a new spot request (loop slide in the course)
- spot fleets -> set of spot instances + optionally on demand instances
- the spot fleet will try to meet the target capacity with price constraints:
  - define multiple launch pools: instance type, OS, AZ
  - can have multiple launch pools, so the fleet can choose
  - spot fleet stops launching instances when reaching capacity or max cost
- strategies:
  - lowestPrice: fom the pool with the lost price - cost optimized, short workload
  - diversified: distributed across all pools - great for availability, long workloads
  - capacityOptimized: pool with the optimal capacity for the number of instances
- Spot Fleet allows us to automatically request spot instances with the lowest price

#

## 31. EC2 Instances Launch Types Hands On

- browsing the AWS console, EC2 instances to view the diff instance types

#

## 32. EC2 Instance Types Deep Dive

- main types to know

  - R: applications that need lot of RAM - in-memory caches
  - C: applications that need good CPU - compute/database
  - M: applications that are balanced - think "medium" - general/web app
  - I: applications that need good local I/O (instance storage) - databases
  - G: applications that need a GPU - video rendering/machine learning

  - T2/T3: burstable instances(up to a capacity)
  - T2/T3-unlimited: unlimited burst
  - [website to compare instances](www.ec2instances.info)

- Burstable instances(T2/T3)
  - when machine needs to process something unexpected(spike) it can burst and CPU can be good
  - they use up "credits"
  - when out of credits, performance can lower
  - use CloudWatch to see usage/credit balance

#

## 33. EC2 AMIs

- Amazon Machine Image
  - ex of base images
  - ubuntu, Fedora, RedHat, Windows, etc
- images can be customized at runtime using EC2 User Data
- but what if we could create our own AMI?
  - can be built for Linux or Windows machines
- advantages of a custom AMI?
  - pre-installed packages that are needed
  - faster boot times(no need for EC2 User Data)
  - machine comes configured w/monitoring/enterprise software
  - security concerns - control over the machines in a network
  - control of maintenance and updates of AMIs over time
  - Active Directory Integration out of the box
  - installing your app ahead of time(faster deploys when auto-scaling)
  - using someone else's AMI that is optimized for running an app, DB, etc
- **AMI are built for a specific reason**
- can be found on Amazon Marketplace
- Your AMI take space and live in S3
- private by default and locked for you acct/region
- you can make them public and share them with other AWS accounts or sell on marketplace
- billed for the S3 space

#

## 34. EC2 AMI Hands On

- creating an AMI from our first instance
- installed apache,
- created index.html -> "hello world"

#

## 35. Cross Account AMI Copy

- to copy an AMI from a billing product(a Windows AMI or an AMI from Marketplace), first launch EC2 instance from that AMI, then create an AMI from your instance

#

## 36. EC2 Placement Groups

- sometimes you want to control placement of your EC2 instances
- strategy can be used to define placement groups
- when you create a placement group, your specify one of 3 strategies:

1. cluster - low latency instances in a single AZ
2. spread - spreads instances across underlying hardware(max 7 instances per group per AZ) - critical applications
3. partition - spreads instances across many diff partitions that rely on diff racks within an AZ. scales to 100s of instances per group. useful for Hadoop, Cassandra, Kafka

- 1. cluster

  - instances are in the same rack, same AZ.
  - pros: low latency, 10Gbps bandwidth between instances
  - cons: if rack fails, all instances fail
  - use cases: big data job that needs to complete fast
  - application that needs extremely low latency, high network throughput

- 2. spread
  - oppo of cluster
  - minimizes failure risk
  - all EC2 instances are located on diff hardware
  - pros: spans multi AZs, reduces risk of failure
  - cons: limited to 7 instances per AZ per placement group
  - use cases: applications that need to maximize HA
  - critical apps where each instance must be isolated from failure from each other
- 3. partition
  - partitions = think of as racks
  - up to 7 partitions per AZ
  - up to 100s of EC2 instances per rack
  - the instances in the partition do not share racks with the instances in other partitions
  - a partition failure can affect many EC2 instances, but wont' affect other partitions
  - EC2 instances get access to the partition information as metadata
  - use cases: HDFS, Cassandra, Kafka

#

## 37. Elastic Network Interfaces(ENI) with Hands On

- logical component in a VPC that represents a virtual network card
- ENIs attributes:
  - primary private v4 address, one more more secondary v4 addresses
  - one elastic v4 IP per private IP v4
  - one public IPv4
  - one or more sec groups
  - a MAC address
- you can create ENIs independently and attach them on the fly on EC2 instances for failover
- bound to a specific AZ

#

## 38. ENI - Extra Reading

- [link for extra reading](https://aws.amazon.com/blogs/aws/new-elastic-network-interfaces-in-the-virtual-private-cloud/)

#

## 39. EC2 Hibernate

- we know we can stop/term instances:
  - stop: the data on EBS is is kept until next start
  - terminate: root vol is set to be destroyed, any EBS volumes that are secondary are kept in tact
- on startup:
  - first start: OS boots, and EC2 user data script is ran
  - following starts: OS boots
  - then application starts, cache warms up, that can take lots of time
- what hibernate does:
  - in-memory (RAM) state is preserved
  - instance boot is much faster- OS is not stopped/restarted
  - under the hood, RAM state is written to a file in the root EBS volume
  - root EBS volume must be encrypted
  - use cases: long running processing
  - saving RAM state
  - services that take time to initialize
- good to know:
  - supported families: C3, C4, C5, M3, M4, M5, R3, R4, R5
  - instance RAM size must be less than 150GB
  - instance size not supported for bare metal instances
  - AMI - Linux 2, linux AMI, Ubuntu, and Windows
  - root volume must be EBS, encrypted, not instance store, and large
  - available for ondemand and reserved instances
  - an instance can be hibernated no more than 60 days

#

## 40. EC2 Hibernate - Hands On

- demo using an m5 large instance
- t2.micro not supported

#

## 41. EC2 for Solution Architect

- EC2 instances are billed by the second, t2.micro is free tier
- on Linux/Mac we use SSH.  Windows < 10 use Putty
- SSH is on port 22.  lock down the sec group to your IP/trusted IPs
- timeout issues -> sec group issues
- permission issues -> run chmod 0400
- sec groups can reference other sec groups instead of IP ranges  
- know diff between private,public, elastic IPs
- you can customize an instance at boot time using EC2 User Data
- know 4 EC2 launch types
  - ondemand
  - reserved
  - spot
  - dedicated host
- know basic types of instances
  - R,C,M,I,G,T2/T3
- you can create AMIs to preinstall software on your EC2 -> faster boots
- AMIs can be copied across regions and accounts
- EC2 can be placed in groups
  - cluster
  - spread
  - partition
#

## 42. Quiz 2: EC2 Final Quiz
