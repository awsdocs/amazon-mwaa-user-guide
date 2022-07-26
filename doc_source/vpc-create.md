# Create the VPC network<a name="vpc-create"></a>

Amazon Managed Workflows for Apache Airflow \(MWAA\) requires an Amazon VPC and specific networking components to support an environment\. This guide describes the different options to create the Amazon VPC network for an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment\.

**Note**  
 Apache Airflow works best in a low\-latency network environment\. If you are using an existing Amazon VPC which routes traffic to another region or to an on\-premise environment, we recommended adding AWS PrivateLink endpoints for Amazon SQS, CloudWatch, Amazon S3, AWS KMS, and Amazon ECR\. For more information about configuring AWS PrivateLink for Amazon MWAA, see [Creating an Amazon VPC network without internet access](#vpc-create-template-private-only)\. 

**Contents**
+ [Prerequisites](#vpc-create-prereqs)
+ [Before you begin](#vpc-create-how-networking)
+ [Options to create the Amazon VPC network](#vpc-create-options)
  + [Option one: Creating the VPC network on the Amazon MWAA console](#vpc-create-mwaa-console)
  + [Option two: Creating an Amazon VPC network *with* Internet access](#vpc-create-template-private-or-public)
  + [Option three: Creating an Amazon VPC network *without* Internet access](#vpc-create-template-private-only)
+ [What's next?](#create-vpc-next-up)

## Prerequisites<a name="vpc-create-prereqs"></a>

The AWS Command Line Interface \(AWS CLI\) is an open source tool that enables you to interact with AWS services using commands in your command\-line shell\. To complete the steps on this page, you need the following:
+ [AWS CLI – Install version 2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)\.
+ [AWS CLI – Quick configuration with `aws configure`](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)\.

## Before you begin<a name="vpc-create-how-networking"></a>
+ The [VPC network](#vpc-create) you specify for your environment can't be changed after the environment is created\.
+ You can use private or public routing for your Amazon VPC and Apache Airflow *Web server*\. To view a list of options, see [Example use cases for an Amazon VPC and Apache Airflow access mode](networking-about.md#networking-about-network-usecase)\.

## Options to create the Amazon VPC network<a name="vpc-create-options"></a>

The following section describes the options available to create the Amazon VPC network for an environment\. 

### Option one: Creating the VPC network on the Amazon MWAA console<a name="vpc-create-mwaa-console"></a>

The following section shows how to create an Amazon VPC network on the Amazon MWAA console\. This option uses [Public routing over the Internet](networking-about.md#networking-about-overview-public)\. It can be used for an Apache Airflow *Web server* with the **Private network** or **Public network** access modes\.

The following image shows where you can find the **Create MWAA VPC** button on the Amazon MWAA console\. 

![\[This image shows where you can find the Create MWAA VPC on the Amazon MWAA console.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-create-vpc.png)

### Option two: Creating an Amazon VPC network *with* Internet access<a name="vpc-create-template-private-or-public"></a>

The following AWS CloudFormation template creates an Amazon VPC network *with Internet access* in your default AWS Region\. This option uses [Public routing over the Internet](networking-about.md#networking-about-overview-public)\. This template can be used for an Apache Airflow *Web server* with the **Private network** or **Public network** access modes\. 

1. Copy the contents of the following template and save locally as `cfn-vpc-public-private.yaml`\. You can also [download the template](./samples/cfn-vpc-public-private.zip)\.

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
   
     SecurityGroup:
       Type: AWS::EC2::SecurityGroup
       Properties:
         GroupName: "mwaa-security-group"
         GroupDescription: "Security group with a self-referencing inbound rule."
         VpcId: !Ref VPC
   
     SecurityGroupIngress:
       Type: AWS::EC2::SecurityGroupIngress
       Properties:
         GroupId: !Ref SecurityGroup
         IpProtocol: "-1"
         SourceSecurityGroupId: !Ref SecurityGroup
   
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
   
     SecurityGroupIngress:
       Description: Security group with self-referencing inbound rule
       Value: !Ref SecurityGroupIngress
   ```

1. In your command prompt, navigate to the directory where `cfn-vpc-public-private.yaml` is stored\. For example:

   ```
   cd mwaaproject
   ```

1. Use the [aws cloudformation create\-stack](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/create-stack.html) command to create the stack using the AWS CLI\.

   ```
   aws cloudformation create-stack --stack-name mwaa-environment --template-body file://cfn-vpc-public-private.yaml
   ```
**Note**  
It takes about 30 minutes to create the Amazon VPC infrastructure\.

### Option three: Creating an Amazon VPC network *without* Internet access<a name="vpc-create-template-private-only"></a>

The following AWS CloudFormation template creates an Amazon VPC network *without Internet access* in your default AWS region\.

**Important**  
 When using a Amazon VPC without internet access, you must grant permission to Amazon ECR to access Amazon S3 using a gateway endpoint\. You can create a gateway endpoint by doing the following:   
 Copy the following `JSON` IAM policy, and save it locally as `s3-gw-endpoint-policy.json`\. The policy grants the minimum required permission for Amazon ECR to access Amazon S3 resources\.   

   ```
   {
     "Statement": [
       {
         "Sid": "Access-to-specific-bucket-only",
         "Principal": "*",
         "Action": [
           "s3:GetObject"
         ],
         "Effect": "Allow",
         "Resource": ["arn:aws:s3:::prod-region-starport-layer-bucket/*"]
       }
     ]
   }
   ```
 Create the endpoint using the following AWS CLI command\. Replace the values for `--vpc-id` and `--route-table-ids` with the information for your Amazon VPC\. Replace `--service-name` with the name according to your region\.   

   ```
   $ aws ec2 create-vpc-endpoint --vpc-id vpc-1a2b3c4d \
   --service-name com.amazonaws.us-west-2.s3 \
   --route-table-ids rtb-11aa22bb \
   --vpc-endpoint-type Gateway \
   --policy-document file://s3-gw-endpoint-policy.json
   ```
For more information about creating Amazon S3 gateway endpoints for Amazon ECR, see [Create the Amazon S3 gateway endpoint](https://docs.aws.amazon.com/AmazonECR/latest/userguide/vpc-endpoints.html#ecr-setting-up-s3-gateway) in the *Amazon Elastic Container Registry User Guide*\.

 This option uses [Private routing without Internet access](networking-about.md#networking-about-overview-private)\. This template can be used for an Apache Airflow *Web server* with the **Private network** access mode only\. It creates the required [VPC endpoints for the AWS services used by an environment](vpc-vpe-create-access.md#vpc-vpe-create-view-endpoints-attach-services)\. 

1. Copy the contents of the following template and save locally as `cfn-vpc-private.yaml`\. You can also [download the template](./samples/cfn-vpc-private.zip)\.

   ```
   AWSTemplateFormatVersion: "2010-09-09"
        
   Parameters:
      VpcCIDR:
        Description: The IP range (CIDR notation) for this VPC
        Type: String
        Default: 10.192.0.0/16
        
      PrivateSubnet1CIDR:
        Description: The IP range (CIDR notation) for the private subnet in the first Availability Zone
        Type: String
        Default: 10.192.10.0/24
        
      PrivateSubnet2CIDR:
        Description: The IP range (CIDR notation) for the private subnet in the second Availability Zone
        Type: String
        Default: 10.192.11.0/24
        
   Resources:
      VPC:
        Type: AWS::EC2::VPC
        Properties:
          CidrBlock: !Ref VpcCIDR
          EnableDnsSupport: true
          EnableDnsHostnames: true
          Tags:
           - Key: Name
             Value: !Ref AWS::StackName
        
      RouteTable:
        Type: AWS::EC2::RouteTable
        Properties:
          VpcId: !Ref VPC
          Tags:
           - Key: Name
             Value: !Sub "${AWS::StackName}-route-table"
        
      PrivateSubnet1:
        Type: AWS::EC2::Subnet
        Properties:
          VpcId: !Ref VPC
          AvailabilityZone: !Select [ 0, !GetAZs  '' ]
          CidrBlock: !Ref PrivateSubnet1CIDR
          MapPublicIpOnLaunch: false
          Tags:
           - Key: Name
             Value: !Sub "${AWS::StackName} Private Subnet (AZ1)"
        
      PrivateSubnet2:
        Type: AWS::EC2::Subnet
        Properties:
          VpcId: !Ref VPC
          AvailabilityZone: !Select [ 1, !GetAZs  '' ]
          CidrBlock: !Ref PrivateSubnet2CIDR
          MapPublicIpOnLaunch: false
          Tags:
           - Key: Name
             Value: !Sub "${AWS::StackName} Private Subnet (AZ2)"
        
      PrivateSubnet1RouteTableAssociation:
        Type: AWS::EC2::SubnetRouteTableAssociation
        Properties:
          RouteTableId: !Ref RouteTable
          SubnetId: !Ref PrivateSubnet1
        
      PrivateSubnet2RouteTableAssociation:
        Type: AWS::EC2::SubnetRouteTableAssociation
        Properties:
          RouteTableId: !Ref RouteTable
          SubnetId: !Ref PrivateSubnet2
        
      S3VpcEndoint:
        Type: AWS::EC2::VPCEndpoint
        Properties:
          ServiceName: !Sub "com.amazonaws.${AWS::Region}.s3"
          VpcEndpointType: Gateway
          VpcId: !Ref VPC
          RouteTableIds:
           - !Ref RouteTable
        
      SecurityGroup:
        Type: AWS::EC2::SecurityGroup
        Properties:
          VpcId: !Ref VPC
          GroupDescription: Security Group for Amazon MWAA Environments to access VPC endpoints
          GroupName: !Sub "${AWS::StackName}-mwaa-vpc-endpoints"
      
      SecurityGroupIngress:
        Type: AWS::EC2::SecurityGroupIngress
        Properties:
          GroupId: !Ref SecurityGroup
          IpProtocol: "-1"
          SourceSecurityGroupId: !Ref SecurityGroup
      
      SqsVpcEndoint:
        Type: AWS::EC2::VPCEndpoint
        Properties:
          ServiceName: !Sub "com.amazonaws.${AWS::Region}.sqs"
          VpcEndpointType: Interface
          VpcId: !Ref VPC
          PrivateDnsEnabled: true
          SubnetIds:
           - !Ref PrivateSubnet1
           - !Ref PrivateSubnet2
          SecurityGroupIds:
           - !Ref SecurityGroup
        
      CloudWatchLogsVpcEndoint:
        Type: AWS::EC2::VPCEndpoint
        Properties:
          ServiceName: !Sub "com.amazonaws.${AWS::Region}.logs"
          VpcEndpointType: Interface
          VpcId: !Ref VPC
          PrivateDnsEnabled: true
          SubnetIds:
           - !Ref PrivateSubnet1
           - !Ref PrivateSubnet2
          SecurityGroupIds:
           - !Ref SecurityGroup
        
      CloudWatchMonitoringVpcEndoint:
        Type: AWS::EC2::VPCEndpoint
        Properties:
          ServiceName: !Sub "com.amazonaws.${AWS::Region}.monitoring"
          VpcEndpointType: Interface
          VpcId: !Ref VPC
          PrivateDnsEnabled: true
          SubnetIds:
           - !Ref PrivateSubnet1
           - !Ref PrivateSubnet2
          SecurityGroupIds:
           - !Ref SecurityGroup
        
      KmsVpcEndoint:
        Type: AWS::EC2::VPCEndpoint
        Properties:
          ServiceName: !Sub "com.amazonaws.${AWS::Region}.kms"
          VpcEndpointType: Interface
          VpcId: !Ref VPC
          PrivateDnsEnabled: true
          SubnetIds:
           - !Ref PrivateSubnet1
           - !Ref PrivateSubnet2
          SecurityGroupIds:
           - !Ref SecurityGroup
        
      EcrApiVpcEndoint:
        Type: AWS::EC2::VPCEndpoint
        Properties:
          ServiceName: !Sub "com.amazonaws.${AWS::Region}.ecr.api"
          VpcEndpointType: Interface
          VpcId: !Ref VPC
          PrivateDnsEnabled: true
          SubnetIds:
           - !Ref PrivateSubnet1
           - !Ref PrivateSubnet2
          SecurityGroupIds:
           - !Ref SecurityGroup
        
      EcrDkrVpcEndoint:
        Type: AWS::EC2::VPCEndpoint
        Properties:
          ServiceName: !Sub "com.amazonaws.${AWS::Region}.ecr.dkr"
          VpcEndpointType: Interface
          VpcId: !Ref VPC
          PrivateDnsEnabled: true
          SubnetIds:
           - !Ref PrivateSubnet1
           - !Ref PrivateSubnet2
          SecurityGroupIds:
           - !Ref SecurityGroup
   
      AirflowApiVpcEndoint:
        Type: AWS::EC2::VPCEndpoint
        Properties:
          ServiceName: !Sub "com.amazonaws.${AWS::Region}.airflow.api"
          VpcEndpointType: Interface
          VpcId: !Ref VPC
          PrivateDnsEnabled: true
          SubnetIds:
           - !Ref PrivateSubnet1
           - !Ref PrivateSubnet2
          SecurityGroupIds:
           - !Ref SecurityGroup  
           
      AirflowEnvVpcEndoint:
        Type: AWS::EC2::VPCEndpoint
        Properties:
          ServiceName: !Sub "com.amazonaws.${AWS::Region}.airflow.env"
          VpcEndpointType: Interface
          VpcId: !Ref VPC
          PrivateDnsEnabled: true
          SubnetIds:
           - !Ref PrivateSubnet1
           - !Ref PrivateSubnet2
          SecurityGroupIds:
           - !Ref SecurityGroup   
                                            
      AirflowOpsVpcEndoint:
        Type: AWS::EC2::VPCEndpoint
        Properties:
          ServiceName: !Sub "com.amazonaws.${AWS::Region}.airflow.ops"
          VpcEndpointType: Interface
          VpcId: !Ref VPC
          PrivateDnsEnabled: true
          SubnetIds:
           - !Ref PrivateSubnet1
           - !Ref PrivateSubnet2
          SecurityGroupIds:
           - !Ref SecurityGroup
   
   Outputs:
      VPC:
        Description: A reference to the created VPC
        Value: !Ref VPC
        
      MwaaSecurityGroupId:
        Description: Associates the Security Group to the environment to allow access to the VPC endpoints 
        Value: !Ref SecurityGroup
        
      PrivateSubnets:
        Description: A list of the private subnets
        Value: !Join [ ",", [ !Ref PrivateSubnet1, !Ref PrivateSubnet2 ]]
        
      PrivateSubnet1:
        Description: A reference to the private subnet in the 1st Availability Zone
        Value: !Ref PrivateSubnet1
        
      PrivateSubnet2:
        Description: A reference to the private subnet in the 2nd Availability Zone
        Value: !Ref PrivateSubnet2
   ```

1. In your command prompt, navigate to the directory where `cfn-vpc-private.yml` is stored\. For example:

   ```
   cd mwaaproject
   ```

1. Use the [aws cloudformation create\-stack](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/create-stack.html) command to create the stack using the AWS CLI\.

   ```
   aws cloudformation create-stack --stack-name mwaa-private-environment --template-body file://cfn-vpc-private.yml
   ```
**Note**  
It takes about 30 minutes to create the Amazon VPC infrastructure\.

1. You'll need to create a mechanism to access these VPC endpoints from your computer\. To learn more, see [Managing access to VPC endpoints on Amazon MWAA](vpc-vpe-access.md)\.

**Note**  
 You can further restrict outbound access in the CIDR of your Amazon MWAA security group\. For example, you can restrict to itself by adding a self\-referencing outbound rule, the [prefix list](https://docs.aws.amazon.com/vpc/latest/privatelink/vpce-gateway.html) for Amazon S3, and the CIDR of your Amazon VPC\. 

## What's next?<a name="create-vpc-next-up"></a>
+ Learn how to create an Amazon MWAA environment in [Create an Amazon MWAA environment](create-environment.md)\.
+ Learn how to create a VPN tunnel from your computer to your Amazon VPC with private routing in [Tutorial: Configuring private network access using an AWS Client VPN](tutorials-private-network-vpn-client.md)\.