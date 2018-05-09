# Cloud Computing Final Review

## DynamoDB

> **DynamoDB** - fully managed NoSQL database service

- fully managed - managed by aws
- NoSQL - does not require strict schemas like traditional SQL databases

### DynamoDB vs S3

S3 should be used in cases of large files that are typically infrequently accessed whereas DynamoDB should store smaller files sometimes pointers to S3 files

### Features

- define table throughput capacity for each table
- On demand backup
- encryption at rest
- delete expired items automatically
- automatically replicated across availability zones
- global tables allow syncing across AWS regions

### Concepts

**Table** - collection of items

**Item** - collection of attributes (item = row and attribute = column)

**atribute**

- Data types
  - scalar type (string, number, binary, boolean, null)
  - Document types (list and map)
  - Set Types (multiple scalar values)
- when creating table data type of primary key attribute needs to be specified
- primary key attribute can be: String, number binary

### Primary Key

2 types:

- partition key
  - composed of one attribute
  - used with a hash function to determine where to tutor an item
  - only one time can have a specific value
- Partition and Sort Key
  - composite primary key
  - first attribute is the partition key
  - second is the sort key
  - all items with the same partition key are stored together
  - if have same partition key then need to have different sort key

### API

- control plane - to work with tables and streams
  - `CreateTable`
  - `DescribeTable`
  - `ListTables`
  - `UpdateTable`
  - `DeleteTable`
- data plane - to work with data items in the table
  - `PutItem`
  - `BatchWriteItem`
  - `GetItem`
  - `BatchGetItem`
  - `Query` - partition key needs to be specified
  - `Scan` - retrive all item sin the specified table or index
    - `FilterExpression` can be used to limit items return and is applied only after table is scanned
    - each scan returns a page of keys
    - `LastEvaluatedKey` should be passed to subsequent scans as the `ExclusiveStartKey`
  - `UpdateItem`
  - `DeleteItem`
  - `BatchWriteItem`

- streams - work with streams
  - capture modification events on tables
  - each event is recorded
  - before and after state captured (modified, added, deleted all cause capture)

### Condition Expressions

- used to determine which item should be modified
- if condition expression evaluates to true then the operations succeeds
- : indicates an expression attribute value acting as a placeholder for values not known till runtime (expression attribute value)

### Read Consistency

- tables in one region are completely separate from table sin other regions (meaning you can have tables with the same name in different regions)
- table data is replicated across availability zones within a region
- when data is written all data is updated eventually
  - window of inconsistency is 1 second or less
- DynamoDB supports eventually consistent reads and strongly consistent reads
  - controlled by the `ConsistentRead` parameter
    - false - eventually consistent behavior
    - true -strongly consistent behavior but not available if there are network partitions

### Throughput capacity for reads and writes

- 1 strongly consistent read per second for data up to 4KB
- 2 eventually consistent reads per second for data up to 4KB
- 1 write per second for data up to 1KB
- max item size - 400KB (don't use dynamo to store large data items)
- Dynamo will throttle read/write requests if the request rat goes beyond the declared read/write capacity throughput

### Throughput capacity management

- Auto scaling
  - define a range for read and write capacity units
  - define target utilization percentage within the range
  - request throttling not be used
- Provisioned throughput
  - max amount of capacity that application can consume fro table
  - request throttling done if exceeds capacity
- Reserved capacity
  - can be purchased to avoid request throttling

### Good Practice

- **tables** - for partition keys choose something with a large variety of values 
- **items** -use one to many table rather than large set attributes

### Secondary Indexes

copy of table with only selected attributes

- allows querying of the table through a secondary alternative key
- can be done globally and locally with partition and sort key

### Authentication and Access Control

- dynamo only supports IAM policies (no resource based policies)
- Ownership
  - root account - own
  - create IAM user and grant permission to table - you still own it
  - create IAM role with permission to create table - you still own the resource

### IAM Condition Operators

`IfExists` - if key is present process as specified if not then evaluate condition element as false

## IaaS

**IaaS** - infrastructure as a service

- Infrastructure
  - compute
    - vms (ec2)
    - containers (docker)
  - storage
    - object storage (s3)
    - non relational stores (dynamodb)
    - relational stores (amazon rds, google cloud SQL)
  - networking
    - virtual private cloud
    - load ballancers
- as a service
  - multi-tenant - same service is able to support multiple users
  - pay as you go
  - exposing apis (typically with dev tools)
  - web service running on private or public data centers accessible over HTTP

### Examples:

- EC2 is IaaS for VMs
- S3 - Object storage
- DynamoDB - non relational datastores
- RDS - for relational databases

## IaC

**IaC** - Infrastructure as code i.e. a web service that works with infrastructure defined in declarative format (the code)

- declarative inputs (express what is required not how to do it): SQL
- high level abstractions: resource, resource groups

#### Advantages

- easier to understand what is being provisioned than when you have to allocate resources in code
- robust single and multi-resource provisioning
- sharable infrastructure definition
- repeatable creation of infrastructure
- easy to rollback to previous version of resource
- can track resource templates in a  version control system

### AWS CloudFormation

- abstracts the resources away through the use of a template that defines a stack of resources to provision
- template describing infrastructure. i.e. a declarative definition (YAML, JSON formate)
  - give template to CloudFormation web service and it provisions all infrastructure resources

#### Declared

- resources & properties
- input parameters
- output parameters

#### Not Declared

IAM permissions (must have the appropriate permissions for all the resources that are defined in the template)

#### Challenges

- reuse
  - CF: input parameters
  - AWS IAM: policy variables
- order to provision resources
  - explicit
  - implicit through depends on (what CF does)
- supporting all life-cycle actions
  - create, read, update, delate
  - setup/configure
- how to define resource delete
  - OpenStack: delete in template and do stack update
- Semantics
  - all or nothing - if one resource fails then all fail
  - partial creation - is it useful in any situation?

#### CFN Template

```yaml
AWSTemplateFormatVersion: "version date"
Description:
	String
Metadata:
	tempalte metadata
Parameters:
	set of parameters
Mappings:
	set of mappings
Conditions:
	set of conditions
Transform:
	set of transforms
Resources:   \\ this one is required
	ResouceLogicalName:
		Type:AWS::Service::Resource
		Properties:
			Propterty1: someProperty
			Property2: someOtherProperty
Outputs:
	set of outputs
```

#### Resources Example

```yaml
Resources:
	HelloBucket:
		Type:AWS::S3::Bucket
		Properties:
			AccessControl: PublicRead
			WebsiteConfiguration:
				IndexDocument: index.html
				ErrorDocument: error.html
```

**Logical Name** - name used in the template

**Physical Name** - LogicalName+Stack Name+Unique ID (generated by AWS)

#### Resource Attributes

**Creation Policy** - prevents status from reaching create complete until CF receives specified number of success signals or timeout exceeded

```yaml
CreationPolicy:
	AutoScalingCreationPolicy:
		MinSuccessfulInstancesPercent: Integer
	ResourceSignal:
		Count: Integer
		Timeout: String
```

**DeletionPolicy** - 

preserve state before stack is deleted:

```yaml
Resources:
	myS3Bucket:
		Type:AWS::S3::Bucket
		DeletionPolicy: Snapshot
```

keep resource after stack is deleted:

```yaml
Resources:
	myS3Bucket:
		Type:AWS::S3::Bucket
		DeletionPolicy: Retain
```

**DependsOn** - specify which resources the resource depends upon essentially specifying an order

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
	Ec2Instance:
		Type:AWS::EC2::Instance
		DependsOn: myDB
	myDB:
		Type:AWS::RDS::DBInstance
		Properties:
			AllocatedStorage: '5'
			DBInstanceClass: db.m1.small
			EngineVersion: '5.5'
			MasterUsername: MyName
			MasterUserPassword: MyPassword
```

**Metadata** - allows the association of structured data with a  resource

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
	MyS3Bucket:
		Type:AWS::S3::Bucket
		Metadata:
			Object1: Location1
			Object2: Location2
```

#### Intrinsic Functions

- used to assign values to properties that are not available until runtime

##### Ref

- inbuilt function in AWS

Input: logical name of resource/parameter

Output: Physical name of resource/parameter

##### GetAtt

- in-built function to get attributes of a resource

Input: logical name of rate resource & name of the attribute to be retrieved

Output: attribute value

##### Input Parameters

- parameter defiition
  - type of parameter (string, num, aws type)
  - constraints on parameter val
  - default value

##Platform & PaaS

> **Platform** - set of one or more infastructure resources configured to run specific kinds of applications

### Platform as a service

- a web service that applies transformations on infrastructure resource to generate the required platform
- current PaaSes also deploy app on platform
- typically internally PaaS systems use IaaS systems to create resources
- can use IaC template to pull application code, build it and deploy it for static applications but IaC systems were optimized for working with infrastructure
  - IaC is built mainly for Operations Engineers vs PaaSes which are fore application developers
  - both use IaaS behind the scenes (PaaS might even use IaC)
- PaaS allows app developers to do production deployment by themselves
- lead to DevOps

#### Features:

- build and deploy applicaiton
- load balancing for app instances
- provide DNS name for application endpoint connected to load ballencer
- provide ability to configure load balancer with SSL certificates
- manual and automatic instance scaling
- **Canary development** - test new app with a subset of users
- app dev with no downtime (**blue/green deployment**)
- roll back deployment application
- collect application logs
- collect application metrics

### DevOps

> **DevOps**
>
> - functional roll (devs who occasionally perform operations duties)
> - set of steps (operations engineers who use dev practices ex. codified infrastructure, testing infrastructure code, version control)

In reality its a mix of both: app devs should be aware of infrastructure such as IaC etc and app engineers should practice good dev practices

## Web Apps

**Web Pages** - static

**Web Apps** - dynamic

- dynamic content
- persistent storage
- API access most of the time
- responsive
- secure
- scalable
- robust and reliable

**Web sites** - web pages + web apps

### Elements

- HTTP
- frameworks (python flask, django, java servlets)
- architecture (layers)
  - HTML UI
  - web app request handling
  - business logic
  - data access
  - db

## Elastic Beanstalk

- PaaS
- supports dev and deployment of source code
- deploys on EC2 instances
  - handles provisioning and deployment
  - also deploys elastic load balancer with a public DNS record

**Application**

folder/directory containing the source code

**Application version**

version of the code stored in S3

**Environment**

application version deployed on a set of AWS resources

**Environment Tier**

determines if deploying frontend web app or backend job processor

**Environment Configuration**

determines parameters and settings of environment resources

### Host Manager

beanstalk software component on each EC2 instance

- deploys app
- aggregates events and metrics for retrieval via console
- generating instance level events
- monitoring the application log files for critical errors
- monitoring the application server
- patching instance components
- rotating application log files and publishing them to S3



## Database Basics

## Docker

Solves the problem of how to get applications to behave the same in dev and deployment and the same on other machines

- repeatability
  - hardware similarities are solved with something like Elastic beanstalk a PaaS or IaC service
  - consistency of language libraries is solved using things like requirements and OS level packages are solved with containers by packaging the OS
- utilization: containers increase utilization of the machine by running multiple applications on a single physical machine
- resource isolation: should have separate file systems, processes and network traffic should be separate and no container should be able to affect other running containers

### How Docker Works

- pulls base OS image
- executes each line in docker file on base image
- builds the file image in the form of layers
- each line is a layer

### Twelve Factor App Manifesto

1. version control
2. declare and isolate dependencies
3. store application config in environment
4. treat backing services as attached resource
5. separate build and run stages
6. execute application as processes
7. expose applications through ports
8. scale app through process model
9. fast app startup and graceful shutdown
10. parity between environment
11. generate log streams
12. run admin tasks as one off processes

## Container Ochestration

### Advantages of Containers

- better utilization of the host
- repeatability of deployments
- abstract away host OS to application (doesn't matter what OS its run on)

### Container Ochestration

allows you to run containers across a number of hosts

#### Orchestration Systems

- docker swarm
- apache mesos
- ECS amazon elastic container service
- kubernetes - most advanced among these systems

## Containers and Managed Services

how to bind and authorize the container with a managed service like SQL database:

bind through env variable

giving permissions:

- manually modify the security group
- automate modifying the security group with something like CaaStle
- proxy container that all app containers communicate with and the proxy container communicates with the service

## Google Cloud Platform GCP

### GCP IAM

- google account for console access
- service account for programatic access

**Resources** organized in a hierarchical manner with project at the top

**Permissions** determine what operations are allowed on a resource (<service>.<resource>.<verb>) and typically correspond 1 to 1

- permissions can not be assigned to users directly. they must be assigned to a role and the user gains them through that role
- Roles
  - primitive roles
    - owner, editor, viewer
  - predefined roles
    - roles that give fine grained permissions that the primitive roles cannot satisfy
  - custom roles
    - roles that you can creat if predefined roles do not satisfy needs
- children in the resource hierarchy inherit permissions from parent and cannot restrict access to permissions granted at a higher level
- **effective policy** - union of policies defined at the resource
- user access
  - manual through browser good for 1 hour
  - service account w/ key file

## Microservices

### Monolithic Structure

- all functionality embedded in the same code
- pros
  - easy to dev
  - easy to deply
- cons
  - need to be developed by same team
  - typically entire app is in same language
  - independent evolution of application components is not possible
  - independent scaling of application components is not possible

### Microservice Architecture

- break up application functionality int o separate services that are accessible over HTTP/REST
- this is made easy by containerization and kubernetes (beanstalk and app engine are good for managing monolithic applications)
- need to be programed to assume other microservices are not available
- pros
  - multiple teams can work
  - independent evolution and scaling of application components
  - different languages
- cons
  - overhead when engining application development

## Kubernetes

Container orchestration system developed by google

- written in go

### Kubernetes Control Loop

if desired state does not equal current state then perform steps to get to desired state - this is the kubernetes control loop

### Basic Abstractions

- Pod
  - represents a unit of deployment
    - application container
    - storage resources required by the container
    - unique network IP address
    - options that govern how the container should run
- Service
  - defines a logical set of pods and a poly by which to access them
  - **label selector** - determines set of pods targeted by a service
  - each pod has a unique IP but the IP is not exposed to the outside world without a service
  - allow your application to receive traffic
    - cluster IP: exposes the cluster on an internal IP in the cluster
    - Node Port: exposes the service on the selected nodes
    - load balancer: creates external load balancer in underlying cloud and assigns external IP
    - External Name: exposes service using arbitrary name
- Volume
- Namespace

### Deployment Controller

- declarative model for creating/updating pods
- way to expose pods to outside world

### CI/CD

- in order to keep the IP address constant you need to create an ingress object

### Namespaces

mechanism that allows using same kubernetes cluster for running pods, services, deployments etc

### Labels

mechanism for selecting and accessing kubernetes objects

## Cloud Computing

#### Advantages

- no upfront investment
- more secure
- easily adopt dev ops/agile software dev practices
- quickly adopt newer technologies
- available across the globe

### Concerns

- vendor lock in - trapped at one org because all your shit is there
- migration - how to move a shit load of data when that company does you dirty
- cloud expertise - how to best utilize

##### Solutions

- dont migrate everything
- spread across multiple providers
- use a provider that uses an open source tech like open stack

### Multicloud

- using more than one cloud provider
- use software that supports multiple clouds
  - first mile
  - caastle
  - cloud foundry
  - terraform
  - gravitant/ibm

### Private Cloud

- enterprise gets their own machines for their info
- vmware, microsoft system center, open stack

## CICD

### Continuous Integration

- continuously testing software
  - unit tests
  - integrations tests
  - functional tests
  - load tests
  - penetration tests

### Continuous Delivery

