
# AWS Overview and EC2 & Security Groups

## AWS (Amazon Web Services)

AWS is a leading cloud provider, similar to Azure and Alibaba Cloud, offering on-demand cloud computing resources. It provides a wide range of interconnected services used by major companies like Netflix, Twitter, and LinkedIn.

## AWS Products

> **EC2 (Elastic Compute Cloud):** Virtual machines.

> **ECR (Elastic Container Registry):** Container image storage.

> **ECS (Elastic Container Service):** Container orchestration.

> **Elastic Beanstalk:** Deployment service for web applications.

> **Lambda:** Serverless computing service.

> **Elastic Load Balancing:** Distributes incoming traffic across multiple targets.

> **CloudFront:** Content delivery network with low latency.

> **CloudFormation:** Infrastructure as code service, similar to Ansible.

> **Kinesis:** Real-time data streaming service, akin to Kafka.

> **Route 53:** Domain name service.

> **S3 (Simple Storage Service):** Scalable object storage.

> **RDS, Aurora, DynamoDB:** Managed database services.

> **Auto Scaling:** Automatically adjusts EC2 instances based on demand.

> **Cognito:** Authentication service.

> **IAM (Identity and Access Management):** Security and access control.

> **CloudWatch:** Monitoring and logging service.

> **CodeCommit, CodeBuild, CodeDeploy, CodePipeline:** Services for source control, continuous integration, and continuous deployment.

## AWS Timeline

- 2002: AWS started.
- 2004: First public service, SQS.
- 2006: SQS, S3, EC2 released.
- 2007: Expanded globally beyond North America.

AWS is a leader in the cloud computing space, used for various IT platforms and applications, including storage, IT backup, website hosting, mobile, social applications, and gaming.

## AWS Infrastructure

> **Regions:** Geographically separated areas where AWS data centers are located.
Availability Zones: Isolated locations within regions.

## IAM & AWS CLI

> **Root Account:** Only used for account creation; avoid using it for daily tasks.

> **User Groups and Permissions:** Assign users to groups and manage permissions through IAM.

## Security Best Practices:
Create strong passwords and enforce Multi-Factor Authentication (MFA).
Use roles to allow AWS services to interact.
Regularly check IAM credentials and do not share access keys.

## EC2 – Elastic Compute Cloud

> **EC2 Overview:** A collection of services under IaaS, providing virtual machines, persistent storage, load balancing, and auto-scaling.

> **EC2 User Data:** Allows customization of instances at startup, executing scripts for tasks like software installation.

### EC2 Instance Types:
> **General Purpose:** Balanced CPU, memory, and networking.

> **Compute Optimized:** High CPU power for tasks like media transcoding.

> **Memory Optimized:** Large memory capacity for databases and big data.

> **Storage Optimized:** High I/O operations for large datasets.

### EC2 Security Groups

Security Groups act as firewalls, controlling traffic to/from EC2 instances:

> **Allow Rules Only:** Manage inbound and outbound traffic with specified IP ranges and ports.

> **Regional and VPC Specific:** Security Groups must be redefined if regions or VPCs change.

> **Best Practices:** Separate Security Groups for SSH and ensure proper configurations for application accessibility.

### EC2 Buying Options


> **On-Demand Instances:** Pay-as-you-go, suitable for short-term workloads.

> **Reserved Instances:** Discounted rates for long-term commitments (1-3 years).

> **Spot Instances:** Cost-effective but can be terminated unexpectedly.

> **Dedicated Hosts:** Entire physical servers reserved, useful for compliance and licensing needs.

## AWS CloudFormation Example

A simple CloudFormation template to create an EC2 instance:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Basit bir EC2 t2.micro instance'1 baslatan CloudFormation sablonu.

Resources:
  EC2Instance:
    Type: "AWS::EC2::Instance"
    Properties:
      InstanceType: 't2.micro'
      ImageId: 'ami-0c55b159cbfafe1f0'
      KeyName: 'your-key-pair'

Outputs:
  Instanceld:
    Description: 'EC2 Instance ID'
    Value: !Ref EC2Instance
````

## Commands to manage the CloudFormation stack:

```bash
aws cloudformation create-stack --stack-name MyEC2Stack --template-body file://template.yaml

aws cloudformation delete-stack --stack-name MyEC2Stack 

aws cloudformation describe-stacks --stack-name MyEC2Stack
```

# Case (Hand-on)

In the case part, we created an AWS account, and launched a free tier EC2 instance.