# Troubleshooting Amazon Managed Workflows for Apache Airflow \(MWAA\)<a name="troubleshooting"></a>

This topic describes common questions and resolutions to some errors and issues you may encounter when using Apache Airflow on Amazon Managed Workflows for Apache Airflow \(MWAA\)\.

**Contents**
+ [Common questions](#t-common-questions)
  + [When should I use AWS Step Functions vs\. Amazon MWAA?](#t-step-functions)
+ [Create bucket](#troubleshooting-create-bucket)
  + [I can't select the option for S3 Block Public Access settings](#t-create-bucket)
+ [Create environment](#troubleshooting-create-environment)
  + [I tried creating an environment and it's stuck in the "Creating" state](#t-stuck-failure)
  + [I tried to create an environment but it shows the status as "Create failed"](#t-create-environ-failed)
  + [I tried to select a VPC and received a "Network Failure" error](#t-network-failure)
  + [I received a service, partition, or resource "must be passed" error](#t-service-partition)
+ [Update environment](#troubleshooting-update-environment)
  + [I tried changing the environment class but the update failed](#t-rollback-billing-failure)
+ [Updating requirements\.txt](#troubleshooting-dependencies)
  + [Adding `apache-airflow-providers-amazon` causes my environment to fail](#t-requirements)
  + [I specified a new version of my `requirements.txt` and it's taking more than 20 minutes to update my environment](#t-requirements)
+ [Access environment](#troubleshooting-access-environment)
  + [I can't access the Apache Airflow UI](#t-no-access-airflow-ui)
+ [Broken DAG](#troubleshooting-broken-dags)
  + [I received a 'Broken DAG' error when using Amazon DynamoDB operators](#missing-boto)
  + [I received 'Broken DAG: No module named psycopg2' error](#missing-postgres-library)
  + [I received a 'Broken DAG' error when using the Slack operators](#missing-slack)
  + [I received various errors installing Google/GCP/BigQuery](#missing-bigquery-cython)
  + [I received 'Broken DAG: No module named Cython' error](#broken-cython)
+ [Operators](#troubleshooting-operators)
  + [I received an error using the BigQuery operator](#bigquery-operator-ui)
+ [Connections](#troubleshooting-connections)
  + [I can't connect to Snowflake](#missing-snowflake)
  + [I can't connect to Secrets Manager](#access-secrets-manager)
  + [I can't connect to my MySQL server on '<DB\-identifier\-name>\.cluster\-id\.<region>\.rds\.amazonaws\.com'](#mysql-server)
+ [Web server](#troubleshooting-web-server)
  + [I'm using the BigQueryOperator and it's causing my web server to crash](#operator-biquery)
  + [I see a 5xx error accessing the web server](#5xx-webserver)
+ [Tasks](#troubleshooting-tasks)
  + [I see my tasks stuck in the running state](#stranded-tasks)
+ [Logs](#troubleshooting-view-logs)
  + [I can’t see my task logs or I received a remote log error in the Airflow UI](#t-task-logs)
  + [I keep seeing a ResourceAlreadyExistsException error in CloudTrail](#t-cloudtrail)
  + [I keep seeing an Invalid request error in CloudTrail](#t-cloudtrail-bucket)

## Common questions<a name="t-common-questions"></a>

### When should I use AWS Step Functions vs\. Amazon MWAA?<a name="t-step-functions"></a>

1. You can use Step Functions to process individual customer orders, since Step Functions can scale to meet demand for one order or one million orders\.

1. If you’re running an overnight workflow that processes the previous day’s orders, you can use Step Functions or Amazon MWAA\. Amazon MWAA allows you an open source option to abstract the workflow from the AWS resources you're using\.

For common questions, see [Amazon MWAA frequently asked questions](mwaa-faqs.md)\.

## Create bucket<a name="troubleshooting-create-bucket"></a>

The following topic describes the errors you may receive when creating an Amazon S3 bucket\.

### I can't select the option for S3 Block Public Access settings<a name="t-create-bucket"></a>

The [execution role](mwaa-create-role.md) for your Amazon MWAA environment needs permission to the `GetBucketPublicAccessBlock` action on the Amazon S3 bucket to verify the bucket blocked public access\. We recommend the following steps:

1. Follow the steps to [Attach a JSON policy to your execution role](mwaa-create-role.md)\. 

1. Attach the following JSON policy:

   ```
   {
      "Effect":"Allow",
      "Action":[
         "s3:GetObject*",
         "s3:GetBucket*",
         "s3:List*"
      ],
      "Resource":[
         "arn:aws:s3:::{your-s3-bucket-name}",
         "arn:aws:s3:::{your-s3-bucket-name}/*"
      ]
   }
   ```

   Substitute the sample placeholders in *\{your\-s3\-bucket\-name\}* with your Amazon S3 bucket name, such as *my\-mwaa\-unique\-s3\-bucket\-name*\.

1. To run a troubleshooting script that checks the Amazon VPC network setup and configuration for your Amazon MWAA environment, see the [Verify Environment](https://github.com/awslabs/aws-support-tools/tree/master/MWAA) script in AWS Support Tools on GitHub\.

## Create environment<a name="troubleshooting-create-environment"></a>

The following topic describes the errors you may receive when creating an environment\.

### I tried creating an environment and it's stuck in the "Creating" state<a name="t-stuck-failure"></a>

We recommend the following steps:

1. If you're using a **Public network**, verify that the subnets route to a [NAT gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html), and that the NAT gateway's subnets route to an [Internet gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)\.

1. To run a troubleshooting script that checks the Amazon VPC network setup and configuration for your Amazon MWAA environment, see the [Verify Environment](https://github.com/awslabs/aws-support-tools/tree/master/MWAA) script in AWS Support Tools on GitHub\.

### I tried to create an environment but it shows the status as "Create failed"<a name="t-create-environ-failed"></a>

We recommend the following steps:

1. Check user permissions\. Amazon MWAA performs a dry run against a user's credentials before creating an environment\. You may not have permission to create some of the resources for an environment\. For example, the **Private network** access mode requires permission to create VPC endpoints \(AWS PrivateLink\), and your AWS account may not have Amazon VPC permissions\. We recommend asking your AWS account administrator for access\.

1. Check execution role permissions\. An execution role is an AWS Identity and Access Management \(IAM\) role with a permissions policy that grants Amazon MWAA permission to invoke the resources of other AWS services \(such as Amazon S3, CloudWatch, Amazon SQS, Amazon ECR\) on your behalf\. Your [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk) or [AWS owned CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk) also needs to be permitted access\. To learn more, see [Execution role](mwaa-create-role.md)\.

1. Check private subnets\. If you're using *public routing* in your Amazon VPC, verify that the two private subnets route to a NAT gateway, and that the NAT gateway's subnets route to an Internet gateway, as defined in [About networking on Amazon MWAA](networking-about.md)\.

1. Check network access\. Verify that each of the AWS resources used by the environment are configured to allow network traffic, as defined in [Security in your VPC](vpc-security.md)\.

1. Check Apache Airflow logs\. If you enabled Apache Airflow logs, verify your log groups were created successfully on the [Logs groups page](https://console.aws.amazon.com/cloudwatch/home#logsV2:log-groups) on the CloudWatch console\. If you see blank logs, the most common reason is due to missing permissions in your execution role for CloudWatch or Amazon S3 where logs are written\. To learn more, see [Execution role](mwaa-create-role.md)\.

1. To run a troubleshooting script that checks the Amazon VPC network setup and configuration for your Amazon MWAA environment, see the [Verify Environment](https://github.com/awslabs/aws-support-tools/tree/master/MWAA) script in AWS Support Tools on GitHub\.

### I tried to select a VPC and received a "Network Failure" error<a name="t-network-failure"></a>

We recommend the following steps:
+ If you see a "Network Failure" error when you try to select a VPC when creating your environment, turn off any in\-browser proxies that are running, and then try again\.

### I received a service, partition, or resource "must be passed" error<a name="t-service-partition"></a>

We recommend the following steps:
+ This may be because the S3 URI you entered for the S3 bucket for your environment includes a '/' at the end of the URI\. To resolve this issue, remove the '/' in the S3 bucket path\. The value should be an S3 URI in the following format:

  ```
  s3://your-bucket-name
  ```

## Update environment<a name="troubleshooting-update-environment"></a>

The following topic describes the errors you may receive when updating an environment\.

### I tried changing the environment class but the update failed<a name="t-rollback-billing-failure"></a>

If you update your environment to a different environment class \(such as changing an `mw1.medium` to an `mw1.small`\), and the request to update your environment failed, the environment status goes into an `UPDATE_FAILED` state and the environment is rolled back to, and is billed according to, the previous stable version of an environment\.

We recommend the following steps:

1. Test your Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

1. To run a troubleshooting script that checks the Amazon VPC network setup and configuration for your Amazon MWAA environment, see the [Verify Environment](https://github.com/awslabs/aws-support-tools/tree/master/MWAA) script in AWS Support Tools on GitHub\.

## Updating requirements\.txt<a name="troubleshooting-dependencies"></a>

The following topic describes the errors you may receive when updating your `requirements.txt`\.

### Adding `apache-airflow-providers-amazon` causes my environment to fail<a name="t-requirements"></a>

`apache-airflow-providers-xyz` is only compatible with Apache Airflow 2\.0\. `apache-airflow-backport-providers-xyz` is compatible with Apache Airflow 1\.10\.12\.

### I specified a new version of my `requirements.txt` and it's taking more than 20 minutes to update my environment<a name="t-requirements"></a>

If it takes more than twenty minutes for your environment to install a new version of a `requirements.txt` file, the environment update failed and Amazon MWAA is rolling back to the last stable version of the container image\.

1. Check package versions\. We recommend always specifying either a specific version \(`==`\) or a maximum version \(`>=`\) for the Python dependencies in your `requirements.txt`\.

1. Check Apache Airflow logs\. If you enabled Apache Airflow logs, verify your log groups were created successfully on the [Logs groups page](https://console.aws.amazon.com/cloudwatch/home#logsV2:log-groups) on the CloudWatch console\. If you see blank logs, the most common reason is due to missing permissions in your execution role for CloudWatch or Amazon S3 where logs are written\. To learn more, see [Execution role](mwaa-create-role.md)\.

1. Check Apache Airflow configuration options\. If you're using Secrets Manager, verify that the key\-value pairs you specified as an Apache Airflow configuration option were configured correctly\. To learn more, see [Configuring an Apache Airflow connection using a Secrets Manager secret key](connections-secrets-manager.md)\.

1. Check execution role permissions\. An execution role is an AWS Identity and Access Management \(IAM\) role with a permissions policy that grants Amazon MWAA permission to invoke the resources of other AWS services \(such as Amazon S3, CloudWatch, Amazon SQS, Amazon ECR\) on your behalf\. Your [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk) or [AWS owned CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk) also needs to be permitted access\. To learn more, see [Execution role](mwaa-create-role.md)\.

1. Check network access\. Verify that each of the AWS resources used by the environment are configured to allow network traffic, as defined in [Security in your VPC](vpc-security.md)\.

1. To run a troubleshooting script that checks the Amazon VPC network setup and configuration for your Amazon MWAA environment, see the [Verify Environment](https://github.com/awslabs/aws-support-tools/tree/master/MWAA) script in AWS Support Tools on GitHub\.

## Access environment<a name="troubleshooting-access-environment"></a>

The following topic describes the errors you may receive when accessing an environment\.

### I can't access the Apache Airflow UI<a name="t-no-access-airflow-ui"></a>

We recommend the following steps:

1. Check user permissions\. You may not have been granted access to a permissions policy that allows you to view the Apache Airflow UI\. To learn more, see [Accessing an Amazon MWAA environment](access-policies.md)\.

1. Check network access\. This may be because you selected the **Private network** access mode\. If the URL of your Apache Airflow UI is in the following format `387fbcn-8dh4-9hfj-0dnd-834jhdfb-vpce.c10.us-west-2.airflow.amazonaws.com`, it means that you're using *private routing* for your Apache Airflow *Web server*\. You can either update the Apache Airflow access mode to the **Public network** access mode, or create a mechanism to access the VPC endpoint for your Apache Airflow *Web server*\. To learn more, see [Managing access to VPC endpoints on Amazon MWAA](vpc-vpe-access.md)\.

## Broken DAG<a name="troubleshooting-broken-dags"></a>

The following topic describes the errors you may receive when running DAGs\.

### I received a 'Broken DAG' error when using Amazon DynamoDB operators<a name="missing-boto"></a>

We recommend the following steps:

1. Test your Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

1. Add the following package to your `requirements.txt`\.

   ```
   boto
   ```

1. Explore ways to specify Python dependencies in a `requirements.txt` file, see [Managing Python dependencies in requirements\.txt](best-practices-dependencies.md)\.

### I received 'Broken DAG: No module named psycopg2' error<a name="missing-postgres-library"></a>

We recommend the following steps:

1. Test your Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

1. Add the following to your `requirements.txt` with your Apache Airflow version\. For example:

   ```
   apache-airflow[postgres]==1.10.12
   ```

1. Explore ways to specify Python dependencies in a `requirements.txt` file, see [Managing Python dependencies in requirements\.txt](best-practices-dependencies.md)\.

### I received a 'Broken DAG' error when using the Slack operators<a name="missing-slack"></a>

We recommend the following steps:

1. Test your Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

1. Add the following package to your `requirements.txt` and specify your Apache Airflow version\. For example:

   ```
   apache-airflow[slack]==1.10.12
   ```

1. Explore ways to specify Python dependencies in a `requirements.txt` file, see [Managing Python dependencies in requirements\.txt](best-practices-dependencies.md)\.

### I received various errors installing Google/GCP/BigQuery<a name="missing-bigquery-cython"></a>

Amazon MWAA uses Amazon Linux which requires a specific version of Cython and cryptograpy libraries\. We recommend the following steps:

1. Test your Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

1. Add the following package to your `requirements.txt`\.

   ```
   grpcio==1.27.2
   cython==0.29.21
   pandas-gbq==0.13.3
   cryptography==3.3.2
   apache-airflow-backport-providers-amazon[google]
   ```

1. If you’re not using backport providers, you can use:

   ```
   grpcio==1.27.2
   cython==0.29.21
   pandas-gbq==0.13.3
   cryptography==3.3.2
   apache-airflow[gcp]==1.10.12
   ```

1. Explore ways to specify Python dependencies in a `requirements.txt` file, see [Managing Python dependencies in requirements\.txt](best-practices-dependencies.md)\.

### I received 'Broken DAG: No module named Cython' error<a name="broken-cython"></a>

Amazon MWAA uses Amazon Linux which requires a specific version of Cython\. We recommend the following steps:

1. Test your Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

1. Add the following package to your `requirements.txt`\.

   ```
   cython==0.29.21
   ```

1. Cython libraries have various required pip dependency versions\. For example, using `awswrangler==2.4.0` requires `pyarrow<3.1.0,>=2.0.0`, so pip3 tries to install `pyarrow==3.0.0` which causes a Broken DAG error\. We recommend specifying the oldest acceptible version explicity\. For example, if you specify the minimum value `pyarrow==2.0.0` before `awswrangler==2.4.0` then the error goes away, and the `requirements.txt` installs correctly\. The final requirements should look like this:

   ```
   cython==0.29.21
   pyarrow==2.0.0
   awswrangler==2.4.0
   ```

1. Explore ways to specify Python dependencies in a `requirements.txt` file, see [Managing Python dependencies in requirements\.txt](best-practices-dependencies.md)\.

## Operators<a name="troubleshooting-operators"></a>

The following topic describes the errors you may receive when using Operators\.

### I received an error using the BigQuery operator<a name="bigquery-operator-ui"></a>

Amazon MWAA does not support operators with UI extensions\. We recommend the following steps:

1. A workaround is to override the extension by adding a line in the DAG to set `<operator name>.operator_extra_links = None` after importing the problem operators\. For example:

   ```
   from airflow.contrib.operators.bigquery_operator import BigQueryOperator
   BigQueryOperator.operator_extra_links = None
   ```

1. You can use this approach for all DAGs by adding the above to a plugin\. For an example, see [Creating a custom plugin for Apache Airflow PythonVirtualenvOperator](samples-virtualenv.md)\.

## Connections<a name="troubleshooting-connections"></a>

The following topic describes the errors you may receive when connecting to an environment\.

### I can't connect to Snowflake<a name="missing-snowflake"></a>

We recommend the following steps:

1. Test your Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

1. Add the following entries to the requirements\.txt for your environment\.

   ```
   asn1crypto == 0.24.0
   snowflake-connector-python == 1.7.2
   ```

1. Explore ways to specify Python dependencies in a `requirements.txt` file, see [Managing Python dependencies in requirements\.txt](best-practices-dependencies.md)\.

1. Add the following imports to your DAG:

   ```
   from airflow.contrib.hooks.snowflake_hook import SnowflakeHook
   from airflow.contrib.operators.snowflake_operator import SnowflakeOperator
   ```

Ensure the Apache Airflow connection object includes the following key\-value pairs:

1. **Conn Id: **snowflake\_conn

1. **Conn Type: **Snowflake

1. **Host: **<my account>\.<my region if not us\-west\-2>\.snowflakecomputing\.com

1. **Schema: **<my schema>

1. **Login: **<my user name>

1. **Password: **\*\*\*\*\*\*\*\*

1. **Port: ** <port, if any>

1. **Extra: **

   ```
   {
     "account": "<my account>",
     "warehouse": "<my warehouse>",
     "database": "<my database>",
     "region": "<my region if not using us-west-2 otherwise omit this line>"
   }
   ```

For example:

```
>>> import json
>>> from airflow.models.connection import Connection
>>> myconn = Connection(
...    conn_id='snowflake_conn',
...    conn_type='Snowflake',
...    host='YOUR_ACCOUNT.YOUR_REGION.snowflakecomputing.com',
...    schema='YOUR_SCHEMA'
...    login='YOUR_USERNAME',
...    password='YOUR_PASSWORD',
...    port='YOUR_PORT'
...    extra=json.dumps(dict(account='YOUR_ACCOUNT', warehouse='YOUR_WAREHOUSE, database='YOUR_DB_OPTION', region='YOUR_REGION')),
... )
```

### I can't connect to Secrets Manager<a name="access-secrets-manager"></a>

We recommend the following steps:
+ Learn how to create secret keys for your Apache Airflow connection and variables in [Configuring an Apache Airflow connection using a Secrets Manager secret key](connections-secrets-manager.md)\.

### I can't connect to my MySQL server on '<DB\-identifier\-name>\.cluster\-id\.<region>\.rds\.amazonaws\.com'<a name="mysql-server"></a>

Amazon MWAA's security group and the RDS security group need an ingress rule to allow traffic to and from one another\. We recommend the following steps:

1. Modify the RDS security group to allow all traffic from Amazon MWAA security group\.

1. Modify the Amazon MWAA security group to allow all traffic from the RDS security group\.

1. Rerun your tasks again and verify whether the SQL query succeeded by checking logs\.

## Web server<a name="troubleshooting-web-server"></a>

The following topic describes the errors you may receive for your Apache Airflow web server in an environment\.

### I'm using the BigQueryOperator and it's causing my web server to crash<a name="operator-biquery"></a>

We recommend the following steps:
+ Apache Airflow operators such as the `BigQueryOperator` and `QuboleOperator` that contain `operator_extra_links` could cause your Apache Airflow web server to crash\. These operators attempt to load code to your web server, which is not permitted for security reasons\. We recommend patching the operators in your DAG by adding the following code after your import statements:

  ```
  BigQueryOperator.operator_extra_links = None
  ```

  To update your DAG code, see [](configuring-dag-folder.md)\.

### I see a 5xx error accessing the web server<a name="5xx-webserver"></a>

We recommend the following steps:

1. Check Apache Airflow configuration options\. Verify that the key\-value pairs you specified as an Apache Airflow configuration option, such as AWS Secrets Manager, were configured correctly\. To learn more, see [I can't connect to Secrets Manager](#access-secrets-manager)\.

1. Check the `requirements.txt`\. Verify the providers package and other libraries are compatible with Apache Airflow v1\.10\.12\.

1. Explore ways to specify Python dependencies in a `requirements.txt` file, see [Managing Python dependencies in requirements\.txt](best-practices-dependencies.md)\.

## Tasks<a name="troubleshooting-tasks"></a>

The following topic describes the errors you may receive for Apache Airflow tasks in an environment\.

### I see my tasks stuck in the running state<a name="stranded-tasks"></a>

We recommend the following steps:

1. If you have a DAG with tasks stuck in the running state, you can try to clear the tasks or mark them as succeeded or failed\. This allows the autoscaling component for your environment to scale down the number of workers running in an environment\. The following image shows an example of a stranded task\.  
![\[This is an image with a stranded task.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-airflow-scaling.png)

1. Choose the circle for the stranded task, and then select **Clear** \(as shown\)\. This allows Amazon MWAA to scale down workers; otherwise, Amazon MWAA can't determine which DAGs are enabled or disabled, and can't scale down, if there are still queued tasks\.  
![\[Apache Airflow Actions\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-airflow-scaling-menu.png)

## Logs<a name="troubleshooting-view-logs"></a>

The following topic describes the errors you may receive when viewing Apache Airflow logs\.

### I can’t see my task logs or I received a remote log error in the Airflow UI<a name="t-task-logs"></a>

If you see blank logs, or the follow error when viewing *Task logs* in the Airflow UI:

```
*** Reading remote log from Cloudwatch log_group: airflow-{environmentName}-Task log_stream: {DAG_ID}/{TASK_ID}/{time}/{n}.log.Could not read remote logs from log_group: airflow-{environmentName}-Task log_stream: {DAG_ID}/{TASK_ID}/{time}/{n}.log.
```
+ We recommend the following steps:

  1. Verify that you enabled task logs at the INFO level in your environment details view\.

  1. Verify that your operator has the appropriate Python libraries to load correctly\. You can try eliminating imports until you find the one that is causing the issue\.

  1. Explore ways to specify Python dependencies in a `requirements.txt` file, see [Managing Python dependencies in requirements\.txt](best-practices-dependencies.md)\.

### I keep seeing a ResourceAlreadyExistsException error in CloudTrail<a name="t-cloudtrail"></a>

```
"errorCode": "ResourceAlreadyExistsException",
    "errorMessage": "The specified log stream already exists",
    "requestParameters": {
        "logGroupName": "airflow-MyAirflowEnvironment-DAGProcessing",
        "logStreamName": "scheduler_cross-account-eks.py.log"
    }
```

Certain Python requirements such as `apache-airflow-backport-providers-amazon` roll back the `watchtower` library that Amazon MWAA uses to communicate with CloudWatch to an older version\. We recommend the following steps:
+ Add the following library to your `requirements.txt`

  ```
  watchtower==1.0.6
  ```

### I keep seeing an Invalid request error in CloudTrail<a name="t-cloudtrail-bucket"></a>

```
Invalid request provided: Provided role does not have sufficient permissions for s3 location airflow-xxx-xxx/dags
```

If you're creating an Amazon MWAA environment and an Amazon S3 bucket using the same AWS CloudFormation template, you need to add a `DependsOn` section within your AWS CloudFormation template\. The two resources \(*MWAA Environment* and *MWAA Execution Policy*\) have a dependency in AWS CloudFormation\. We recommend the following steps:
+ Add the following **DependsOn** statement to your AWS CloudFormation template\.

  ```
  ...
        MaxWorkers: 5
        NetworkConfiguration:
          SecurityGroupIds:
            - !GetAtt SecurityGroup.GroupId
          SubnetIds: !Ref subnetIds
        WebserverAccessMode: PUBLIC_ONLY
      DependsOn: MwaaExecutionPolicy
  
      MwaaExecutionPolicy:
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
  ...
  ```

  For an example, see [Quick start tutorial for Amazon Managed Workflows for Apache Airflow \(MWAA\)](quick-start.md)\.