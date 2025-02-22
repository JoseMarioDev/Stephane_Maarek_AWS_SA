## 223. AWS Security - Section introduction

- introduction
- focal part of the exam

#

## 224. Encryption 101

- encryption in flight(SSL)
  - data is encrypted before sending and decrypted after receiving
  - SL certs helps w/encryption(HTTPS)
  - encryption in flight ensures no MITM attack
- Server side encryption at rest
  - data is encrypted after being rec'd by server
  - dta is decrypted before being sent
  - stored in encrypted form thanks to a key(usually a data key)
  - enryption/decrpytion keys must be managed somewhere and the server must have access to it
- client side encryption
  - data is encrypted by the client and never decrypted by the server
  - data will be decrypted by a receiving client
  - the server should not be able to decrypt the data
  - could leverage envelope encryption

#

## 225. KMS Overview

- Key Mgmt Service
  - easy to control access to your data, AWS manages keys for us
  - fully integrated w/IAM for authorization
  - seamlessly integrated into: EBS, S3, Redshift, RDS, SSM
  - you can also use CLI/SDK
- KMS - Customer Master Key(CMK) Types
  - Symmetric(AES-256 keys)
    - single encryption key used for encrypt/decrypt
    - AWS services that are integrated with KMS use Symmetric CMKs
    - necessary for envelope encryption
    - you never get access to the key unencrypted(must call KMS API to use)
  - Asymmetric (RSA & ECC Key pairs)
    - public(encrypt) and Private(decrypt) key pair
    - used for encrypt/decrypt or sign/verify operations
    - public key is downloadable, but you cant access the private key unencrypted
    - use case: encryption outside of AWS by users who cant call the KMS API
- able to fully manage keys and policies:
  - create
  - rotation policies
  - disable
  - enable
- able to audit key usage using cloudtrail
- three types of CMK
  - AWS managed
  - user keys created in KMS
  - user keys imported
- KMS 101
  - anytime you need to share sensitive information use KMS
  - database passwords
  - creds to external services
  - private key of SSL certs
- value in KMS is that the CMK used to encrypt data can never be retrieved by the user, and the CMK can be rotated for extra security
- KMS can only help in encryption up to 4kb of data per call
- KMS keys are bound to a specific region
- key policies
  - similar to S3 bucket policies
  - you cannot control access without them
  - default KMS key policies
    - created if you dont specify a specific policy
    - complete access to root user

#

## 226. KMS Hands On w/CLI

- KMS service in console
- three tabs on left - AWS managed keys, Customer managed keys, custom key stores
- create customer managed keys
- choose between symmetric or asymmetric
- key material origin
- create alias
- key administrator
- config policy on who can use. root user(everyone) by default can use
- create key
- click on key for settings
- using cli, how to encrypt files.
- file with commands is in resources folder

#

## 227. SSM Parameter Store Overview

- securely store configuration and secrets
- optional seamless encryption using KMS
- serverless, scalable, durable easy SDK
- version tracking of configurations/secrets
- configuration mgmt using path and IAM
- notifications with CloudWatch events
- integration with cloudformation
- uses a folder structure like hierarchy
- 2 tiers - standard and advanced
- can set parameter policies
  - assign a TTL
  - deleting sensitive data such as passwords

#

## 228. SSM Parameter Store Hands On(CLI)

- systems manager console
- parameter store on left pane
- create parameter
- /myapp/dev/db-url for example

#

## 229. SSM Parameter Store Hands On(AWS Lambda)

- create a function
- name and choose runtime
- permissions
- create function
- body of function is using boto31 sdk to call SSM and get the parameters back
- test function
- check IAMS permissions
- run your function

#

## 230. AWS Secrets Manager - Overview

- newer service, meant for storing secrets
- capability to force rotation of secrets every X days
- automate generation of secrets on rotation
- integrate w/RDS
- secrets are encrypted using KMS
- mostly meant for RDS integration

#

## 231. AWS Secrets Manager - Hands On

- secrets manager in console
- store a new secret
- select secret type
- select "other type of secrets"
- specify key/value pair
- select encryption key
- name your secret
- configure automatic rotation
- choose Lambda function invoked to handle rotation
- store

#

## 232. CloudHSM

- KMS -> AWS manages the software for encryption
- CloudHSM -> AWS provisions encryption hardware
- dedicated hardware(HSM = hardware security module)
- you manage your own encryption keys entirely(not AWS)
- HSM device is tamper resistent
- CloudHSM clusters are spread across multiple AZ(HA) - you must set up
- supports both symmetric/asymmetric keys
- no free tier
- must use CloudHSM software

#

## 233. Shield - DDoS Protection

- 2 flavors
  - standard: free services enabled by default for every AWS customer
  - provides protection from DDoS attacks
  - advanced: DDoS mitigation service(3k per month cost)
  - protests against more sophisticated attacks on EC2, ELB, CF, AWS GA, R53
  - 24/7 access to AWS DDoS response team
  - protects against higher fees due to DDoS attacks

#

## 234. Web Application Firewall (WAF)

- protects against layer 7 exploits
- HTTP layer
- deploy on ALB, API GW, CloudFront
- define web ACL - access control list
- rules on IP addresses, HTTP headers, HTTP body, URI strings
- common attacks projection: SQL injection, Cross-site scripting(XSS)
- size limits on queries, geo-match to block countries
- rate based rules to count occurrences of events - for DDoS attacks
- AWS firewall manager
  - manage all rules in all accts
- ![arch for DDoS protection using shield and WAF](img/20-DDoS-arch.png)

#

## 235. WAF & Shield - Hands On

- WAF and shield console
- create web ACL
- name
- resource type to associate with
  - choose cloudfront distributions
  - add resources to protect
  - add rules/groups
  - create web ACL
- Shield - select from left pane
  - click on activate to enable advanced
- AWS WAF - left pane
  - config firewall rules

#

## 236. Shared Responsibility Model

- aws responsibilities
  - security of cloud
    - protecting infrastructure that runs AWS services
    - manages services like S3, DDB, RDS
- customer responsibility
  - mgmt of the guest OS, sec patches, updates, firewall network config, IAM

#

## Quiz 19: Security & Encryption Quiz

-

#
