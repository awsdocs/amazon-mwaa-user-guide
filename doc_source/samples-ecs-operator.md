# Connecting to Amazon ECS using the `ECSOperator`<a name="samples-ecs-operator"></a>

 The topic describes how you can use the `ECSOperator` to connect to an Amazon Elastic Container Service \(Amazon ECS\) container from Amazon MWAA\. In the following steps, you'll add the required permissions to your environment's execution role, use a AWS CloudFormation template to create an Amazon ECS Fargate cluster, and finally create and upload a DAG that connects to your new cluster\. 

**Topics**
+ [Version](#samples-ecs-operator-version)
+ [Prerequisites](#samples-ecs-operator-prereqs)
+ [Permissions](#samples-ecs-operator-permissions)
+ [Create an Amazon ECS cluster](#create-cfn-template)
+ [Code sample](#samples-ecs-operator-code)

## Version<a name="samples-ecs-operator-version"></a>
+ You can use the code example on this page with **Apache Airflow v2 and above** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="samples-ecs-operator-prereqs"></a>

To use the sample code on this page, you'll need the following:
+ An [Amazon MWAA environment](get-started.md)\.

## Permissions<a name="samples-ecs-operator-permissions"></a>
+ The execution role for your environment needs permission to run tasks in Amazon ECS\. You can either attach the [AmazonECS\_FullAccess](https://console.aws.amazon.com/iam/home#policies/arn:aws:iam::aws:policy/AmazonECS_FullAccess$jsonEditor) AWS\-managed policy to your execution role, or create and attach the following policy to your execution role\.

  ```
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Sid": "VisualEditor0",
              "Effect": "Allow",
              "Action": [
                  "ecs:RunTask",
                  "ecs:DescribeTasks"
              ],
              "Resource": "*"
          },
          {
              "Action": "iam:PassRole",
              "Effect": "Allow",
              "Resource": [
                  "*"
              ],
              "Condition": {
                  "StringLike": {
                      "iam:PassedToService": "ecs-tasks.amazonaws.com"
                  }
              }
          }
      ]
  }
  ```
+  In addition to adding the required premissions to run tasks in Amazon ECS, you must also modify the CloudWatch Logs policy statement in your Amazon MWAA execution role to allow access to the Amazon ECS task log group as shown in the following\. The Amazon ECS log group is created by the AWS CloudFormation template in [Create an Amazon ECS cluster](#create-cfn-template)\. 

  ```
  {
      "Effect": "Allow",
      "Action": [
          "logs:CreateLogStream",
          "logs:CreateLogGroup",
          "logs:PutLogEvents",
          "logs:GetLogEvents",
          "logs:GetLogRecord",
          "logs:GetLogGroupFields",
          "logs:GetQueryResults"
      ],
      "Resource": [
          "arn:aws:logs:region:account-id:log-group:airflow-environment-name-*",
          "arn:aws:logs:*:*:log-group:ecs-mwaa-group:*"
      ]
  }
  ```

 For more information about the Amazon MWAA execution role, and how to attach a policy, see [Execution role](mwaa-create-role.md)\. 

## Create an Amazon ECS cluster<a name="create-cfn-template"></a>

 Using the following AWS CloudFormation template, you will build an Amazon ECS Fargate cluster to use with your Amazon MWAA workflow\. For more information, see [Creating a task definition](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-task-definition) in the *Amazon Elastic Container Service Developer Guide*\. 

1. Create a JSON file with the following code and save it as `ecs-mwaa-cfn.json`\.

   ```
   {
       "AWSTemplateFormatVersion": "2010-09-09",
       "Description": "This template deploys an ECS Fargate cluster with an Amazon Linux image as a test for MWAA.",
       "Parameters": {
           "VpcId": {
               "Type": "AWS::EC2::VPC::Id",
               "Description": "Select a VPC that allows instances access to ECR, as used with MWAA."
           },
           "SubnetIds": {
               "Type": "List<AWS::EC2::Subnet::Id>",
               "Description": "Select at two private subnets in your selected VPC, as used with MWAA."
           },
           "SecurityGroups": {
               "Type": "List<AWS::EC2::SecurityGroup::Id>",
               "Description": "Select at least one security group in your selected VPC, as used with MWAA."
           }
       },
       "Resources": {
           "Cluster": {
               "Type": "AWS::ECS::Cluster",
               "Properties": {
                   "ClusterName": {
                       "Fn::Sub": "${AWS::StackName}-cluster"
                   }
               }
           },
           "LogGroup": {
               "Type": "AWS::Logs::LogGroup",
               "Properties": {
                   "LogGroupName": {
                       "Ref": "AWS::StackName"
                   },
                   "RetentionInDays": 30
               }
           },
           "ExecutionRole": {
               "Type": "AWS::IAM::Role",
               "Properties": {
                   "AssumeRolePolicyDocument": {
                       "Statement": [
                           {
                               "Effect": "Allow",
                               "Principal": {
                                   "Service": "ecs-tasks.amazonaws.com"
                               },
                               "Action": "sts:AssumeRole"
                           }
                       ]
                   },
                   "ManagedPolicyArns": [
                       "arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"
                   ]
               }
           },
           "TaskDefinition": {
               "Type": "AWS::ECS::TaskDefinition",
               "Properties": {
                   "Family": {
                       "Fn::Sub": "${AWS::StackName}-task"
                   },
                   "Cpu": 2048,
                   "Memory": 4096,
                   "NetworkMode": "awsvpc",
                   "ExecutionRoleArn": {
                       "Ref": "ExecutionRole"
                   },
                   "ContainerDefinitions": [
                       {
                           "Name": {
                               "Fn::Sub": "${AWS::StackName}-container"
                           },
                           "Image": "137112412989.dkr.ecr.us-east-1.amazonaws.com/amazonlinux:latest",
                           "PortMappings": [
                               {
                                   "Protocol": "tcp",
                                   "ContainerPort": 8080,
                                   "HostPort": 8080
                               }
                           ],
                           "LogConfiguration": {
                               "LogDriver": "awslogs",
                               "Options": {
                                   "awslogs-region": {
                                       "Ref": "AWS::Region"
                                   },
                                   "awslogs-group": {
                                       "Ref": "LogGroup"
                                   },
                                   "awslogs-stream-prefix": "ecs"
                               }
                           }
                       }
                   ],
                   "RequiresCompatibilities": [
                       "FARGATE"
                   ]
               }
           },
           "Service": {
               "Type": "AWS::ECS::Service",
               "Properties": {
                   "ServiceName": {
                       "Fn::Sub": "${AWS::StackName}-service"
                   },
                   "Cluster": {
                       "Ref": "Cluster"
                   },
                   "TaskDefinition": {
                       "Ref": "TaskDefinition"
                   },
                   "DesiredCount": 1,
                   "LaunchType": "FARGATE",
                   "PlatformVersion": "1.3.0",
                   "NetworkConfiguration": {
                       "AwsvpcConfiguration": {
                           "AssignPublicIp": "ENABLED",
                           "Subnets": {
                               "Ref": "SubnetIds"
                           },
                           "SecurityGroups": {
                               "Ref": "SecurityGroups"
                           }
                       }
                   }
               }
           }
       }
   }
   ```

1.  In your command prompt, use the following AWS CLI command to create a new stack\. You must replace the values `SecurityGroups` and `SubnetIds` with values for your Amazon MWAA environment's security groups and subnets\. 

   ```
   $ aws cloudformation create-stack \
   --stack-name my-ecs-stack --template-body file://ecs-mwaa-cfn.json \
   --parameters ParameterKey=SecurityGroups,ParameterValue=your-mwaa-security-group \
   ParameterKey=SubnetIds,ParameterValue=your-mwaa-subnet-1\\,your-mwaa-subnet-1 \
   --capabilities CAPABILITY_IAM
   ```

    Alternatively, you can use the following shell script\. The script retrieves the required values for your environment's security groups, and subnets using the `[get\-environment](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/mwaa/get-environment.html)` AWS CLI command, then creates the stack accordingly\. To run the script, do the following\. 

   1.  Copy, and save the script as `ecs-stack-helper.sh` in the same directory as your AWS CloudFormation template\. 

      ```
      #!/bin/bash
      
      joinByString() {
        local separator="$1"
        shift
        local first="$1"
        shift
        printf "%s" "$first" "${@/#/$separator}"
      }
      
      response=$(aws mwaa get-environment --name $1)
      
      securityGroupId=$(echo "$response" | jq -r '.Environment.NetworkConfiguration.SecurityGroupIds[]')
      subnetIds=$(joinByString '\,' $(echo "$response" | jq -r '.Environment.NetworkConfiguration.SubnetIds[]'))
      
      aws cloudformation create-stack --stack-name $2 --template-body file://ecs-cfn.json \
      --parameters ParameterKey=SecurityGroups,ParameterValue=$securityGroupId \
      ParameterKey=SubnetIds,ParameterValue=$subnetIds \
      --capabilities CAPABILITY_IAM
      ```

   1.  Run the script using the following commands\. Replace `environment-name` and `stack-name` with your information\. 

      ```
      $ chmod +x ecs-stack-helper.sh
      $ ./ecs-stack-helper.bash environment-name stack-name
      ```

    If successful, you'll see the following output displaying your new AWS CloudFormation stack ID\. 

   ```
   {
       "StackId": "arn:aws:cloudformation:us-west-2:123456789012:stack/my-ecs-stack/123456e7-8ab9-01cd-b2fb-36cce63786c9"
   }
   ```

 After your AWS CloudFormation stack is completed and AWS has provisioned your Amazon ECS resources, you're ready to create and upload your DAG\. 

## Code sample<a name="samples-ecs-operator-code"></a>

1. Open a command prompt, and navigate to the directory where your DAG code is stored\. For example:

   ```
   cd dags
   ```

1. Copy the contents of the following code sample and save locally as `mwaa-ecs-operator.py`, then upload your new DAG to Amazon S3\.

   ```
   from http import client
   from airflow import DAG
   from airflow.providers.amazon.aws.operators.ecs import ECSOperator
   from airflow.utils.dates import days_ago
   import boto3
   
   CLUSTER_NAME="mwaa-ecs-test-cluster" #Replace value for CLUSTER_NAME with your information.
   CONTAINER_NAME="mwaa-ecs-test-container" #Replace value for CONTAINER_NAME with your information.
   LAUNCH_TYPE="FARGATE"
   
   with DAG(
       dag_id = "ecs_fargate_dag",
       schedule_interval=None,
       catchup=False,
       start_date=days_ago(1)
   ) as dag:
       client=boto3.client('ecs')
       services=client.list_services(cluster=CLUSTER_NAME,launchType=LAUNCH_TYPE)
       service=client.describe_services(cluster=CLUSTER_NAME,services=services['serviceArns'])
   
       ecs_operator_task = ECSOperator(
           task_id = "ecs_operator_task",
           dag=dag,
           cluster=CLUSTER_NAME,
           task_definition=service['services'][0]['taskDefinition'],
           launch_type=LAUNCH_TYPE,
           overrides={
               "containerOverrides":[
                   {
                       "name":CONTAINER_NAME,
                       "command":["ls", "-l", "/"],
                   },
               ],
           },
   
           network_configuration=service['services'][0]['networkConfiguration'],
           awslogs_group="mwaa-ecs-zero",
           awslogs_stream_prefix=f"ecs/{CONTAINER_NAME}",
       )
   ```
**Note**  
 In the example DAG, for `awslogs_group`, you might need to modify the log group with the name for your Amazon ECS task log group\. The example assumes a log group named `mwaa-ecs-zero`\. For `awslogs_stream_prefix`, use the Amazon ECS task log stream prefix\. The example assumes a log stream prefix, `ecs`\. 

1.  Run the following AWS CLI command to copy the DAG to your environment's bucket, then trigger the DAG using the Apache Airflow UI\. 

   ```
   $ aws s3 cp your-dag.py s3://your-environment-bucket/dags/
   ```

1. If successful, you'll see output similar to the following in the task logs for `ecs_operator_task` in the `ecs_fargate_dag` DAG:

   ```
   [2022-01-01, 12:00:00 UTC] {{ecs.py:300}} INFO - Running ECS Task -
   Task definition: arn:aws:ecs:us-west-2:123456789012:task-definition/mwaa-ecs-test-task:1 - on cluster mwaa-ecs-test-cluster
   [2022-01-01, 12:00:00 UTC] {{ecs-operator-test.py:302}} INFO - ECSOperator overrides:
   {'containerOverrides': [{'name': 'mwaa-ecs-test-container', 'command': ['ls', '-l', '/']}]}
   .
   .
   .
   [2022-01-01, 12:00:00 UTC] {{ecs.py:379}} INFO - ECS task ID is: e012340b5e1b43c6a757cf012c635935
   [2022-01-01, 12:00:00 UTC] {{ecs.py:313}} INFO - Starting ECS Task Log Fetcher
   [2022-01-01, 12:00:00 UTC] {{ecs.py:119}} INFO - [2022-07-19, 17:54:03 UTC] total 52
   [2022-01-01, 12:00:00 UTC] {{ecs.py:119}} INFO - [2022-07-19, 17:54:03 UTC] lrwxrwxrwx   1 root root    7 Jun 13 18:51 bin -> usr/bin
   [2022-01-01, 12:00:00 UTC] {{ecs.py:119}} INFO - [2022-07-19, 17:54:03 UTC] dr-xr-xr-x   2 root root 4096 Apr  9  2019 boot
   [2022-01-01, 12:00:00 UTC] {{ecs.py:119}} INFO - [2022-07-19, 17:54:03 UTC] drwxr-xr-x   5 root root  340 Jul 19 17:54 dev
   [2022-01-01, 12:00:00 UTC] {{ecs.py:119}} INFO - [2022-07-19, 17:54:03 UTC] drwxr-xr-x   1 root root 4096 Jul 19 17:54 etc
   [2022-01-01, 12:00:00 UTC] {{ecs.py:119}} INFO - [2022-07-19, 17:54:03 UTC] drwxr-xr-x   2 root root 4096 Apr  9  2019 home
   [2022-01-01, 12:00:00 UTC] {{ecs.py:119}} INFO - [2022-07-19, 17:54:03 UTC] lrwxrwxrwx   1 root root    7 Jun 13 18:51 lib -> usr/lib
   [2022-01-01, 12:00:00 UTC] {{ecs.py:119}} INFO - [2022-07-19, 17:54:03 UTC] lrwxrwxrwx   1 root root    9 Jun 13 18:51 lib64 -> usr/lib64
   [2022-01-01, 12:00:00 UTC] {{ecs.py:119}} INFO - [2022-07-19, 17:54:03 UTC] drwxr-xr-x   2 root root 4096 Jun 13 18:51 local
   [2022-01-01, 12:00:00 UTC] {{ecs.py:119}} INFO - [2022-07-19, 17:54:03 UTC] drwxr-xr-x   2 root root 4096 Apr  9  2019 media
   [2022-01-01, 12:00:00 UTC] {{ecs.py:119}} INFO - [2022-07-19, 17:54:03 UTC] drwxr-xr-x   2 root root 4096 Apr  9  2019 mnt
   [2022-01-01, 12:00:00 UTC] {{ecs.py:119}} INFO - [2022-07-19, 17:54:03 UTC] drwxr-xr-x   2 root root 4096 Apr  9  2019 opt
   [2022-01-01, 12:00:00 UTC] {{ecs.py:119}} INFO - [2022-07-19, 17:54:03 UTC] dr-xr-xr-x 103 root root    0 Jul 19 17:54 proc
   [2022-01-01, 12:00:00 UTC] {{ecs.py:119}} INFO - [2022-07-19, 17:54:03 UTC] dr-xr-x-\-\-   2 root root 4096 Apr  9  2019 root
   [2022-01-01, 12:00:00 UTC] {{ecs.py:119}} INFO - [2022-07-19, 17:54:03 UTC] drwxr-xr-x   2 root root 4096 Jun 13 18:52 run
   [2022-01-01, 12:00:00 UTC] {{ecs.py:119}} INFO - [2022-07-19, 17:54:03 UTC] lrwxrwxrwx   1 root root    8 Jun 13 18:51 sbin -> usr/sbin
   [2022-01-01, 12:00:00 UTC] {{ecs.py:119}} INFO - [2022-07-19, 17:54:03 UTC] drwxr-xr-x   2 root root 4096 Apr  9  2019 srv
   [2022-01-01, 12:00:00 UTC] {{ecs.py:119}} INFO - [2022-07-19, 17:54:03 UTC] dr-xr-xr-x  13 root root    0 Jul 19 17:54 sys
   [2022-01-01, 12:00:00 UTC] {{ecs.py:119}} INFO - [2022-07-19, 17:54:03 UTC] drwxrwxrwt   2 root root 4096 Jun 13 18:51 tmp
   [2022-01-01, 12:00:00 UTC] {{ecs.py:119}} INFO - [2022-07-19, 17:54:03 UTC] drwxr-xr-x  13 root root 4096 Jun 13 18:51 usr
   [2022-01-01, 12:00:00 UTC] {{ecs.py:119}} INFO - [2022-07-19, 17:54:03 UTC] drwxr-xr-x  18 root root 4096 Jun 13 18:52 var
   .
   .
   .
   [2022-01-01, 12:00:00 UTC] {{ecs.py:328}} INFO - ECS Task has been successfully executed
   ```