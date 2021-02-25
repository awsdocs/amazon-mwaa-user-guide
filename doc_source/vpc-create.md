# Create the VPC network<a name="vpc-create"></a>

To use Amazon Managed Workflows for Apache Airflow \(MWAA\), you'll need the VPC networking components required by an Amazon MWAA environment\.

You can use an existing VPC that meets these requirements, create the VPC and networking components on the Amazon MWAA console, or use the provided AWS CloudFormation template to create the VPC and other required networking components\. This guide describes the required VPC networking components, and the steps to create the VPC network for an Amazon MWAA environment using the Amazon MWAA console, or using an Amazon MWAA AWS CloudFormation template\.

**Topics**
+ [Required VPC networking components](#vpc-create-required)
+ [Prerequisites](#vpc-create-prereqs)
+ [How it works](#vpc-create-how)
+ [Create Amazon MWAA VPC](#vpc-create-onconsole)
+ [Using an Amazon MWAA AWS CloudFormation template](#vpc-create-template-code)
+ [What's next?](#create-vpc-next-up)

## Required VPC networking components<a name="vpc-create-required"></a>

Your Amazon VPC and its networking components must meet the following requirements to support an Amazon MWAA environment:

1. Two private subnets in two different availability zones within the same Region\.

1. You'll also need one of the following:

   1. Two public subnets that are configured to route the private subnet data to the Internet\.

   1. Or [VPC endpoint services \(AWS PrivateLink\)](https://docs.aws.amazon.com/vpc/latest/userguide/endpoint-service.html) access to the AWS services used by your environment\.

**Note**  
If you are unable to provide Internet routing for your two private subnets, [VPC endpoint services \(AWS PrivateLink\)](https://docs.aws.amazon.com/vpc/latest/userguide/endpoint-service.html) access to the AWS services used by your environment \(Amazon CloudWatch, CloudWatch Logs, Amazon ECR, Amazon S3, Amazon SQS, AWS Key Management Service\) is required\.

## Prerequisites<a name="vpc-create-prereqs"></a>

The AWS Command Line Interface \(AWS CLI\) is an open source tool that enables you to interact with AWS services using commands in your command\-line shell\. To complete the steps in this section, you need the following:
+ [AWS CLI – Install version 2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
+ [AWS CLI – Quick configuration with `aws configure`](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)

## How it works<a name="vpc-create-how"></a>

This section describes the public and private network options, and the architecture of an Amazon MWAA environment\.

### Environment architecture<a name="configuring-networking-infra"></a>

The following image shows the architecture of a Amazon MWAA environment\.

![\[This image shows the architecture of a Amazon MWAA environment.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-architecture.png)
+ When you create an environment, Amazon MWAA creates an AWS\-managed Amazon Aurora PostgreSQL metadata database and an Fargate container in each of your two private subnets in different availability zones\. For example, a metadata database and container in `us-east-1a` and a metadata database and container in `us-east-1b` availability zones for the `us-east-1` region\.
+ The environment class you choose for your Amazon MWAA environment determines the size of the AWS\-managed AWS Fargate containers where the [Celery Executor](https://airflow.apache.org/docs/apache-airflow/stable/executor/celery.html) runs, and the AWS\-managed Amazon Aurora PostgreSQL metadata database where the Apache Airflow scheduler creates task instances\.
+ The Apache Airflow workers on an Amazon MWAA environment use the [Celery Executor](https://airflow.apache.org/docs/apache-airflow/stable/executor/celery.html) to queue and distribute tasks to multiple Celery workers from an Apache Airflow platform\. The Celery Executor runs in an AWS Fargate container\. If a Fargate container in one availability zone fails, Amazon MWAA switches to the other container in a different availability zone to run the Celery Executor, and the Apache Airflow scheduler creates a new task instance in the Amazon Aurora PostgreSQL metadata database\.

### Private and public network<a name="vpc-create-how-networking"></a>
+ Amazon MWAA provides private and public networking options for your Apache Airflow web server\. A public network allows the Apache Airflow UI to be accessed over the Internet by users granted access to your IAM policy\. A private network limits access to the Apache Airflow UI to users within your VPC\. Additional configuration is required to use a private network\. To learn more, see [Amazon MWAA network access](configuring-networking.md)\.

## Create Amazon MWAA VPC<a name="vpc-create-onconsole"></a>

You can use the Amazon MWAA console to create the VPC network needed for an Amazon MWAA environment\. The following image shows where you can find the **Create MWAA VPC** button on the Amazon MWAA console\.

![\[This image shows where you can find the Create MWAA VPC on the MWAA console.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-create-vpc.png)

**Note**  
This creates a VPC and the required networking components for an environment\. You need to create and configure additional resources for a private network\. It can take several minutes to create the VPC network\.

## Using an Amazon MWAA AWS CloudFormation template<a name="vpc-create-template-code"></a>

AWS CloudFormation allows you to create AWS resources using a template that describes the AWS resources you want you want to create\. The following section describes how to use an Amazon MWAA AWS CloudFormation template to create the VPC network in your default region\.

### AWS CloudFormation VPC stack specifications<a name="vpc-create-template-components"></a>

The following Amazon MWAA AWS CloudFormation template creates a VPC network in your default region with the following specifications\.
+ a VPC with a `10.192.0.0/16` CIDR rule
+ a VPC security group that directs all inbound traffic to your Amazon MWAA environment and all outbound traffic to `0.0.0.0/0`
+ one public subnet with a `10.192.10.0/24` CIDR rule in your region's first availability zone
+ one public subnet with a `10.192.11.0/24` CIDR rule in your region's second availability zone
+ one private subnet with a `10.192.20.0/24` CIDR rule in your region's first availability zone
+ one private subnet with a `10.192.21.0/24` CIDR rule in your region's second availability zone
+ creates and attaches an Internet gateway to the public subets
+ creates and attaches two NAT gateways to the private subnets
+ creates and attaches two elastic IP addresses \(EIPs\) to the NAT gateways
+ creates and attaches two public route tables to the public subnets
+ creates and attaches two private route tables to the private subnets

### Create the AWS CloudFormation VPC template<a name="vpc-create-template-components"></a>

Copy the template and paste it to a new file, then save it as `vpctemplate.yaml` in preparation for the next step\. You can also [download the template](./samples/mwaa-vpc-cfn.zip)\.

```
Description:  This template deploys a VPC, with a pair of public and private subnets spread
  across two Availability Zones. It deploys an internet gateway, with a default
  route on the public subnets. It deploys a pair of NAT gateways (one in each AZ),
  and default routes for them in the private subnets.

Parameters:
  EnvironmentName:
    Description: An environment name that is prefixed to resource names
    Type: String
    Default: mwaa-

  VpcCIDR:
    Description: Please enter the IP range (CIDR notation) for this VPC
    Type: String
    Default: 10.192.0.0/16

  PublicSubnet1CIDR:
    Description: Please enter the IP range (CIDR notation) for the public subnet in the first Availability Zone
    Type: String
    Default: 10.192.10.0/24

  PublicSubnet2CIDR:
    Description: Please enter the IP range (CIDR notation) for the public subnet in the second Availability Zone
    Type: String
    Default: 10.192.11.0/24

  PrivateSubnet1CIDR:
    Description: Please enter the IP range (CIDR notation) for the private subnet in the first Availability Zone
    Type: String
    Default: 10.192.20.0/24

  PrivateSubnet2CIDR:
    Description: Please enter the IP range (CIDR notation) for the private subnet in the second Availability Zone
    Type: String
    Default: 10.192.21.0/24

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VpcCIDR
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: !Ref EnvironmentName

  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: !Ref EnvironmentName

  InternetGatewayAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      InternetGatewayId: !Ref InternetGateway
      VpcId: !Ref VPC

  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: !Select [ 0, !GetAZs '' ]
      CidrBlock: !Ref PublicSubnet1CIDR
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} Public Subnet (AZ1)

  PublicSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: !Select [ 1, !GetAZs  '' ]
      CidrBlock: !Ref PublicSubnet2CIDR
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} Public Subnet (AZ2)

  PrivateSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: !Select [ 0, !GetAZs  '' ]
      CidrBlock: !Ref PrivateSubnet1CIDR
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} Private Subnet (AZ1)

  PrivateSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: !Select [ 1, !GetAZs  '' ]
      CidrBlock: !Ref PrivateSubnet2CIDR
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} Private Subnet (AZ2)

  NatGateway1EIP:
    Type: AWS::EC2::EIP
    DependsOn: InternetGatewayAttachment
    Properties:
      Domain: vpc

  NatGateway2EIP:
    Type: AWS::EC2::EIP
    DependsOn: InternetGatewayAttachment
    Properties:
      Domain: vpc

  NatGateway1:
    Type: AWS::EC2::NatGateway
    Properties:
      AllocationId: !GetAtt NatGateway1EIP.AllocationId
      SubnetId: !Ref PublicSubnet1

  NatGateway2:
    Type: AWS::EC2::NatGateway
    Properties:
      AllocationId: !GetAtt NatGateway2EIP.AllocationId
      SubnetId: !Ref PublicSubnet2

  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} Public Routes

  DefaultPublicRoute:
    Type: AWS::EC2::Route
    DependsOn: InternetGatewayAttachment
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  PublicSubnet1RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PublicRouteTable
      SubnetId: !Ref PublicSubnet1

  PublicSubnet2RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PublicRouteTable
      SubnetId: !Ref PublicSubnet2


  PrivateRouteTable1:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} Private Routes (AZ1)

  DefaultPrivateRoute1:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId: !Ref PrivateRouteTable1
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId: !Ref NatGateway1

  PrivateSubnet1RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PrivateRouteTable1
      SubnetId: !Ref PrivateSubnet1

  PrivateRouteTable2:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: !Sub ${EnvironmentName} Private Routes (AZ2)

  DefaultPrivateRoute2:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId: !Ref PrivateRouteTable2
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId: !Ref NatGateway2

  PrivateSubnet2RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PrivateRouteTable2
      SubnetId: !Ref PrivateSubnet2

  NoIngressSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupName: "no-ingress-sg"
      GroupDescription: "Security group with no ingress rule"
      VpcId: !Ref VPC

Outputs:
  VPC:
    Description: A reference to the created VPC
    Value: !Ref VPC

  PublicSubnets:
    Description: A list of the public subnets
    Value: !Join [ ",", [ !Ref PublicSubnet1, !Ref PublicSubnet2 ]]

  PrivateSubnets:
    Description: A list of the private subnets
    Value: !Join [ ",", [ !Ref PrivateSubnet1, !Ref PrivateSubnet2 ]]

  PublicSubnet1:
    Description: A reference to the public subnet in the 1st Availability Zone
    Value: !Ref PublicSubnet1

  PublicSubnet2:
    Description: A reference to the public subnet in the 2nd Availability Zone
    Value: !Ref PublicSubnet2

  PrivateSubnet1:
    Description: A reference to the private subnet in the 1st Availability Zone
    Value: !Ref PrivateSubnet1

  PrivateSubnet2:
    Description: A reference to the private subnet in the 2nd Availability Zone
    Value: !Ref PrivateSubnet2

  NoIngressSecurityGroup:
    Description: Security group with no ingress rule
    Value: !Ref NoIngressSecurityGroup
```

### Creating the VPC stack using the AWS CLI<a name="vpc-create-the-stack"></a>

Use the [aws cloudformation create\-stack](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/create-stack.html) command to create the VPC stack with the Amazon MWAA AWS CloudFormation template using the AWS CLI\.

```
aws cloudformation create-stack --stack-name mwaaenvironment --template-body file://vpctemplate.yaml
```

## What's next?<a name="create-vpc-next-up"></a>
+ Learn how to create an Amazon MWAA environment in [Create an Amazon MWAA environment](create-environment.md)\.