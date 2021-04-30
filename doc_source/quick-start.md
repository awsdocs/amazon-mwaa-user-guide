# Quick start tutorial for Amazon Managed Workflows for Apache Airflow \(MWAA\)<a name="quick-start"></a>

This quick start tutorial uses an AWS CloudFormation template that creates the Amazon VPC infrastructure, an Amazon S3 bucket with a `dags` folder, and an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment at the same time\. It then walks you through three AWS Command Line Interface \(AWS CLI\) commands to upload a DAG to Amazon S3, then run the DAG in Apache Airflow, and view logs in CloudWatch\. It concludes by walking you through the steps to create an IAM policy for an Apache Airflow development team\.

**Topics**
+ [In this tutorial](#quick-start-overview)
+ [Prerequisites](#quick-start-before)
+ [Step one: Save the AWS CloudFormation template locally](#quick-start-template)
+ [Step two: Create the stack using the AWS CLI](#quick-start-createstack)
+ [Step three: Upload a DAG to Amazon S3 and run in the Apache Airflow UI](#quick-start-upload-dag)
+ [Step four: View logs in CloudWatch Logs](#quick-start-logs)
+ [Step five: Create IAM policies for an Apache Airflow development team](#quick-start-create-group)
+ [What's next?](#quick-start-next-up)

## In this tutorial<a name="quick-start-overview"></a>

The AWS CloudFormation template on this page creates the following:
+ **VPC infrastructure**\. This template uses [Public routing over the Internet](networking-about.md#networking-about-overview-public)\. It uses the [Public network access mode](configuring-networking.md#access-overview-public) for the Apache Airflow *Web server* in `WebserverAccessMode: PUBLIC_ONLY`\.
+ **Amazon S3 bucket**\. The AWS CloudFormation template creates an Amazon S3 bucket with a `dags` folder\. It's configured to **Block all public access**, with **Bucket Versioning** enabled, as defined in [Create an Amazon S3 bucket for Amazon MWAA](mwaa-s3-bucket.md)\.
+ **Amazon MWAA environment**\. The AWS CloudFormation template creates an Amazon MWAA environment that is associated to the `dags` folder in the Amazon S3 bucket, an execution role with permission to AWS services used by Amazon MWAA, and it specifies the default for encryption using an [AWS owned CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk), as defined in [Create an Amazon MWAA environment](create-environment.md)\.
+ **CloudWatch Logs**\. The AWS CloudFormation template enables Apache Airflow logs in CloudWatch at the "INFO" level and up for the following log groups: *Airflow scheduler log group*, *Airflow web server log group*, *Airflow worker log group*, *Airflow DAG processing log group*, and the *Airflow task log group*\.

In this tutorial, you'll create and complete the following:
+ **Upload and run DAG**\. The tutorial walks you through the steps to upload Apache Airflow's tutorial DAG for Apache Airflow v1\.10\.12 to Amazon S3, and then run in the Apache Airflow UI, as defined in [Adding or updating DAGs](configuring-dag-folder.md)\.
+ **View logs**\. The tutorial walks you through the steps to view the *Airflow web server log group* in CloudWatch Logs\.
+ **Create IAM policy**\. The tutorial walks you through the steps to create an IAM policy for your Apache Airflow development team, as defined in [Accessing an Amazon MWAA environment](access-policies.md)\.

## Prerequisites<a name="quick-start-before"></a>

The AWS Command Line Interface \(AWS CLI\) is an open source tool that enables you to interact with AWS services using commands in your command\-line shell\. To complete the steps on this page, you need the following:
+ [AWS CLI – Install version 2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
+ [AWS CLI – Quick configuration with `aws configure`](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)

## Step one: Save the AWS CloudFormation template locally<a name="quick-start-template"></a>
+ Copy the contents of the following template and save locally as `mwaa_public_network.yaml`\. You can also [download the template](./samples/mwaa_public_network.zip)\.

  ```
  AWSTemplateFormatVersion: "2010-09-09"
  
  Parameters:
  
    EnvironmentName:
      Description: An environment name that is prefixed to resource names
      Type: String
      Default: MWAAEnvironment
  
    VpcCIDR:
      Description: The IP range (CIDR notation) for this VPC
      Type: String
      Default: 10.192.0.0/16
  
    PublicSubnet1CIDR:
      Description: The IP range (CIDR notation) for the public subnet in the first Availability Zone
      Type: String
      Default: 10.192.10.0/24
  
    PublicSubnet2CIDR:
      Description: The IP range (CIDR notation) for the public subnet in the second Availability Zone
      Type: String
      Default: 10.192.11.0/24
  
    PrivateSubnet1CIDR:
      Description: The IP range (CIDR notation) for the private subnet in the first Availability Zone
      Type: String
      Default: 10.192.20.0/24
    PrivateSubnet2CIDR:
      Description: The IP range (CIDR notation) for the private subnet in the second Availability Zone
      Type: String
      Default: 10.192.21.0/24
    MaxWorkerNodes:
      Description: The maximum number of workers that can run in the environment
      Type: Number
      Default: 2
    DagProcessingLogs:
      Description: Log level for DagProcessing
      Type: String
      Default: INFO
    SchedulerLogsLevel:
      Description: Log level for SchedulerLogs
      Type: String
      Default: INFO
    TaskLogsLevel:
      Description: Log level for TaskLogs
      Type: String
      Default: INFO
    WorkerLogsLevel:
      Description: Log level for WorkerLogs
      Type: String
      Default: INFO
    WebserverLogsLevel:
      Description: Log level for WebserverLogs
      Type: String
      Default: INFO
  
  Resources:
    #####################################################################################################################
    # CREATE VPC
    #####################################################################################################################
  
    VPC:
      Type: AWS::EC2::VPC
      Properties:
        CidrBlock: !Ref VpcCIDR
        EnableDnsSupport: true
        EnableDnsHostnames: true
        Tags:
          - Key: Name
            Value: MWAAEnvironment
  
    InternetGateway:
      Type: AWS::EC2::InternetGateway
      Properties:
        Tags:
          - Key: Name
            Value: MWAAEnvironment
  
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
  
    EnvironmentBucket:
      Type: AWS::S3::Bucket
      Properties:
        VersioningConfiguration:
          Status: Enabled
        PublicAccessBlockConfiguration: 
          BlockPublicAcls: true
          BlockPublicPolicy: true
          IgnorePublicAcls: true
          RestrictPublicBuckets: true
  
    #####################################################################################################################
    # CREATE MWAA
    #####################################################################################################################
  
    MwaaEnvironment:
      Type: AWS::MWAA::Environment
      DependsOn: MwaaExecutionPolicy
      Properties:
        Name: !Sub "${AWS::StackName}-MwaaEnvironment"
        SourceBucketArn: !GetAtt EnvironmentBucket.Arn
        ExecutionRoleArn: !GetAtt MwaaExecutionRole.Arn
        DagS3Path: dags
        NetworkConfiguration:
          SecurityGroupIds:
            - !GetAtt SecurityGroup.GroupId
          SubnetIds:
            - !Ref PrivateSubnet1
            - !Ref PrivateSubnet2
        WebserverAccessMode: PUBLIC_ONLY
        MaxWorkers: !Ref MaxWorkerNodes
        LoggingConfiguration:
          DagProcessingLogs:
            LogLevel: !Ref DagProcessingLogs
            Enabled: true
          SchedulerLogs:
            LogLevel: !Ref SchedulerLogsLevel
            Enabled: true
          TaskLogs:
            LogLevel: !Ref TaskLogsLevel
            Enabled: true
          WorkerLogs:
            LogLevel: !Ref WorkerLogsLevel
            Enabled: true
          WebserverLogs:
            LogLevel: !Ref WebserverLogsLevel
            Enabled: true
    SecurityGroup:
      Type: AWS::EC2::SecurityGroup
      Properties:
        VpcId: !Ref VPC
        GroupDescription: !Sub "Security Group for Amazon MWAA Environment ${AWS::StackName}-MwaaEnvironment"
        GroupName: !Sub "airflow-security-group-${AWS::StackName}-MwaaEnvironment"
    
    SecurityGroupIngress:
      Type: AWS::EC2::SecurityGroupIngress
      Properties:
        GroupId: !Ref SecurityGroup
        IpProtocol: "-1"
        SourceSecurityGroupId: !Ref SecurityGroup
  
    SecurityGroupEgress:
      Type: AWS::EC2::SecurityGroupEgress
      Properties:
        GroupId: !Ref SecurityGroup
        IpProtocol: "-1"
        CidrIp: "0.0.0.0/0"
  
    MwaaExecutionRole:
      Type: AWS::IAM::Role
      Properties:
        AssumeRolePolicyDocument:
          Version: 2012-10-17
          Statement:
            - Effect: Allow
              Principal:
                Service:
                  - airflow-env.amazonaws.com
                  - airflow.amazonaws.com
              Action:
               - "sts:AssumeRole"
        Path: "/service-role/"
  
    MwaaExecutionPolicy:
      DependsOn: EnvironmentBucket
      Type: AWS::IAM::ManagedPolicy
      Properties:
        Roles:
          - !Ref MwaaExecutionRole
        PolicyDocument:
          Version: 2012-10-17
          Statement:
            - Effect: Allow
              Action: airflow:PublishMetrics
              Resource:
                - !Sub "arn:aws:airflow:${AWS::Region}:${AWS::AccountId}:environment/${EnvironmentName}"
            - Effect: Deny
              Action: s3:ListAllMyBuckets
              Resource:
                - !Sub "${EnvironmentBucket.Arn}"
                - !Sub "${EnvironmentBucket.Arn}/*"
  
            - Effect: Allow
              Action:
                - "s3:GetObject*"
                - "s3:GetBucket*"
                - "s3:List*"
              Resource:
                - !Sub "${EnvironmentBucket.Arn}"
                - !Sub "${EnvironmentBucket.Arn}/*"
            - Effect: Allow
              Action:
                - logs:DescribeLogGroups
              Resource: "*"
  
            - Effect: Allow
              Action:
                - logs:CreateLogStream
                - logs:CreateLogGroup
                - logs:PutLogEvents
                - logs:GetLogEvents
                - logs:GetLogRecord
                - logs:GetLogGroupFields
                - logs:GetQueryResults
                - logs:DescribeLogGroups
              Resource:
                - !Sub "arn:aws:logs:${AWS::Region}:${AWS::AccountId}:log-group:airflow-${AWS::StackName}*"
            - Effect: Allow
              Action: cloudwatch:PutMetricData
              Resource: "*"
            - Effect: Allow
              Action:
                - sqs:ChangeMessageVisibility
                - sqs:DeleteMessage
                - sqs:GetQueueAttributes
                - sqs:GetQueueUrl
                - sqs:ReceiveMessage
                - sqs:SendMessage
              Resource:
                - !Sub "arn:aws:sqs:${AWS::Region}:*:airflow-celery-*"
            - Effect: Allow
              Action:
                - kms:Decrypt
                - kms:DescribeKey
                - "kms:GenerateDataKey*"
                - kms:Encrypt
              NotResource: !Sub "arn:aws:kms:*:${AWS::AccountId}:key/*"
              Condition:
                StringLike:
                  "kms:ViaService":
                    - !Sub "sqs.${AWS::Region}.amazonaws.com"
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
  
    MwaaApacheAirflowUI:
      Description: MWAA Environment
      Value: !Sub  "https://${MwaaEnvironment.WebserverUrl}"
  ```

## Step two: Create the stack using the AWS CLI<a name="quick-start-createstack"></a>

1. In your command prompt, navigate to the directory where `mwaa_public_network.yml` is stored\. For example:

   ```
   cd mwaaproject
   ```

1. Use the [aws cloudformation create\-stack](https://docs.aws.amazon.com/cli/latest/reference/cloudformation/create-stack.html) command to create the stack using the AWS CLI\.

   ```
   aws cloudformation create-stack --stack-name mwaa-environment-public-network --template-body file://mwaa_public_network.yml --capabilities CAPABILITY_IAM
   ```
**Note**  
It takes over 30 minutes to create the Amazon VPC infrastructure, Amazon S3 bucket, and Amazon MWAA environment\.

## Step three: Upload a DAG to Amazon S3 and run in the Apache Airflow UI<a name="quick-start-upload-dag"></a>

1. Copy the contents of the `tutorial.py` file for [Apache Airflow v1\.10\.12](https://airflow.apache.org/docs/apache-airflow/1.10.12/tutorial.html#example-pipeline-definition) and save locally as `tutorial.py`\.

1. In your command prompt, navigate to the directory where `tutorial.py` is stored\. For example:

   ```
   cd mwaaproject
   ```

1. Use the following command to list all of your Amazon S3 buckets\.

   ```
   aws s3 ls
   ```

1. Use the following command to list the files and folders in the Amazon S3 bucket for your environment\.

   ```
   aws s3 ls s3://YOUR_S3_BUCKET_NAME
   ```

1. Use the following script to upload the `tutorial.py` file to your `dags` folder\. Substitute the sample value in *YOUR\_S3\_BUCKET\_NAME*\.

   ```
   aws s3 cp tutorial.py s3://YOUR_S3_BUCKET_NAME/dags/
   ```

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose **Open Airflow UI**\.

1. Choose the **tutorial** DAG\.

1. Choose **Trigger DAG**\.

## Step four: View logs in CloudWatch Logs<a name="quick-start-logs"></a>

You can view Apache Airflow logs in the CloudWatch console for all of the Apache Airflow logs which were enabled by the AWS CloudFormation stack\. The following section shows how to view logs for the *Airflow web server log group*\.

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose the **Airflow web server log group** on the **Monitoring** pane\.

1. Choose the `webserver_console_ip` log in **Log streams**\.

## Step five: Create IAM policies for an Apache Airflow development team<a name="quick-start-create-group"></a>

Let's say you have an Apache Airflow development team of ten developers and you want to create a group in IAM named `AirflowDevelopmentGroup` to apply permissions to all of your developers\. These users need access to the `AmazonMWAAFullConsoleAccess`, `AmazonMWAAAirflowCliAccess`, and `AmazonMWAAWebServerAccess` permission policies to [access an Amazon MWAA environment and Apache Airflow UI](access-policies.md)\. This section describes how to create a group in IAM, create and attach these policies, and associate the group to an IAM user\. 

**To create the AmazonMWAAFullConsoleAccess policy**

1. Download the [AmazonMWAAFullConsoleAccess access policy](./samples/AmazonMWAAFullConsoleAccess.zip)\.

1. Open the [Policies page](https://console.aws.amazon.com/iam/home#/policies) on the IAM console\.

1. Choose **Create policy**\.

1. Choose the **JSON** tab\.

1. Paste the JSON policy for `AmazonMWAAFullConsoleAccess`\.

1. Substitute the following values:

   1. *\{your\-account\-id\}* – your AWS account ID \(such as `0123456789`\)

   1. *\{your\-kms\-id\}* – the `aws/airflow` identifer for an [AWS owned CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk)

1. Choose the **Review policy**\.

1. Type `AmazonMWAAFullConsoleAccess` in **Name**\.

1. Choose **Create policy**\.

**To create the AmazonMWAAWebServerAccess policy**

1. Download the [AmazonMWAAWebServerAccess access policy](./samples/AmazonMWAAWebServerAccess.zip)\.

1. Open the [Policies page](https://console.aws.amazon.com/iam/home#/policies) on the IAM console\.

1. Choose **Create policy**\.

1. Choose the **JSON** tab\.

1. Paste the JSON policy for `AmazonMWAAWebServerAccess`\.

1. Substitute the following values:

   1. *\{your\-region\}* – the region of your Amazon MWAA environment \(such as `us-east-1`\)

   1. *\{your\-account\-id\}* – your AWS account ID \(such as `0123456789`\)

   1. *\{your\-environment\-name\}* – your Amazon MWAA environment name \(such as `MyAirflowEnvironment`\)

   1. *\{airflow\-role\}* – the `Admin` Apache Airflow [Default Role](https://airflow.apache.org/docs/apache-airflow/1.10.6/security.html?highlight=ldap#default-roles)

1. Choose **Review policy**\.

1. Type `AmazonMWAAWebServerAccess` in **Name**\.

1. Choose **Create policy**\.

**To create the AmazonMWAAAirflowCliAccess policy**

1. Download the [AmazonMWAAAirflowCliAccess access policy](./samples/AmazonMWAAAirflowCliAccess.zip)\.

1. Open the [Policies page](https://console.aws.amazon.com/iam/home#/policies) on the IAM console\.

1. Choose **Create policy**\.

1. Choose the **JSON** tab\.

1. Paste the JSON policy for `AmazonMWAAAirflowCliAccess`\.

1. Choose the **Review policy**\.

1. Type `AmazonMWAAAirflowCliAccess` in **Name**\.

1. Choose **Create policy**\.

**To create the group**

1. Open the [Groups page](https://console.aws.amazon.com/iam/home#/groups) on the IAM console\.

1. Type a name of `AirflowDevelopmentGroup`\.

1. Choose **Next Step**\.

1. Type `AmazonMWAA` to filter results in **Filter**\.

1. Select the three policies you created\.

1. Choose **Next Step**\.

1. Choose **Create Group**\.

**To associate to a user**

1. Open the [Users page](https://console.aws.amazon.com/iam/home#/users) on the IAM console\.

1. Choose a user\.

1. Choose **Groups**\.

1. Choose **Add user to groups**\.

1. Select the **AirflowDevelopmentGroup**\.

1. Choose **Add to Groups**\.

## What's next?<a name="quick-start-next-up"></a>
+ Learn about best practices when specifying Python dependencies in [Managing Python dependencies in requirements\.txt](best-practices-dependencies.md)\.
+ Run some of the DAG code samples in [Code examples for Amazon Managed Workflows for Apache Airflow \(MWAA\)](sample-code.md)\.
+ Learn more about how to specify Python dependencies in a `requirements.txt` and custom plugins in `plugins.zip` in [Working with DAGs on Amazon MWAA](working-dags.md)\.