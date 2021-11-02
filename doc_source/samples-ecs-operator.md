# Amazon ECS operator on Amazon MWAA<a name="samples-ecs-operator"></a>

The following sample code uses an Amazon Elastic Container Service \(Amazon ECS\) operator with a Docker container image in a [Amazon ECS private repository](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/private-auth.html)\. You can also use a Docker container image in a private repository, such as JFrog's Artifactory, then use the Amazon ECS operator with Fargate if you don't want to manage the underlying Amazon EC2 instances\.

**Topics**
+ [Version](#samples-ecs-operator-version)
+ [Prerequisites](#samples-ecs-operator-prereqs)
+ [Permissions](#samples-ecs-operator-permissions)
+ [Code sample](#samples-ecs-operator-code)

## Version<a name="samples-ecs-operator-version"></a>
+ The sample code on this page can be used with **Apache Airflow v2\.0\.2** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

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
+  In addition to adding the required premissions to run tasks in Amazon ECS, you must also modify the CloudWatch Logs policy statement in your Amazon MWAA execution role to allow access to the Amazon ECS task log group\. For example, given an Amazon ECS task log group named `hello-world`, you must add the log group ARN, `arn:aws:logs:*:*:log-group:/aws/ecs/hello-world:log-stream:/airflow/*`, to the execution role permission policy as shown in the following example\. 

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
                  "arn:aws:logs:{{region}}:{{accountId}}:log-group:airflow-{{envName}}-*",
                  "arn:aws:logs:*:*:log-group:/aws/ecs/hello-world:log-stream:/airflow/*"
              ]
          }
  ```

 For more information about the Amazon MWAA execution role, and how to attach a policy, see [Execution role](mwaa-create-role.md)\. 

## Code sample<a name="samples-ecs-operator-code"></a>

1.  Create an artifactory private repository with hello\-world docker image\. For more information about creating an artifactory private repository, see [Getting Started with Artifactory as a Docker Registry](https://www.jfrog.com/confluence/display/JFROG/Getting+Started+with+Artifactory+as+a+Docker+Registry) on the JFrog website\. 

1.  Create a Secrets Manager secret for [authenticating to your private repository](http://aws.amazon.com/blogs/compute/introducing-private-registry-authentication-support-for-aws-fargate/)\. For more information about private registry authentication, see [Private registry authentication for tasks](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/private-auth)\. 

1.  Create an Amazon ECS cluster named `democluster` and a Fargate task definition named `hello-world` For more information, see [Creating a task definition](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-task-definition)\. When creating the task definition, use the Secrets Manager ARN to point to your private repository\. You can use the following AWS CloudFormation template to create a cluster and task definition\. 

   ```
   AWSTemplateFormatVersion: "2010-09-09"
     Description:  This template deploys an ECS Fargate cluster.
     Parameters:
       VpcId:
         Type: AWS::EC2::VPC::Id
         Description: Select your VPC. If you are using the VPC from the CD tutorial, select the mwaa-cicd VPC.
       SubnetIds:
         Type: List<AWS::EC2::Subnet::Id>
         Description: Select your subnet. If you are using the subnet from the CD tutorial, select mwaa-cicd-Public Subnet (AZ1).
       SecurityGroups:
         Type: List<AWS::EC2::SecurityGroup::Id>
         Description: Select your security group. If you are using the security group from the CD tutorial, select the mwaa-cicd security group.
     Resources:
       Cluster:
         Type: AWS::ECS::Cluster
         Properties:
           ClusterName: !Sub "${AWS::StackName}"
   
       LogGroup:
         Type: AWS::Logs::LogGroup
         Properties:
           LogGroupName: !Ref AWS::StackName
           RetentionInDays: 30
   
       ExecutionRole:
         Type: AWS::IAM::Role
         Properties:
           AssumeRolePolicyDocument:
             Statement:
               - Effect: Allow
                 Principal:
                   Service: ecs-tasks.amazonaws.com
                 Action: sts:AssumeRole
           ManagedPolicyArns:
             - arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy
   
       TaskDefinition:
         Type: AWS::ECS::TaskDefinition
         Properties:
           Family: !Sub "${AWS::StackName}"
           Cpu: 2048
           Memory: 4096
           NetworkMode: awsvpc
           ExecutionRoleArn: !Ref ExecutionRole
           ContainerDefinitions:
             - Name: !Sub "${AWS::StackName}"
               Image: YOUR_IMAGE_URL
               PortMappings:
                 - Protocol: tcp
                   ContainerPort: YOUR_PORT
                   HostPort: YOUR_PORT
               LogConfiguration:
                 LogDriver: awslogs
                 Options:
                   awslogs-region: !Ref AWS::Region
                   awslogs-group: !Ref LogGroup
                   awslogs-stream-prefix: ecs
           RequiresCompatibilities:
             - FARGATE
   
       Service:
         Type: AWS::ECS::Service
         Properties:
           ServiceName: !Sub "${AWS::StackName}"
           Cluster: !Ref Cluster
           TaskDefinition: !Ref TaskDefinition
           DesiredCount: 1
           LaunchType: FARGATE
           PlatformVersion: 1.3.0
           NetworkConfiguration:
             AwsvpcConfiguration:
               AssignPublicIp: ENABLED
               Subnets: !Ref SubnetIds
               SecurityGroups: !Ref SecurityGroups
   ```

1.  Create a DAG and an Amazon ECS operator as shown in the following\. 

   ```
   import datetime
   import os
   from airflow import DAG
   from airflow.contrib.operators.ecs_operator import ECSOperator
   dag = DAG(
       dag_id="ecs_fargate_dag",
       default_args={
           "owner": "airflow",
           "depends_on_past": False,
           "email": ["airflow@example.com"],
           "email_on_failure": False,
           "email_on_retry": False,
       },
       default_view="graph",
       schedule_interval=None,
       start_date=datetime.datetime(2020, 1, 1),
       tags=["example"],
   )
   # generate dag documentation
   dag.doc_md = __doc__
   # [START howto_operator_ecs]
   hello_world = ECSOperator(
       task_id="hello_world",
       dag=dag,
       cluster="c",
       task_definition="hello-world",
       launch_type="FARGATE",
       overrides={
           "containerOverrides": [ ],
       },
       network_configuration={
           'awsvpcConfiguration': {
               'securityGroups': ['sg-xxxx'],
               'subnets': ['subnet-xxxx', 'subnet-yyyy'],
               'assignPublicIp': "ENABLED"
           },
       },
       tags={
           "Customer": "X",
           "Project": "Y",
           "Application": "Z",
           "Version": "0.0.1",
           "Environment": "Development",
       },
       awslogs_group="/aws/ecs/hello-world",
       awslogs_stream_prefix="/ecs/hello-world",  # replace with your container name
   )
   # [END howto_operator_ecs]
   ```
**Note**  
 In the example DAG, for `awslogs_group`, you might need to modify the log group with the name for your Amazon ECS task log group\. The example assumes a log group named `hello-world`\. For `awslogs_stream_prefix`, use the Amazon ECS task log stream prefix, and the name of your container\. The example assumes a log stream prefix, `ecs`, and a container named `hello-world`\. 