# Tutorial: Configuring the aws\-mwaa\-local\-runner in a Continuous Delivery \(CD\) pipeline<a name="tutorials-docker"></a>

This tutorial guides you through the process of building a continuous delivery \(CD\) pipeline in GitHub using Amazon Managed Workflows for Apache Airflow \(MWAA\)'s [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) to test your Apache Airflow code locally\. It uses AWS CloudFormation templates to create the AWS resources you need\. You'll learn how to: 
+ Optionally, create an Amazon VPC to use the AWS resources for this tutorial\.
+ Make a copy of the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) in your GitHub account by forking the repository\.
+ Create, tag, and push the Docker image of the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) to a private Amazon ECR repository\.
+ Create an Amazon Elastic Container Service \(Amazon ECS\) cluster and an AWS Fargate service for the Docker container image of the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner)\.

**Contents**
+ [About the GitHub workflows](#tutorials-docker-about)
+ [Prerequisites](#tutorials-docker-prereqs)
  + [Docker](#tutorials-docker-prereqs-resources)
  + [AWS CLI](#tutorials-docker-prereqs-cli)
+ [Step one: \(Optional\) Create an Amazon VPC](#tutorials-docker-vpc-create)
+ [Step two: Build the Docker image](#tutorials-docker-local-build)
+ [Step three: Create the Docker image registry in Amazon ECR](#tutorials-docker-repo-permissions)
+ [Step four: Authenticate the Docker client](#tutorials-docker-auth-client)
+ [Step five: Push the Docker image](#tutorials-docker-push)
+ [Step six: Create the Amazon ECS cluster](#tutorials-docker-fargate)
+ [Step seven: Push the task definition to GitHub](#tutorials-docker-git-task)
+ [Step eight: Add your AWS and Amazon ECR information to GitHub](#tutorials-docker-git-workflow-secrets)
+ [Step nine: Setup the GitHub starter workflow](#tutorials-docker-git-workflow-starter)
+ [Step ten: Test the GitHub workflow](#tutorials-docker-git-workflow-test)
+ [Clean up](#tutorials-docker-cleanup)
+ [What's next?](#tutorials-docker-next-up)

## About the GitHub workflows<a name="tutorials-docker-about"></a>

In practice, we want our GitHub workflow to build and deploy a new Docker container image for your local runner to Amazon ECR and Amazon ECS only if the tests in GitHub were successful\. The GitHub workflows in this tutorial use the following GitHub actions\.
+ The [actions/checkout](https://github.com/actions/checkout) workflow to checkout the `main` branch of your local runner repository\.
+ The [configure\-aws\-credentials](https://github.com/aws-actions/configure-aws-credentials) workflow to use AWS credential secrets\.
+ The [amazon\-ecr\-login](https://github.com/aws-actions/amazon-ecr-login) workflow to login to and deploy the container image to Amazon ECR\.
+ The [amazon\-ecs\-render\-task\-definition](https://github.com/aws-actions/amazon-ecs-render-task-definition) workflow to insert the container image URI into an Amazon ECS task definition JSON file, creating a new task definition file\.
+ The [amazon\-ecs\-deploy\-task\-definition](https://github.com/aws-actions/amazon-ecs-deploy-task-definition) workflow to register the Amazon ECS task definition and deploy it to the Amazon ECS Fargate service\.

## Prerequisites<a name="tutorials-docker-prereqs"></a>

This section describes the tools and resources required to complete the steps in the tutorial\.

### Docker<a name="tutorials-docker-prereqs-resources"></a>
+ [Docker installed](https://docs.docker.com/get-docker/) for your operating system\.

### AWS CLI<a name="tutorials-docker-prereqs-cli"></a>

The AWS Command Line Interface \(AWS CLI\) is an open source tool that enables you to interact with AWS services using commands in your command\-line shell\. To complete the steps on this page, you need the following:
+ [AWS CLI – Install version 2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)\.
+ [AWS CLI – Quick configuration with `aws configure`](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)\.

## Step one: \(Optional\) Create an Amazon VPC<a name="tutorials-docker-vpc-create"></a>

You need an Amazon VPC with an Internet gateway attached, a private subnet with a NAT gateway attached, and a public subnet with the auto\-assign public IP address configuration assigned\. If you don't want to modify the Amazon VPC you're using for your Amazon MWAA environment, create a new VPC with these components using the AWS CloudFormation template in this section\. 

**Note**  
After you complete this step, keep the AWS CloudFormation console open to copy the details in the **Outputs** tab\.

1. Open the [AWS CloudFormation](https://console.aws.amazon.com/cloudformation/home#) console\.

1. Use the AWS Region selector to select your region\.

1. Choose **Create stack**, **Template is ready**, and then **Upload a template file**\.

1. Use the `mwaa_local_runner_vpc_public.yaml` template to create the stack\. We recommend providing a stack name of **mwaa\-cicd\-vpc**\. You can also [download the template](./samples/mwaa_local_runner_vpc_public.zip)\.

   ```
   Description:  This template deploys a VPC, with a pair of public and private subnets 
     across two Availability Zones (AZ). It deploys an internet gateway, with a default
     route on the public subnets. It deploys a pair of NAT gateways (one in each AZ),
     and default routes for them in the private subnets.
   
   Parameters:
     EnvironmentName:
       Description: A name that is prefixed to resource names
       Type: String
       Default: mwaa-cicd-
   
     VpcCIDR:
       Description: The IP range (CIDR notation) for this VPC
       Type: String
       Default: 10.0.0.0/16
   
     PublicSubnet1CIDR:
       Description: The IP range (CIDR notation) for the public subnet in the first Availability Zone
       Type: String
       Default: 10.0.0.0/24
   
     PrivateSubnet1CIDR:
       Description: The IP range (CIDR notation) for the private subnet in the first Availability Zone
       Type: String
       Default: 10.0.1.0/24
   
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
             Value: !Sub ${EnvironmentName}Public Subnet (AZ1)
   
     PrivateSubnet1:
       Type: AWS::EC2::Subnet
       Properties:
         VpcId: !Ref VPC
         AvailabilityZone: !Select [ 0, !GetAZs  '' ]
         CidrBlock: !Ref PrivateSubnet1CIDR
         MapPublicIpOnLaunch: false
         Tags:
           - Key: Name
             Value: !Sub ${EnvironmentName}Private Subnet (AZ1)
   
     NatGateway1EIP:
       Type: AWS::EC2::EIP
       DependsOn: InternetGatewayAttachment
       Properties:
         Domain: vpc
   
     NatGateway1:
       Type: AWS::EC2::NatGateway
       Properties:
         AllocationId: !GetAtt NatGateway1EIP.AllocationId
         SubnetId: !Ref PublicSubnet1
   
     PublicRouteTable:
       Type: AWS::EC2::RouteTable
       Properties:
         VpcId: !Ref VPC
         Tags:
           - Key: Name
             Value: !Sub ${EnvironmentName}Public Routes
   
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
   
     PrivateRouteTable1:
       Type: AWS::EC2::RouteTable
       Properties:
         VpcId: !Ref VPC
         Tags:
           - Key: Name
             Value: !Sub ${EnvironmentName}Private Routes (AZ1)
   
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
   
     SecurityGroup:
       Type: AWS::EC2::SecurityGroup
       Properties:
         GroupName: "mwaa-cicd"
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
       Value: !Join [ ",", [ !Ref PublicSubnet1 ]]
   
     PrivateSubnets:
       Description: A list of the private subnets
       Value: !Join [ ",", [ !Ref PrivateSubnet1 ]]
   
     PublicSubnet1:
       Description: A reference to the public subnet in the 1st Availability Zone
       Value: !Ref PublicSubnet1
   
     PrivateSubnet1:
       Description: A reference to the private subnet in the 1st Availability Zone
       Value: !Ref PrivateSubnet1
   
     SecurityGroupIngress:
       Description: Security group with a self-referencing inbound rule
       Value: !Ref SecurityGroupIngress
   ```

1. After you've created the Amazon VPC, choose **Outputs**, and copy all of the identifiers for your Amazon VPC\.

## Step two: Build the Docker image<a name="tutorials-docker-local-build"></a>

The following steps show how to fork the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) repository on *GitHub*\. By default, it uses the `main` branch of the local runner, which is the latest version of Apache Airflow\. The latest version available is Apache Airflow v2\.2\.2\. A fork is a copy of a repository\. Forking a repository allows you to freely experiment with and push your DAGs, Python dependencies, and custom plugins to your repository without affecting the original project\. 

1. Open [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner), choose **Fork**\.

1. Open a command prompt and create a project directory locally, and clone the forked repository in this directory\. For example:

   ```
   mkdir mwaa-cicd
   cd mwaa-cicd
   git clone git@github.com:your-git-user-name/aws-mwaa-local-runner.git
   cd aws-mwaa-local-runner
   ```

1. **Optional \- macOS**\. Open the [Docker Desktop](https://www.docker.com/products/docker-desktop) program\.

1. Return to your command prompt window to build the Docker container image within the `aws-mwaa-local-runner` directory\.

   ```
   ./mwaa-local-env build-image
   ```

1. Start the local Apache Airflow environment\.

   ```
   ./mwaa-local-env start
   ```

1. Login to Apache Airflow at [http://localhost:8080/](http://localhost:8080/):

   1. **Username**: admin

   1. **Password**: test

1. After you've forked the repository and confirmed Apache Airflow is running locally, use *Ctrl \+ C* on your keyboard to stop the container\.

## Step three: Create the Docker image registry in Amazon ECR<a name="tutorials-docker-repo-permissions"></a>

You need specific permissions for your user, group, or role in AWS Identity and Access Management \(IAM\) to complete the steps in this tutorial\. This section contains a AWS CloudFormation template that creates the Amazon ECR repository and attaches the minimum permissions needed\. By default, the template gives any user in the AWS account access\. You can substitute the sample ARN for any existing role\(s\), user\(s\) or group\(s\) in IAM that needs access to the AWS resources in your CD pipeline\.

**Note**  
After you complete this step, keep the AWS CloudFormation console open to copy the details in the **Outputs** tab\.

1. Open the [AWS CloudFormation](https://console.aws.amazon.com/cloudformation/home#) console\.

1. Use the AWS Region selector to select your region\.

1. Choose **Create stack**, **Template is ready**, and then **Upload a template file**\.

1. Use the `mwaa_local_runner_ecr_repo.yaml` template to create the stack\. We recommend providing a stack name of **mwaa\-cicd\-ecr**\. You can also [download the template](./samples/mwaa_local_runner_ecr_repo.zip)\.

   ```
   AWSTemplateFormatVersion: '2010-09-09'
   Description: Creates an ECR repository with the permissions needed to push and pull images. It gives any user in the account access.
   Parameters: 
       RepoName:
         Type: String
         Default: mwaa-cicd
         Description: The name of the ECR repository.
   
   Resources:
     Repository:
       Type: AWS::ECR::Repository
       Properties:
         RepositoryName: !Ref RepoName
         RepositoryPolicyText: 
           Version: "2012-10-17"
           Statement: 
             - Sid: AllowPushPull
               Effect: Allow
               Principal: 
                 AWS: 
                   - !Sub "arn:aws:iam::${AWS::AccountId}:root"
               Action: 
                 - "ecr:GetAuthorizationToken"
                 - "ecr:GetRepositoryPolicy"
                 - "ecr:GetDownloadUrlForLayer"
                 - "ecr:BatchGetImage"
                 - "ecr:BatchCheckLayerAvailability"
                 - "ecr:PutImage"
                 - "ecr:ListImages"
                 - "ecr:DescribeImages"
                 - "ecr:DescribeRepositories"
                 - "ecr:InitiateLayerUpload"
                 - "ecr:UploadLayerPart"
                 - "ecr:CompleteLayerUpload"
   Outputs:
     EcrRepoName:
       Value: !Ref RepoName
     
     EcrRepositoryUri:
       Value: !GetAtt Repository.RepositoryUri
   ```

1. After you've created the repository, choose **Outputs**, and copy the Amazon ECR URI in `EcrRepositoryUri`\. You can also use the [describe\-repositories](https://docs.aws.amazon.com/cli/latest/reference/ecr/describe-repositories.html) command in the AWS CLI to retrieve the URI of your repository in `repositoryUri`\.

   ```
   aws ecr describe-repositories --repository-names mwaa-cicd
   ```

## Step four: Authenticate the Docker client<a name="tutorials-docker-auth-client"></a>

The following steps show how to add your Amazon ECR credentials to the Docker CLI\. This retrieves an authentication token and authenticates your Docker client to your Amazon ECR private registry\.

1. In your command prompt, change directories to the root of your local runner directory\. For example:

   ```
   cd mwaa-cicd/aws-mwaa-local-runner
   ```

1. Run the Docker list image command, and note the name of the `amazon/mwaa-local` values in `TAG` and `IMAGE ID`\.

   ```
   docker image ls
   ```

1. Run the [get\-login\-password](https://docs.aws.amazon.com/cli/latest/reference/ecr/get-login-password.html) command to authenticate\. This gives the Docker CLI permission to access your AWS account\. Substitute the value in `region` with the AWS Region you specified in previous steps\. For example:

   ```
   aws ecr get-login-password --region YOUR_REGION | docker login --username AWS --password-stdin YOUR_ACCOUNT_ID.dkr.ecr.YOUR_REGION.amazonaws.com
   ```

   The output should look like the following:

   ```
   Login Succeeded
   ```

## Step five: Push the Docker image<a name="tutorials-docker-push"></a>

This section walks you through the steps to tag and push the Docker image to Amazon ECR\.

**Tag the image**

1. If you're not already there in your command prompt, change directories to the root of your local runner directory\. For example:

   ```
   cd mwaa-cicd/aws-mwaa-local-runner
   ```

1. Run the Docker tag command with the `amazon/mwaa-local:2.0` tag from previous steps with your repository URL\. Substitute the sample value with your URI\. For example:

   ```
   docker image tag amazon/mwaa-local:2.0 YOUR_ACCOUNT_ID.dkr.ecr.YOUR_REGION.amazonaws.com/mwaa-cicd:2.0
   ```

   The response doesn't contain any output\.

1. Verify the Docker tag for your repository\.

   ```
   docker image ls
   ```

   The output should look like the following:

   ```
   REPOSITORY                                                         TAG         IMAGE ID       CREATED             SIZE
   YOUR_ACCOUNT.dkr.ecr.us-east-1.amazonaws.com/mwaa-cicd             2.0         5f6473ce154e   About an hour ago   1.33GB
   amazon/mwaa-local                                                  2.0         5f6473ce154e   About an hour ago   1.33GB
   postgres                                                           10-alpine   17ec9988ae21   2 days ago          72.8MB
   ```

1. Run the Docker push command to push the image to the repository\. Substitute the sample value with your URI\. For example:

   ```
   docker image push YOUR_ACCOUNT_ID.dkr.ecr.YOUR_REGION.amazonaws.com/mwaa-cicd:2.0
   ```

1. The output should look like the following:

   ```
   5f70bf18a086: Pushed 
   299dc11cc9fc: Pushed 
   999fe11097c8: Pushed 
   99a9119976aa: Pushed 
   99c8a1192b0a: Pushed 
   ccccac999f7f: Pushed 
   499481199891: Pushed 
   ca9911963715: Pushed 
   cf299ec9e179: Pushed 
   964c9939e7a2: Pushed 
   3d99d0d9910f: Pushed 
   919bfe99a642: Pushed 
   ae99f399c8e7: Pushed 
   latest: digest: sha256:ee0008e584b009be056ef1c0d46f00438b316111c80009141114e45b115d9a52 size: 3033
   ```

1. Run the [describe\-images](https://docs.aws.amazon.com/cli/latest/reference/ecr/describe-images.html) command to view a registry summary\. You can also access an Amazon ECR repository summary in the [Repositories page](https://console.aws.amazon.com/ecr/repositories#) on the Amazon ECR console\.

   ```
   aws ecr describe-images --repository-name mwaa-cicd
   ```

   The output should look like the following:

   ```
   {
       "imageDetails": [
           {
               "registryId": "YOUR_ACCOUNT_ID",
               "repositoryName": "mwaa-cicd",
               "imageDigest": "sha256:ee0008e584b009be056ef1c0d46f00438b316111c80009141114e45b115d9a52",
               "imageTags": [
                   "latest"
               ],
               "imageSizeInBytes": 504790381,
               "imagePushedAt": "2021-08-26T15:39:43-06:00",
               "imageManifestMediaType": "application/vnd.docker.distribution.manifest.v2+json",
               "artifactMediaType": "application/vnd.docker.container.image.v1+json"
           }
       ]
   }
   ```

## Step six: Create the Amazon ECS cluster<a name="tutorials-docker-fargate"></a>

A task definition is required to run Docker containers in an Amazon ECS cluster\. The following AWS CloudFormation template deploys an Amazon ECS Fargate cluster with the Docker image for the local runner\. It contains a task definition for Fargate and the execution role policy required by Amazon ECS\. 

Although you can select the same Amazon VPC, public subnet, and security group as your Amazon MWAA environment in these steps, the descriptions in this template assume you're using the Amazon VPC components created in previous steps\.

1. Open the [AWS CloudFormation](https://console.aws.amazon.com/cloudformation/home#) console\.

1. Use the AWS Region selector to select your region\.

1. Choose **Create stack**, **Template is ready**, and then **Upload a template file**\.

1. Use the `mwaa_local_runner_ecs_cluster.yaml` template to create the stack\. Substitute the value in `Image` with your Amazon ECR repository URI, including the tag\. We recommend providing a stack name of **mwaa\-cicd\-ecs**\. You can also [download the template](./samples/mwaa_local_runner_ecs_cluster.zip)\.

   ```
     AWSTemplateFormatVersion: "2010-09-09"
     Description:  This template deploys an ECS Fargate cluster with an Amazon Linux image.
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
               Image: YOUR_ACCOUNT_ID.dkr.ecr.YOUR_REGION.amazonaws.com/mwaa-cicd:2.0
               PortMappings:
                 - Protocol: tcp
                   ContainerPort: 8080
                   HostPort: 8080
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

## Step seven: Push the task definition to GitHub<a name="tutorials-docker-git-task"></a>

The following steps show how to add the task definition for your Amazon ECS cluster to your forked GitHub repository\.

**Save the task definition locally**

1. Open the [Amazon ECS](https://console.aws.amazon.com/ecs/home#) console\.

1. Use the AWS Region selector to select your region\.

1. Choose **Task Definitions**, **mwaa\-cicd\-ecs**, and then select your task definition\.

1. Choose the **JSON** tab\.

1. Copy and save locally as `task-def.json`\.

**Push the task definition to GitHub**

1. Copy the task definition to the root of your forked repository\. For example, in terminal on macOS:

   ```
   mv ~/Downloads/task-def.json ~/mwaa-cicd/aws-mwaa-local-runner/task-def.json
   ```

1. Change directories to the root of your local runner directory\. For example:

   ```
   cd mwaa-cicd/aws-mwaa-local-runner
   ```

1. Verify the upstream of your forked repository\.

   ```
   git remote -vv
   ```

   The output should look like the following:

   ```
   origin	git@github.com:your-git-user-name/aws-mwaa-local-runner.git (fetch)
   origin	git@github.com:your-git-user-name/aws-mwaa-local-runner.git (push)
   ```

1. Get the status of your git branch\.

   ```
   git status
   ```

1. Push the task definition to your repository\.

   ```
   git add task-def.json
   git commit -m "Create initial commit"
   git push -u origin main
   ```

## Step eight: Add your AWS and Amazon ECR information to GitHub<a name="tutorials-docker-git-workflow-secrets"></a>

1. Open the forked repository in your browser\. For example, https://github\.com/**your\-git\-user\-name**/aws\-mwaa\-local\-runner/\.

1. Choose **Settings**, **Secrets**, **New repository secret**\.

1. Add the following key\-value pairs:

   1. Add `AWS_ACCESS_KEY_ID` in **Name**, and the value of your access key ID in **Value**, and then choose **Add secret**\.

   1. Add `AWS_SECRET_ACCESS_KEY` in **Name**, and the value of your secret key in **Value**, and then choose **Add secret**\.

   1. *Optional* \- Add `AWS_SESSION_TOKEN` in **Name**, and the value of your session token in **Value**, and then choose **Add secret**\.

   1. Add `ECR_REPOSITORY_URI` in **Name**, and add the value of your Amazon ECR repository URI in **Value**, and then choose **Add secret**, excluding the tag\. For example:

      ```
      YOUR_ACCOUNT.dkr.ecr.us-east-1.amazonaws.com/mwaa-cicd
      ```

## Step nine: Setup the GitHub starter workflow<a name="tutorials-docker-git-workflow-starter"></a>

A workflow is a configurable automated process made up of one or more jobs\. The following steps uses the [amazon\-ecr\-login](http://github.com/aws-actions/amazon-ecr-login) GitHub Actions workflow\. The workflow logs into your Amazon ECR registry to pull and push images to your repository after a file is changed in your forked repository\.

1. Open the forked repository in your browser\. For example, https://github\.com/**your\-git\-user\-name**/aws\-mwaa\-local\-runner/\.

1. Choose **Actions**, and **Set up this workflow** for the **Deploy to Amazon ECS** workflow\.

1. Copy the following template and paste in the text field\. Substitute the value in `aws-region`\. You can also [download the template](./samples/mwaa_local_runner_github.zip)\.

   ```
   name: Deploy to Amazon ECS
   
   on:
    push:
     branches:
      - main
   
   env:
     AWS_REGION: us-east-1               # set this to your AWS region, e.g. us-west-2
     ECR_REPOSITORY: mwaa-cicd           # the Amazon ECR repository name
     ECS_SERVICE: mwaa-cicd-ecs          # the Amazon ECS service name
     ECS_CLUSTER: mwaa-cicd-ecs          # the name of the Amazon ECS cluster name
     ECS_TASK_DEFINITION: task-def.json  # the path to the Amazon ECS task definition
     CONTAINER_NAME: mwaa-cicd-ecs       # the name of the Amazon ECS container
     IMAGE_TAG: '2.0'                    # the Amazon ECR container image tag
     LOCAL_TAG: amazon/mwaa-local
   
   defaults:
     run:
       shell: bash
   
   jobs:
     deploy:
       name: Deploy
       runs-on: ubuntu-latest # the GitHub Action VM Runner to execute the workflow
       permissions:
         packages: write
         contents: read
   
       steps:
         - name: Checkout
           uses: actions/checkout@v2
   
         - name: Configure AWS credentials
           uses: aws-actions/configure-aws-credentials@v1
           with:
             aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
             aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
             # aws-session-token: ${{ secrets.AWS_SESSION_TOKEN }} # add if you use an AWS session token
             aws-region: ${{ env.AWS_REGION }}
   
         - name: Login to Amazon ECR
           id: login-ecr
           uses: aws-actions/amazon-ecr-login@v1
   
         - name: Build, tag, and push image to Amazon ECR
           id: build-image
           env:
             ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
             IMAGE_TAG: ${{ env.IMAGE_TAG }}
             LOCAL_TAG: ${{ env.LOCAL_TAG }}
           run: |
             # Build the docker container and push it to ECR so that it can be deployed to ECS.
             ./mwaa-local-env build-image
             docker tag $LOCAL_TAG:$IMAGE_TAG ${{ secrets.ECR_REPOSITORY_URI }}:$IMAGE_TAG
             docker push ${{ secrets.ECR_REPOSITORY_URI }}:$IMAGE_TAG
             echo "::set-output name=image::${{ secrets.ECR_REPOSITORY_URI }}:$IMAGE_TAG"
         - name: Fill in the new image ID in the Amazon ECS task definition
           id: task-def
           uses: aws-actions/amazon-ecs-render-task-definition@v1
           with:
             task-definition: ${{ env.ECS_TASK_DEFINITION }}
             container-name: ${{ env.CONTAINER_NAME }}
             image: ${{ steps.build-image.outputs.image }}
   
         - name: Deploy Amazon ECS task definition
           uses: aws-actions/amazon-ecs-deploy-task-definition@v1
           with:
             task-definition: ${{ steps.task-def.outputs.task-definition }}
             service: ${{ env.ECS_SERVICE }}
             cluster: ${{ env.ECS_CLUSTER }}
             wait-for-service-stability: true
   ```

1. Choose **Start commit**, and add *Add ECS workflow* in the **Description** field\.

1. Choose **Commit directly to the main branch**, **Commit new file**\.

## Step ten: Test the GitHub workflow<a name="tutorials-docker-git-workflow-test"></a>

To test the workflow, you can push an update for the Amazon ECS task definition or the GitHub workflow file \(in `aws.yaml`\) to your local runner repository in GitHub\. The following steps show how to push an update for the task definition to run the workflow\. 

1. In your command prompt, change directories to the root of your local runner directory\. For example:

   ```
   cd mwaa-cicd/aws-mwaa-local-runner
   ```

1. Pull the latest changes\.

   ```
   git pull
   ```

1. Open the task definition and make a minor change in the `task-def.json` file, such as adding a space after the last bracket \(`}`\)\.

   ```
   open task-def.json
   ```

1. Get the status of the branch\.

   ```
   git status
   ```

   The output should look like the following:

   ```
   modified:   task-def.json
   ```

1. Push the task definition to your repository\.

   ```
   git add task-def.json
   git commit -m "Add modified task definition"
   git push -u origin main
   ```

1. Open the **Actions** tab, **Deploy to Amazon ECS** in your repository\. You should see the workflow running:  
![\[This image shows how to log-in to your Apache Airflow UI.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-cicd-tutorial.png)

## Clean up<a name="tutorials-docker-cleanup"></a>

You can use the following steps to delete any of the AWS resources you created in this tutorial\.

1. Open the [AWS CloudFormation](https://console.aws.amazon.com/cloudformation/home#) console\.

1. Select the stack you want to delete, choose **Delete**, and then **Delete stack**\.

## What's next?<a name="tutorials-docker-next-up"></a>
+ Explore all available *AWS for GitHub Actions* at [https://github\.com/aws\-actions](https://github.com/aws-actions)\.