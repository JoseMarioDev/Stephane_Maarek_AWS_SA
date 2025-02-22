# 155. Introduction to Messaging

- when we deploy apps, they will need to communicate with each other
- 2 patterns of app communication
  - sync: app to app
  - async/event based: app to queue to app
- sync between apps can be a problem if there are spikes of traffic
  - ex: video encoding service with spike of service
- better to decouple your apps
  - using SQS: queue model
  - using SNS: pub/sub model
  - using Kinesis: real time streaming model
- these services can scale independently from our app

#

## 156. Amazon SQS - Standard Queues Overview

- client sends a messaage to the queue is called a producer
  - could one/more than one producer
- receivers are called consumers
  - they poll the service to check for messages
  - consumer process message, delete from queue
- fully managed service used to decouple applications
- attributes:
  - unlimited throughput
  - unlimited num of messages in queue
  - retention: default 4 days; max is 14 days
  - low latency <10ms
  - limitation of 256kb
  - can have duplicate messages - at least once delivery
  - can have out of order messages - best effort ordering
- SQS Producers
  - produced to SQS using the SDK(sendmessage API)
  - message is persisted in SQS until a consumer deletes it
- SQS Consumers
  - applications running on EC2 instances, servers, or AWS Lambda
  - consumers poll(check for msgs) the SQS queue. can receive up to 10 messages at a time
  - process messages and do whatever coding needs to be done
  - consumer deletes msg from SQS using the deletemessage API
- SQS multiple EC2 instance consumers
  - scale this up
  - consumers receive and process msgs in parallel
  - that why there is "at lesat once" delivery
  - best effort msging order
  - consumers delete msgs after processing
  - we can scale consumers horizontally to improve throughput of processing
- SQS w/ASG
  - setup a cloudwatch metric
    - ex: approximateNumberOfMessages reaches certain length
    - triggers an Cloudwatch Alarm
    - notifies the ASG to add more instances
- SQS to decouple application tiers
  - ex: app that renders videos
  - decouple into 2 apps
    - first app receives the videos sends a msg to SQS to process the video
    - back end app polls SQS, gets msg, processes the videos; stores in S3 bucket when done
    - this way, you can use diff instances based on need for the FE and BE
    - decouples app so there is no bottlenecks
- SQS security
  - encryption
    - inflight using HTTPS API
    - at rest encryption using KMS keys
    - client can also do client side encryption if it wants
  - access control
    - IAM policies to regulate access to the SQS API
  - SQS access policies
    - similar to S3 bucket policies
    - useful for cross account access to SQS queues
    - useful for allowing other services(SNS, S3...) write to SQS queue

#

## 157. SQS - Standard Queue Hands On

- SQS console
- create message
- choose standard vs FIFO
- configuration
- retention period
- access policy
- basic/advanced
- encryption
- create queue
- top right hand side, send and rec msgs
-

#

## 158. SQS - Message Visibility Timeout

- after a msg if polled by a consumer, it becomes invisible to other consumers
- default the "message visibility timeout" is 30 seconds
- that means the msg has 30 seconds to be processed
- after the msg visibility timeout is over, the message is visible in SQS
- if a msg is not processed within the visibility timeout, it will be processed twice
- a consumer could call the ChangeMessageVisibility API to get more time
- can change default setting in configuration of the queue

#

## 159. SQS - Dead Letter Queues

- if consumer fails to process msg within the visibility timeout, the msg goes back into queue
- we can set a threshold of how many times a msg can go back to queue
- after the MaximumReceives threshold is exceeded, the msg goes into a dead letter queue(DLQ)
- useful for debugging
- make sure to process the msgs in the DLQ before they expire
- good idea to check retention time
- config of original queue, enable DLQ and specify the arn of the DLQ, configure settings

#

## 160. SQS - Delay Queues

- delay a msg so consumers dont see immediately. up to 15 mins
- default is 0 seconds - msgs avail right away
- can set a default at queue level
- can override the default on send using the DelaySeconds parameter

#

## 161. SQS - FIFO queues

- first in first out
- ordering guarantee
- has limited throughput
- 300 msg/per second without batching
- exactly once send capabilities by removing dupes
- messages processed in order by the consumer
- select FIFO when creating queues
- must name queue with .fifo extension
  - example: myqueue.fifo

#

## 162. SQS + Auto Scaling Group

- scale num of instances based on num of msgs in queue
- create a CloudWatch custom view metric
  - que length divided by num of instances
  - create alarm when it reaches a certain number
  - step scaling policy when it's above/below the metric
- this is how we create decoupling

#

## 163. Amazon SNS - Overview

- what if you want to send one msg to many receivers?
- pub/sub model
- pub sends msg to queue, establishes a topic. the subscribers sub to topic and get the msg
- the event producer only sends msg to one SNS topic
- as many event receivers(subscribers) as we want listening to the SNS topic notifications
- each sub to the topic will get all the msgs(new feat to filter msgs)
- up to 10 million subscriptions per topic
- 100k topics limit
- subscribers can be:
  - SQS
  - HTTP/HTTPS
  - lambda
  - email
  - SMS msgs
  - mobile notifications
- integrates with lots of AWS services
  - cloudwatch for alarms
  - ASG notifications
  - S3 buckets
  - CloudFormation
- how to publish
  - use the topic publish SDK
  - create a subscription
  - publish to topic
  - direct publish (for mobile apps SDK)
- SNS - Security
  - inflight using HTTPS API
  - atrest using KMS keys
  - client side if client want to perform itself
- access controls
  - IAM policies to regulate access
- SNS access policies
  - cross accts
  - allowing other services to use

#

## 164. SNS - Hands On

- SNS console
- create topic
- details
  - name
  - encryption
  - access policy
  - retry policy
- create topic
- create subscription
  - choose protocol

#

## 165. SNS and SQS - Fan Out Pattern

- idea is to push once into SNS, receive in all SQS queues that are subscribers
  - so, SQS queues are subscribers, SNS is the publisher
- fully decoupled no data loss
- allows for data persistence, delayed processing, retries of work
- can abb more subs over time
- sqs queue needs to have access policy that allows SNS to write to it
- SNS cannot send msg to SQS FIFO queues - AWS limitation

#

## 166. Kinesis Data Streams Overview

- managed alternative to Apache Kafka
- big data streaming tool
- great for logs, metrics, IoT,clickstreams
- great for "real-time" data
- great for streaming processing frameworks(Spark, NiFi, etc)
- data is automatically replicated to 3 AZs
- associated with big data real time
- 3 products
  - kinesis streams: low latency streaming ingest at scale
  - kinesis analytics: perform realtime analytics on streams using SQL
  - kinesis firehose: load streams into S3, Redshift, ElasticSearch
- kinesis streams:
  - streams are divided into ordered shards/partitions
  - producers -> shards -> consumers
  - data retention default is 1 day, can go up to 7 days
  - ability to reprocess/replay data
  - multiple apps can consume the same stream
  - real time processing with scale of throughput
  - once data is inserted in kinesis, can't be deleted(immutable)
- one stream is made of many diff shards
- 1 mb/s or 1000 messages/s at write per shard
- 2 mb/s at read per shard
- billing is per shard provisioned, can have as many shards as you want
- batching available or per msg calls
- shards can evolve over time
- records are ordered per shard
- put records
  - use the putRecord API + partition key that gets hashed to determine shard ID
  - same key goes to the same partition
  - msgs get a sequence number
  - choose partition key that is highly distributed
- use batching with putRecords to reduce costs and increase throughput
- provisionedthroughputexceeded if we go over the limits
- can use CLI, AWS SDK or 3rd party libraries
- API exceptions
  - when sending more data than shard can handle
  - make sure you dont have hotshard (partition key is bad and too much data is going to that partition)
  - solution: retry with backoff, increase shards, change partition key
- consumers
  - can be CLI, SDK
  - kinesis client library
- security
  - uses IAM policy
  - encrypt inflight using HTTPS endpts
  - at rest using KMS
  - can encrypt clientside, but much harder
  - VPC endpoints available for Kinesis to access within VPC

#

## 167. Kinesis Data Streams Hands On

- kinesis console
- get started
- choose resource to create
- create data stream
- name
- shards
- create stream
- click on stream to see details
- encryption
- data retention period
- shard level metrics
- monitoring
- tags
- in cli aws kinesis help to see details
-

#

## 168. Kinesis Firehose & Kinesis Data Analytics

- full managed service, no administration, automatic scaling, serverless
- used to load data into Redshift, Amazon S3, ElasticSearch, Splunk
- near real time
- 60 seconds latency minimum for non full batches
- supports many data formats, conversions,
- pay for amt of data going through firehose
- kinesis data streams vs firehose
  - steams
    - going to write custom code(producer/consumer)
    - realtime
    - must manage scaling - shard splitting/merging
    - data storage
  - firehose
    - full managed, send to S3, splunk, redshift, elasticsearch
    - serverless data transformations with Lambda
    - near real time
    - automated scaling
    - no data storage
  - Kinesis Analytics
    - perform real time analytics using SQL
    - auto scaling
    - continuos
    - pay for actual consumption

#

## 169. Data Ordering for Kinesis vs SQS FIFO

- ordering data into Kinesis
  - should be using a diff partition key for each piece of data
    - truck in the example
  - hashing the partition key with the msg will determine what shard to put data into
    - truck example
- ordering data into SQS
  - for SQS FIFO, if you dont use a group ID, messages are consumed in the order sent, with only one consumer
  - you want to scale the number of consumers, but you want msgs to be grouped when they are related to one another
  - they you use a group ID

#

## 170. SQS vs SNS vs Kinesis

- SQS
  - consumer pull data
  - data is deleted after being consumed
  - no order guarantee - unless using FIFO
  - can have as many consumers as you want
- SNS
  - pub/sub model
  - up to 10 million subs
  - data is persistent
  - integrate with SQS for fan out arch
- kinesis
  - consumers pull data
  - possibility to replay data
  - real time big data analytics
  - ordering at shard level

#

## 171. Amazon MQ

- when migrating current onprem queue services to cloud, instead of re-engineering the app to use SQS, SNS, we can use amazon active MQ
- Amazon MQ - managed Apache Active MQ
- Amazon MQ - doesnt scale as much as SQS/SNS
- runs on a dedicated machine, can run in HA with failover
- has both queue feature and topic feature

#

## Quiz 13: Messaging and Integration Quiz

-

#
