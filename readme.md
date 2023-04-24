# 1️⃣ Getting Started

## 🟨Serverless Checklist
- No servers to provision or manage
- Automatically scales with usage
- Never pay for idle
- Availability and fault tolerance built-in

## 🟨IAM - Identity and Access Management
### Group
- A group of **IAM Users**
- Can attach a list of **Policies**
### Policy
- Cannot be directly attached to **AWS Services** directly
- Can be attached to **Roles** & **Groups**
### User
- An entity to represent a person or application and interact with **AWS**
### Role 
- An **AWS** identity with permission **Policies**

## 🟨AWS Serverless Services
- Compute
    - AWS Lambda
    - AWS Fargate
- Storage
    - Aurora Serverless
    - DynamoDB
    - Amazon S3
- Integration + Analytics
    - Amazon API Gateway
    - Amazon SQS
    - AWS Step Functions
    - AWS Glue
    - Amazon SNS
    - AWS AppSync

## 🟨Quiz 1
- Using **Lambda Alias** you can **split traffic** between different Lambda versions
- Lambda is **not the only** serverless service in **AWS**
- Lambda **environment variable** is a **Key-Value** pair that you can dynamically pass to function
- Basic Lambda **compute settings** are 
    - Memory selection
    - Timeout selection
- Lambda uses **CloudWatch** to monitor **Metrics** and **Logs**

# 2️⃣ AWS API Gateway

## 🟥What is an API?
A set of clearly defined method of **communication** between various components

## 🟥What is an API Gateway
An API management tool that serves as a single point of entry into a system, sitting between the application user and a collection of backend services
- Create, configure, and host an API
- Handles authentication & authorization
- Handles tracing, caching, and throttling
- \+ much more...

## 🟥Components
- Resources
    - Methods
        - Client → Request → Integration → Lambda ↴
        - Client ← Method Response ← Integration Response ↲
- Stages - for deployment
- Authorizers - control access
- Gateway Responses
- Usage Plan
    - Meter API usage
    - Enforce a throttling and quota limit on each API key
- API Keys

## 🟥AWS Cross Account Call
- Must specify the **ARN** of the **Lambda function** in **account b**
- Must add a policy to allow the **API Gateway ARN** in **account a** 

## 🟥Lambda Versioning
Publishing a version increments the number of the latest version and you can access the latest version by `$LATEST`

## 🟥Lambda Alias
- Can point to more than one version
- Can divert percentage of the traffic to different versions - ex: 60% of traffic to v1 and 40% to v2
- Point to **Alias** from an **API Gateway**
- Can achieve **Blue/Green deployment**

### Consuming Lambda Alias
- From the API Gateway **Integration Request**, you can change the Lambda Function to point to the Alias by following this structure: `functionName:${stageVariables.aliasName}`
- From the API Gateway **Stages**, you can set the **Stage Variables** 
### _Why use the Stage Variables instead of hardcoding the Alias name?_
- Because this way, each deployment **Stage** can point to a different **Alias**

## 🟥Canary
**API Gateway** has this feature out of the box.
It aims to reduce the risk of interoducing a new software version to production by slowly rolling out changes.

## 🟥Endpoint Types
- Edge Optimized
    - Reduce client latency from **anywhere** on the internet
    - Deployed to the **CloudFront** network
- Regional
    - Deployed to the current **selected region**
- Private
    - Expose APIs only inside **VPC**

## 🟥Caching
- Caching can be enabled for better performance and cost savings
- Cache can be configured at the stage or method level
- Cache can be based on query string, headers or both
- Cache can be invalidated manually or automatically based on a time-to-live (TTL) value

## 🟥API Types
- REST - works with
    - Lambda
    - HTTP
    - AWS Services
- HTTP API - works with
    - HTTP API
    - HTTP backends
- WebSocket API - works with
    - Lambda
    - HTTP
    - AWS Services

# 3️⃣ Lambda Advanced Concepts

## 🟦 Scaling
