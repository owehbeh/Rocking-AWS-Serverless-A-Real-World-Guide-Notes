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

### Provisioned Concurrency

- Pre-initialized execution environments
- No cold start / throttling due to super fast scaling needs
- AWS keeps assigned capacity _"Warm"_

### Service Quotas

You can request a concurrency limit increase for the Lambda functions

- Go to **Service Quotas**
- Select **AWS Lambda**
- Select **Concurrent executions**
- Click on **Requrest quota increase**

## 🟦 External Dependencies

### Use AWS Toolkit on VSCode to install external dependencies

1. Download **AWS Toolkit**
2. **Login** using IAM credentials
3. Create a **workspace** to your Lambda Functions
4. Open AWS Toolkit **explorer** and **select** your Lambda Function
5. Right click > **download**
6. Open **terminal** under the downloaded local Lambda Function **directory**
7. **Install** missing dependencies
8. **Upload** the directory of the updated Lambda Function

### Use AWS Cloud9

- Same steps to follow
- No need to install any extensions/login
- Code in the cloud inside your browser

## 🟦 Lambda Container Image Support

- AWS provides base images
- Runtime interface client manages interaction between Lambda service and function code
- Can create custom images

### Why use it?

    - Utilize existing dockerized apps
    - Create images with what is needed (faster startup time)
    - Local testing with runtime interface emulator
    - 10GB package size instead of 50MB Zip deployment

### 👣 Steps to follow

1. Write your app code
2. Dockerize your app
    - Select base image - amazon/aws-lambda-nodejs:12
    - Copy app files
    - Run npm install
    - `CMD["app.lambdaHandler"]` is a required command and won't work out of the box if you use a custom container image not provided by AWS. Instead you will have to create a [Lambda compatible container image](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html#images-create-from-alt)
3. Push the docker image to ECR repository

## 🟦 Lambda Layers

### What is it?

- Allows you to share code
- Can be anything:
  - Dependencies
  - Training data
  - Configuration files
  - Business logic code

### Why?

- Built in support for secure sharing by ecosystem
- Promotes seperation of responsibilities
- Helps devs iterate faster on writing business logic - DRY
- Layer gets loaded with function code, so **no** additional execution **latency**

### Limits

- 250MB total size limit (total layers unzipped)
- Up to 5 layers/function

## 🟦 Lambda EFS Integration

- Pay for what you use
- Shared accross concurrent executions of Lambda functions
- Can be used with Provisioned Concurrency
  - Execute any initialization code
  - Any libraries or packages consumed on EFS at this point are downloaded

### Use cases

- Process **large files**
- Import large **ML Models**
- Import large **code libraries**
- **Share files** accross other AWS services (e.g. EC2, EKS etc.)
- With this integration, Lambda is **not stateless** anymore

### 👣 Steps to follow

- **Create** EFS instance
- **Attach** Lambda function **role policy**
  - `AWSLambdaVPCAccessExecutionRole`
  - `AmazonElasticFileSystemClientReadWriteAccess`
- **Attach** Lambda function to **VPC** where EFS was created
  - Attach same security group of EFS
- **Add** File System to Lambda Function
  - Select the created EFS instance
- To access the **shared EFS** from **Cloud9**
  - Attach the same security group on the EC2 that is on the EFC and Lambda Function

## 🟦 RDS Proxy

### What is it?

- Sits **between** Lambda and RDS
- Maintaines a **connection pool**
- Allows apps to **share a pool** of database connections

### Why use it?

- Lambda can **exhaust the database** when large number of connections open and close, leading to throttling behavior and sometimes error
- A lot of **overhead** to manage this inside the **Lambda side**
- Can directly point to new instance in case of failover, which leads to 66% **reduced failover time**
- Can specify how much **% of the connection** uses of the connection pool, allowing other services a share of the connection pool no matter how big Lambda function scales

### Supported Databases

- Amazon Aurora
- RDS MySQL
- RDS PostgreSQL
- RDS MariaDB
- RDS Oracle
- RDS SQL Server

### 👣 Steps to follow

1. IAM Role for RDS Proxy
    - Go to IAM Roles
    - Create new Role
    - Select RDS
    - RDS - Add Role to Database
    - Attach policy - SecretsManagerReadWrite
2. Create RDS MySQL
3. Create RDS Proxy
    - Choose database from step 2
    - Connection pool maximum %
4. IAM Role for Lambda
    - Attach new permission `AmazonEC2FullAccess`
4. Proper VPC and Security Groups for Lambda to Proxy to RDS
5. Peroper external dependencies for Lambda to access RDS MySQL
6. Add env variables to allow access to RDS proxy endpoints from Lambda

## 🟦 SNS - Simple Notification Service

### What is it?

- A Pub-Sub Messaging Service

### Why use it?

- Automatically **Scales**
- Messages **Secured** using AWS KMS keys
- **Fan Out Architecture** through Message filtering
  - Specific message to subscriber A
  - Specific message to subscriber B
  - Specific message to subscriber C

### Queing

- **Standard**
  - Order not guaranteed
  - Message may be delivered more than once
  - Nearly unlimited messages/sec
  - Cheaper
- **FIFO**
  - Order strictly preserved
  - Dedup configuration
    - Avoids duplicate message delivery
  - 300 message/sec
  - Batching supported upto 3000 message/sec
  - A bit more expensive

### SNS Message Filtering

- When adding a subscriber to the SNS topic
- Add a subscription filter policy

## 🟦 EventBridge - event bus

### What is an event bus?

- A mechanis that allows different components to communicate with each other without knowing about each other

### What is EventBridge?

- A serverless service that allows SaaS applications to trigger a workflow/event in AWS

### Why use it?

- **Connect** data from SaaS apps
- Easily build **event-driven architecture**
- Write less code
- Reduce operational overhead

### Custom Event Bus

- **Default** event bus can only have **AWS** services as **source**
- If another app like Zendek running on EC2 wants to send an event, you must use a custom event bus
- Message **content based filtering** is only possible throught a **Custom Bus**

## 🟦 EventBridge | SQS | SNS

Metric | EventBridge | SQS | SNS
------------------|---------|----------|---------
**Scaling** |  Autoscale, default 400 PutEvents and 750 target incovation requests/second and use Lambda/function concurrency to control downstream | Autoscale but use Lambda trigger Batch size and Lambda/function concurrency to control downstream (infinite scaling) | Autoscale but use Lambda/function concurrency to control downstream (infinite scaling)
**Conditional Message Processing** |  - **Event filtering** can route messages to targets based on message.<br>- Can **transform events** before sending to target.<br>- **Schema registery** to describe message and can generate code bindings.<br>- Can **automatically** discover schemas from messages. | **Can't** decide consumer based on message, instead use SNS message filtering with SQS to achieve this | Can invoke different subscriber based on values on message **metadata** using SNS message filtering
**Message Replay** |  Can be **archived** based on rules.<br> Can be **replayed** later. | **Gone** once delivered to subscribers.<br>**No** replay. | **Gone** once delivered to subscribers.<br>**No** replay.
**Message Ordering** |  Order **not** maintained | **FIFO queue** maintains order | **FIFO** maintains order
**Encryption** | **At rest** using KMS<br>HIPAA compliant | **At rest** using KMS<br>HIPAA compliant | **At rest** using KMS<br>HIPAA compliant
**Durability** | Multiple AZ | Multiple AZ | Multiple AZ
**Persistence** | No formal persistence model<br>Retry up to 24 hours<br> Good practice is to create a **catch-all rule**, any message that doesn't satisfy any rule, we can investigate later on | Messages stored for 4 days<br> Min 60 sec max 14 days | No formal persistence model<br>Retry up to 23 days
**Consumption** |  [Lambda/EC2/Step Functions/API Gateway/a lot more..]<br>Use event patterns set on rules to control which events are subscribed-to by different rules | [Lambda/AWS SDK]<br>Remember you can call message delete from within code or let the service handle it via successful Lambda function execution to avoid duplicate executions | [Lambda/SQS/email/mobile push/SMS/HTTP]<br>Use Message Filtering to control which messages go to which subscriber<br>Use Message delivery status to track failures
**Retry/Failure Handling** |  Exponantial back off till 24 hours<br> Use with SQS for **DLQ(dead-later queue)** | Remain in queue until deleted<br>Cannot be seen by other consumers during **visibility timeout**<br> **DLQ(dead-later queue)** can be used | Exponential back off till 23 days min 2 times 1 second apart max38 times 20 minutes apart

## 🟦 EventBridge Pipes

Event-driven architecture

- **Source** - point-to-point integrations between event producers and consumers
- **Filter** - Define an event pattern to filter the events that are sent through the pipe
- **Enrich** - Define an event pattern to filter the events that are sent through the pipe
- **Target** - Send your event to an AWS service, an event bus, or an API destination

# 4️⃣ Step Functions

### An orchestration service that lets you connect your Lambda functions into serverless workflows, called state machines

## 🟪 Why use it?

- Lots of coding for flow control in Lambda
- If the flow changed, Lambda needs to be changed an retested
- Not easy to change the flow

## 🟪 Solution

- Step Function takes care of coordination and flow control
- Create/Change flow in **Visual Console**
- Lambdas become cleaner

## 🟪 Key Components

- States (Whole flow is called state machine)
  - Elements inside state machine
- Tasks
  - All work inside state machine is done by _tasks_
  - A task can be an activity or a Lambda function
- Transitions
  - Tells the State what to do next or where to begin

```JSON
"HelloWorld":{
    "Type":"Task",
    "Resource":"arn:aws:lambda..............",
    "Next":"AfterHelloWorldState",
    "Comment":"Run the HelloWorld Lambda Function"
}
```

## 🟪 Workflow Types

### Once selected for a State Machine, it cannot be changed

- **Standard**
  - Long running
  - Slightly higher latency workflows
  - Cheaper
- **Express**
  - High-volume
  - Low latency
  - Lower duration (Express!) workflow

## 🟪 Nested Workflows Tricky Note

- Default behavior is the Workflow1 will not wait for Workflow2 response to proceed
- To achieve Synchronous flow you have to call the Workflow2 synchronously using the "`.sync`" expression in the "`Resource`" e.g.

```JSON
"HelloWorld":{
    "Type":"Task",
    "Resource":"arn:aws:states:::states:startExecution.sync",
}
```

### Wait For Callback, similar to the callback pattern, allows for a Token to be sent back to Step Function and won't proceed until then

- Can run for a year
- Can specify using **HeartBeatSeconds**

# 5️⃣ Serverless Loggin & Monitoring

## 🟫 Lambda

### Cloudwatch is integrated by default

- Cloudwatch logs
- X-Ray traces

## 🟫 API Gateway

### Can be enabled to a specific stage

## 🟫 CloudTrail vs CloudWatch

 CloudTrail | CloudWatch
----------|---------
 **Infrastructure Logging** | **Application Logging**
 Creation/Deletion of S3 bucket | API request/response payload
 Creation/Deletion of VPC | Logging from Lambda code
 Creation/Deletion of Security Group | Logs for execution of API

- CloudTrail logs can be sent to CloudWatch logs
- All the logs can be fed to an analytic system for actionable insights

## 🟫 CloudWatch Log Insights

- Fully managed log query tool
- Queries massive amount of logs in seconds
- Produces visualizatons
- Lots of pre-built queries

## 🟫 X-Ray

Analyze and Debug Your Applications

### API Gateway

- Have to enable for a specific stage
- Under Logs/Tracing
- Enable X-Ray Tracing

### Lambda

- Go to debugging and error handling
- Enable active tracing
- For extra debugging, you have to import the X-Ray SDK inside Lambda Function and specify in code where and when to log

# 6️⃣ AWS CLI

## 🔲 Cloud9

A cloud IDE for writing, running, and debugging code

## 🔲 Cloudshell

A browser-based shell that gives you command-line access to your AWS resources in the selected AWS region. AWS CloudShell comes pre-installed with popular tools for resource management and creation.

# 7️⃣ Lambda with Cloud9

## 🔳 Why use it?

- Access from anywhere, code in the cloud
- Debug Lambda functions easily

# 8️⃣ Security - API Gateway & Beyond

## 🟩 API Gateway

### Usage Plans

- **Meter** API usage
- Enforce a **throttling** and quota limit on each API key
- Throttling limits define the maximum number of **requests per second** available to each key
- Quota limits define the **number of requests** each API key is allowed to make **over a period**

### API Keys

- Attached to a plan in Usage Plans
- Enabled on the **Method execution** level under Resources

### AWS IAM Authorization

- Generate
  - Access key ID
  - Secret access key
- Allows granular access to specific API methods, each IAM role has access to specific resources

### Resource Policy

- Specify whome to allow and what to allow them to do on the api

```JSON
{
    "Version":"2012-10-17",
    "Statement":[
        "Effect":"Allow",// ALLOW
        "Principal":"*",//WHAT?
        "Action":"execute-api:Invoke",
        "Resource":"arn:aws:execute-api:region:account-id:api-id/",
        "Condition":{
            "IpAddress":{
                "aws:SourceIp":["10.20.30.40"]//WHO?
            }
        }
    ]
}
```

## 🟩 Cognito Users Pools

1. Create **User Pool** - Pool ID
2. Create **App Client** - Client ID
3. Assign User Pool as **Authorizer** for API Gateway **Method**
4. User **sign up** to the pool using Pool ID and Client ID
5. User exchange credentials and **receives Token ID**
6. User **calls API** Method with Token
7. AWS **Validates** Token
8. Backend **Lambda called**

### 🟩 Cognito Federated Identity Pool

1. Create Identity Pool
2. Assign IAM Role for Identity Pool
3. User logs in
4. This also allows **granular access** since you can choose to require this IAM role on the API method level

## 🟩 Cognito Hosted UI

- Provides an OAuth 2.0 compliant authorization server
- Includes default implementation of end user flows such as registration and authentication
- Can customize user flows, such as the addition of Multi Factor Authentication (MFA), by changing user pool configuration

## 🟩 AWS Secrets Manager

- Store, rotate, monitor, and control access to secrets such as database credentials, API keys, and OAuth tokens
- Enable secret rotation using built-in integration for MySQL, PostgreSQL, and Amazon Aurora on Amazon RDS
- Provides sample code to consume with popular coding languages
- Good security option in order not to manage environment key/value pairs, allowing for automated periodical rotation, in other words, more security

## 🟩 Lambda Resource Policies

### Why use it?

- Prevent IAM cross Lambda access
- By default, invoking Lambda function from API **in same AWS account** automatically changes the resource policy to allow the invocation

### Solution

- Remove permissions of resource (API Gateway Sid)
- Through AWS CLI `aws lambda remove-permission --function-name <lambda-name-or-arn> --statement-id <statement-id-of-the-permission>`

## 🟩 AWS Inspector - Lambda Integration

### An automated vulnerability management service that continually scans workloads for software vulnerabilities and unintended network exposure

# 9️⃣ Storage For Serverless

## 🟡 SQL vs NoSQL

SQL | NoSQL
---------|----------
 Predefined Schema | Schemaless
 Structured Data | Structured and Unstructured Data
 Good fit for joins and complex queries | Generally, not good fit for complex multi table queries
 Emphasizes on ACID properties<br>-Atomicity<br>-Consistency<br>-Isolation<br>-Durability | Follows the Brewers CAP theorem<br>-Consistency<br>-Availability<br>-Partition tolerance
 Oracle<br>DB2<br>MS-SQL<br>AWS RDS | AWS DynamoDB<br>MongoDB<br>Cassandra

## 🟡 DynamoDB Globat Tables
### What is it?
Fully managed, multi-Region, and multi-active/multi-master NoSQL database
### Why use it?
- Massively scaled applications with globally dispersed users

## 🟡 DynamoDB Consistent Read
- Strongly Consistent Reads
  - Returns most up-to-date data
  - Can be achieved by setting `ConsistentRead` parameter to true
  - Have less throughput (KB/sec) than eventually consistent reads
- Eventually Consistent Reads
  - Might return old data
  - Returns latest data after a short time
  - Default read behavior

Read Capacity Unit | Write Capacity Unit
---------|----------
 -**One strongly** consister read per second<br> -**Two eventually** consistent read per second | **One** write per second
For item up to **4KB** in size<br>Larger itmes require more capacity unit | For item upto **1KB**<br>Larger items require more capacity units

### How to use strongly consistent reads?
- Pass the option `'ConsistentRead':true` to the get_item function

## 🟡 DynamoDB On Demand
### What is it?
- Usually you have to set minimum provisioned capacity unit and maximum provisioned capacity unit
- Scaling without capacity planning
- Pay-per-request pricing
### Why use it?
- If app traffic is difficult to predict
- Workload has large spikes of short duration
- Possible to switch back and forth between provisioned and on-demand mode
- Indexes created on the table inhereit same scalability and billing model
#### When not to use it?
- If app has super high traffic constantly, it is better to provision min/max capacity units

## 🟡 DynamoDB Streams
### What is it?
- Capture data modification events in near-real-time and in order
  - Add Events
  - Update Events
  - Delete Events
### Why use it?
- Invoke Lambda Functions
- Sync with another database
- Replicate another database

## 🟡 DynamoDB Encryption
Default CMK is created by AWS, in case of a scenario where you need to specify what IAM user/role has access to the KMS key, you need to
1. Create a Lambda Function that encrypt/decrypt data
2. Assign a role to that function
3. Pass the data to this Lambda Function before writing-encrypt/reading-decrypt 

# 1️⃣1️⃣ DevOps for Serverless
## 🔴 AWS CodeCommit
### What is it?
Create secure repositories for sharing code in the cloud

## 🔴 AWS CodeBuild
### What is it?
- Build and test code with elastic scaling
- Pay only for the build time
### How it works?
- Pull source code
- Reads Buildspec File to know what to do
  - Must be in root directory
  - Must be named buildspec.yml
  - Must be YAML formatted
- Run the commands in Buildspec
## 🔴 AWS CodeDeploy
### What is it?
Automate code deployments to maintain application uptime
### Why use it?
- Automatically scales
- Rolling updates
- Blue/Green Deployments
- Stop & Roll Back
- Switch Lambda Traffic
  - All at once
  - Canary
### How it works?
- AppSpec file
  - What tasks to do?
  - In what order?
- Process
  1. files
      - source
        - destination
      - source
        - destination
  1. BeforeInstall
      - location (scripts location)
  2. AfterInstall
      - location (scripts location)
        - timeout
  3. ApplicationStart
      - location (scripts location)
        - timeout
  4. ValidateService
      - location (scripts location)
        - timeout
        - runas (IAM user)
## 🔴 AWS CodePipeline
### What is it?
Visualize and automate the different stages of software release process.
### How it works?
You can add stages as you wish, but mainly stages would look like:
- Source
  - CodeCommit
  - GitHub
  - BitBucket
  - Etc..
- Build
  - Run commands to build
  - Run commands to test
  - Save build files on S3 buckets
- Deploy
  - AWS ElasticBeanstalk
  - AWS EC2
  - AWS CloudFormation
  - AWS CodeDeploy
## 🔴 AWS CodeStar
### What is it?
Quickly develop, build, and deploy applications 
- 
### Why use it?
- Templates for Amazon EC2, AWS Lambda, and AWS Elastic Beanstalk
- CodePipeline created
  - Each step is visualized
- Integrated with Jira and Github issues
- Assign team member and roles from the service

# 1️⃣2️⃣ AWS SAM (Serverless Application Model)
## 🟢 What is it?
AWS SAM is an open-source framework that provides developer tools for building and running serverless applications on AWS. The AWS SAM toolkit consists of two primary parts:
1. AWS SAM template specification
2. AWS SAM command line interface (AWS SAM CLI)
## 🟢 Why use it?
- Abstracts lines of CloudFormation
- Build serverless applications faster
- Less maintenance
## 🟢 How it works?
- Create YAML file
```YAML
AWSTemplateFormatVersion: '2010-09-09'
Transform: 'AWS::Serverless-2016-10-31'
Description: Any description goes here
Resources:
  HelloWorldAPI:
    Type: 'AWS::Serverless::Api'
    Properties:
      StageName: Dev
      DefinitionUrl: ./APIfiles/swagger-lambda.yaml
  samfunction:
    Type: 'AWS::Serverless::Function'
    Properties:
      FunctionName: HelloWorld
      Handler: index.handler
      Runtime: nodejs17.x
      CodeUri: ./folder/hellow_world.js
      Description: vegeterian pizza 🍕
      Events
        GetApi:
          Type: Api
          Properties:
            RestApiId: !Ref "HelloWorldAPI"
            Path: /
            Method: GET
```
- Build YAML file
  - `sam package --template-file <YAML file> --s3-bucket <S3 Bucket> --output-template-file <Output YAML file>`
- Deploy
  - `sam deploy --template-file <Output YAML file> --stack-name <Stack name> --capabilities <IAM Role>`
- Check CloudFormation for progress

# 1️⃣3️⃣ Serverless Frameworks
- AWS 
  - AWS SAM
  - AWS CodeStar
  - AWS Amplify
  - AWS Chalice
- Third-Party
  - Serverless Framework
  - SPARTA
  - Claudia.js
  - Zappa
  - APEX

# 1️⃣4️⃣ Serverless Vs Container

Serverless | Container
---------|---------
 Run code without provisioning or managing servers | Unit of software that packages up code and all its dependencies
 Easier to onboard | Complete control of environment
 Focus on solving business problems | Rich ecosystem
 Shines at event driven architectures | Faster migration to cloud with other softwares
 Suited when traffic is unpredictable | Suited when traffic is predictable
 Automatically scales with traffic | Scales horizontally spinning a new EC2 for example while only 50% of this EC2 is utilized so you pay for 50% idle
Cheaper when traffic is unpredictable | Cheaper when traffic is predictable

## 🟠 AWS Fargate
### What is it?
- Serverless version of Container
### Why use it?
- No need to create a cluster or determeine EC2 size
- Scales on-demand
- Pay for what you use

## 🟠 ECS vs EKS vs Fargate
### What is a container orchestrator?
A utility that is designed to easily manage complex containerization deployments across multiple container hosts and locations from one central location
### Why use it?
Removes the burden of
- Deployment of Containers
- Redundancy and availability of Containers
- Sacling up or down of Containers
- Load balancing
- Health monitoring
- Service discovery
- And more...
### Container Orchestrators
- Docker Swarm
- Apacke Mesos
- Cattle
- Nomad
- Empire
- **AWS ECS**
- Kubernetes
  - **AWS EKS**
- **AWS Fargate**


ECS | EKS | Fargate
---------|----------|---------
 Container Orchestration  by AWS | Managed Kubernetes (Open Source) platform by AWS | Containers on-demand
 Require creating cluster | Require creating cluster | No cluster required
 Control Plane 0$ | Control Plane 144$ | Pay for tasks based on CPU and Memory

# 1️⃣5️⃣ Serverless Architectures & Advanced Optimization Techniques
## 🟣 Optimizing Lambda 
- Use pre-handler logic strategically
  - Scopes
    - Global Scope:
      - Cold Start: Executed on first execution
      - Warm Start: Does not executes
    - Lambda Handler: Re-runs on each execution
  - Example
    - Move database connections to global scope
- Concise function logic
  - Minimize dependencies
  - Import required parts of packages
    - `from http import request` 
  - Use Lambda Layers for duplicated logic
- Lazy load libraries
  - Under certain condition inside the Lambda Handler `import special-lib`
- Share secrets based on application scope
  - Single function: env vars
  - Multi Functions / Shared Environments: Secrets Manager
- Use Lambda Functions to **TRANSFORM** not ~~TRANSPORT~~
  - ~~SNS > Lambda > S3~~ >>> SNS > S3
- Push orchestration up to Step Functions
  -  `Submit_Job` > `Get_Job_Status` > `Check_Job_Status` > `Wait_30_Seconds` > `Get_Job_Status` > `Check_Job_Status`
  - **Don't** `Wait_30_Seconds` inside a Lambda Function
- Read only what you need
  - Properly indexed databases
  - Use Views instead of compute
  - Use Amazon S3 Select instead of reading a directory and manually looking for a file
## 🟣 Optimizing Lambda Execution Environment
### Use AWS X-Ray to trace
- Cold Start Time
- Lambda Initialization Time
- Execution Time
### Third-Party Tools
- Datadog
- Epsagon
- NodeSource
- IOPipe
- Thundra
### Memory Optimization
Sometimes increased memory leads to significant performance increase and a slightly cost increase, totally worth it.
- [Lambda Power Tuning](https://github.com/alexcasalboni/aws-lambda-power-tuning): Uses AWS Step Functions to run multiple concurrent versions of a Lambda function at different memory allocations and measure the performance
  - Lowest Costs
  - Fastest Execution Times
- CloudWatch Insights
  - Lambda Queries > Determine the amount of overprovisioned memory
- Do you need to put Lambda Function in an Amazon VPC?
  - RDS in VPC: Yes
  - Restrict outbound access to internet: Yes
  - No need? Then don't
- Lambda Execution Models
  - Synchronous
    1. Amazon API Gateway
    2. GET|POST
    3. AWS Lambda Function
  - WebSocket
    1. Amazon API Gateway
    2. Bidirectional ⬆️⬇️
    3. AWS Lambda Function
  - Asynchronous
    1. Amazon SNS|SQS
    2. Amazon S3 (Events)
    3. AWS Lambda Function
  - Poll-based
    1. Amazon Kinesis
    2. Changes
    3. AWS Lambda Service
        - Function
- Do you Sync everywhere?
  - Seperate Sync/Async componenst
    - GET Sync
    - POST Async
  - Sending data to another system
    - Do you need to insert in realtime?
    - Is near realtime okay?
    - Do you need response/callback in same call?
    - API vs topic/queue/stream
- Do you need to expose Lambda to APIs?
  - Only getting called from internal AWS services, then no
### [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/?wa-lens-whitepapers.sort-by=item.additionalFields.sortDate&wa-lens-whitepapers.sort-order=desc&wa-guidance-whitepapers.sort-by=item.additionalFields.sortDate&wa-guidance-whitepapers.sort-order=desc)
- Operational excellence
  - Running monitoring system
  - Improving processes and procedures
  - Automating changes
  - Responding to events
- Security
  - How to encrypt data in-transit at-rest
- Reliability
  - Prevent/Recover from failure
- Performance Efficiency
  - Using the right resources and using them efficiently
- Cost Optimization
  - Avoid un-needed cost
- Sustainability
  - Minimizing the environmental impacts of running cloud workloads
### General Design Principals
  - Speedy, simple, singular
  - Share nothing
  - Orchestrate app with state machines
  - Design for failures and duplicates
  - Use events to trigger transactions
### Design Components
- Compute Layer
  - Lambda
  - API Gateway
  - Step Functions
- Data Layer
  - DynamoDB
  - Aurora Serverless
  - S3
- Messagig and Streaming Layer
  - Kinesis
  - SNS
- User Management and Identity Layer
  - Cognito
- Systems Monitoring and Deployment
  - CloudWatch
  - X-Ray
- Edge Layer
  - CloudFront
### Design Patterns
- RESTful Microservices
  1. Consumer
  2. Amazon API Gateway
  3. AWS Lambda
  4. Amazon DynamoDB | Aurora Serverless
- Stream Processing
  1. Kinesis
  2. AWS Lambda
  3. DynamoDB
- Periodic Job
  1. Schedule - CloudWatch events
  2. Trigger - AWS Lambda
  3. External API
  4. DynamoDB
- Web Application
  1. Web Client
  2. Static Content
      - Amazon CloudFront
      - Amazon S3
  3. User directory / authentication
      - Amazon Cognito
  4. Serverless Backend
      - Amazon API Gateway
      - AWS Lambda
      - Amazon DynamoDB
### When not to use Lambda
- Compare cost with EC2. Price vaires based on
  - Traffic
  - Memory
  - Execution time
- Long Cold Starts
  - ~~If Lambda is in VPC, Cold Starts attach ENI (Elastic Network Interface)~~
    - Fixed by AWS!
    - ENIs are now attached during function **creation** and **NOT** ~~execution~~ 
- Super heavy computing exceeding Lambda Memory/Time limit
    