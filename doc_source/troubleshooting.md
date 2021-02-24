# Troubleshooting Amazon Managed Workflows for Apache Airflow \(MWAA\)<a name="troubleshooting"></a>

This topic describes common questions and resolutions to some errors and issues you may encounter when using Amazon Managed Workflows for Apache Airflow \(MWAA\)\.

**Contents**
+ [Common questions](#t-common-questions)
  + [When should I use AWS Step Functions vs\. Amazon MWAA?](#t-step-functions)
  + [Can I connect to the Aurora PostgreSQLmetadata database?](#q-access-database)
+ [Create environment](#troubleshooting-create-environment)
  + [I tried to create an environment but it shows the status as "Create failed"](#t-create-environ-failed)
  + [I tried to select a VPC and received a "Network Failure" error](#t-network-failure)
  + [I received a service, partition, or resource "must be passed" error](#t-service-partition)
+ [Access environment](#troubleshooting-access-environment)
  + [I can't access the Apache Airflow UI](#t-no-access-airflow-ui)
+ [Broken DAG](#troubleshooting-broken-dags)
  + [I received a 'Broken DAG' error when using Amazon DynamoDB operators](#missing-boto)
  + [I received 'Broken DAG: No module named psycopg2' error](#missing-postgres-library)
  + [I received a 'Broken DAG' error when using the Slack operators](#missing-slack)
+ [Connections](#troubleshooting-connections)
  + [I can't connect to Snowflake](#missing-snowflake)
  + [I can't connect to Secrets Manager](#access-secrets-manager)
+ [Web server](#troubleshooting-web-server)
  + [I'm using the BigQueryOperator and it's causing my web server to crash](#operator-biquery)
+ [Tasks](#troubleshooting-tasks)
  + [I see my tasks stuck in the running state](#stranded-tasks)
+ [Logs](#troubleshooting-view-logs)
  + [I can’t see my task logs or I received a remote log error in the Airflow UI](#t-task-logs)

## Common questions<a name="t-common-questions"></a>

### When should I use AWS Step Functions vs\. Amazon MWAA?<a name="t-step-functions"></a>
+ You can use Step Functions to process individual customer orders, since Step Functions can scale to meet demand for one order or one million orders\.
+ If you’re running an overnight workflow that processes the previous day’s orders, you can use Step Functions or Amazon MWAA\. Amazon MWAA allows you an open source option to abstract the workflow from the AWS resources you are using\.

### Can I connect to the Aurora PostgreSQLmetadata database?<a name="q-access-database"></a>
+ While you can't connect to the database on an Amazon MWAA environment directly, it is possible from a DAG\. To learn more, see [Aurora PostgreSQL database cleanup on an Amazon MWAA environment](samples-database-cleanup.md)\.
+ We also recommend viewing performance metrics for your environment using CloudWatch\. To learn more, see [What Is Amazon CloudWatch?](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)\.

## Create environment<a name="troubleshooting-create-environment"></a>

The following topic describes the errors you may receive when creating an environment\.

### I tried to create an environment but it shows the status as "Create failed"<a name="t-create-environ-failed"></a>

We recommend the following steps:
+ If you try to create an environment and it fails, confirm that the Amazon VPC network includes 2 private subnets that can access the Internet for creating containers\. To learn more, see [Create the VPC network](vpc-create.md)\.
+ Amazon MWAA performs a dry run against a user's credentials before creating an environment\. You may also be receiving this error because you do not have permission to create some of the resources for an environment\. For example, if you chose the **Private network** option, which requires VPC endpoints, your AWS account may not be permitted to create an Amazon MWAA environment with VPC endpoints\. We recommend asking your AWS account administrator for access\.

### I tried to select a VPC and received a "Network Failure" error<a name="t-network-failure"></a>

We recommend the following steps:
+ If you see a "Network Failure" error when you try to select a VPC when creating your environment, turn off any in\-browser proxies that are running, and then try again\.

### I received a service, partition, or resource "must be passed" error<a name="t-service-partition"></a>

We recommend the following steps:
+ This may be because the S3 URI you entered for the S3 bucket for your environment includes a '/' at the end of the URI\. To resolve this issue, remove the '/' in the S3 bucket path\. The value should be an S3 URI in the following format:

  ```
  s3://your-bucket-name
  ```

## Access environment<a name="troubleshooting-access-environment"></a>

The following topic describes the errors you may receive when accessing an environment\.

### I can't access the Apache Airflow UI<a name="t-no-access-airflow-ui"></a>

We recommend the following steps:
+ You may not have been granted access to a permissions policy that allows you to view the Apache Airflow UI\. To learn more, see [Accessing an Amazon MWAA environment](access-policies.md)\.
+ This may be because you selected the **Private network** option for your web server\. If the URL to your Apache Airflow UI is in the following format https://guid\-vpce\.xxx\.airflow\.amazonaws\.com/home, it means that you selected a private network option when you created the environment\. This means that the Apache Airflow UI URL is accessible only as a VPC endpoint from within your VPC\. You can either update the environment to use a public network, or connect the VPC endpoint via a NAT gateway or Linux bastion\. To learn more, see [Linux Bastion Hosts on AWS](https://aws.amazon.com/quickstart/architecture/linux-bastion/)\.

## Broken DAG<a name="troubleshooting-broken-dags"></a>

The following topic describes the errors you may receive when running DAGs\.

### I received a 'Broken DAG' error when using Amazon DynamoDB operators<a name="missing-boto"></a>

We recommend the following steps:
+ Add the following entry to your `requirements.txt`:

  ```
  boto
  ```

  Adding this to your `requirements.txt` is a required dependency for this package\. After you add this dependency to your `requirements.txt`, upload the file to your Amazon S3 bucket, then edit your environment on the Amazon MWAA console to select the new `requirements.txt` version of your file\. To learn more, see [Installing Python dependencies](working-dags-dependencies.md)\.

### I received 'Broken DAG: No module named psycopg2' error<a name="missing-postgres-library"></a>

We recommend the following steps:
+ Add the following entry to your `requirements.txt`:

  ```
  apache-airflow[postgres]
  ```

  Adding this to your `requirements.txt` is a required dependency for this package\. After you add this dependency to your `requirements.txt`, upload the file to your Amazon S3 bucket, then edit your environment on the Amazon MWAA console to select the new `requirements.txt` version of your file\. To learn more, see [Installing Python dependencies](working-dags-dependencies.md)

### I received a 'Broken DAG' error when using the Slack operators<a name="missing-slack"></a>

We recommend the following steps:
+ Add the following entry to your `requirements.txt`:

  ```
  apache-airflow[slack]
  ```

  Adding this to your `requirements.txt` is a required dependency for this package\. After you add this dependency to your `requirements.txt`, upload the file to your Amazon S3 bucket, then edit your environment on the Amazon MWAA console to select the new `requirements.txt` version of your file\. To learn more, see [Installing Python dependencies](working-dags-dependencies.md)

## Connections<a name="troubleshooting-connections"></a>

The following topic describes the errors you may receive when connecting to an environment\.

### I can't connect to Snowflake<a name="missing-snowflake"></a>

We recommend the following steps:
+ Add the following entries to the requirements\.txt for your environment\.

  ```
  asn1crypto == 0.24.0
  snowflake-connector-python == 1.7.2
  ```

  Include the following code in your DAG:

  ```
  from airflow.contrib.hooks.snowflake_hook import SnowflakeHook
  from airflow.contrib.operators.snowflake_operator import SnowflakeOperator
  ```

  Also ensure you have a connection defined in Apache Airflow with the following:
  + **Conn Id: **snowflake\_conn
  + **Conn Type: **Snowflake
  + **Host: **<my account>\.<my region if not us\-west\-2>\.snowflakecomputing\.com
  + **Schema: **<my schema>
  + **Login: **<my user name>
  + **Password: **\*\*\*\*\*\*\*\*
  + **Port: ** <port, if any>
  + **Extra: **

    ```
    {
      "account": "<my account>",
      "warehouse": "<my warehouse>",
      "database": "<my database>",
      "region": "<my region if not using us-west-2 otherwise omit this line>"
    }
    ```

  Adding this to your `requirements.txt` is a required dependency for this package\. After you add this dependency to your `requirements.txt`, upload the file to your Amazon S3 bucket, then edit your environment on the Amazon MWAA console to select the new `requirements.txt` version of your file\. To learn more, see [Installing Python dependencies](working-dags-dependencies.md)

### I can't connect to Secrets Manager<a name="access-secrets-manager"></a>

We recommend the following steps:
+ Adding an Apache Airflow configuration option of `secrets.backend` to `airflow.contrib.secrets.aws_secrets_manager.SecretsManagerBackend`\.
+ Add the connections/variables to AWS Secrets Manager\. For example:
  + For a variable called `max_metadb_storage_days` you would add `airflow/variables/max_metadb_storage_days` with a value of `14` to AWS Secrets Manager\.
  + For a connection called `my_db_connection` you would add `airflow/connections/my_db_connection` with a value of `14` to AWS Secrets Manager\.
+ Add the AWS Secrets Manager read policy to your environment’s execution role\.

While `backend_kwargs` is not supported, you can use a workaround to override the Secrets Manager function call by adding the following to your DAGs\. The following example adds a "2" to the prefix:

```
from airflow.contrib.secrets.aws_secrets_manager import SecretsManagerBackend
```

```
def get_variable(self, key):
  return self._get_secret('airflow/variables2', key)

SecretsManagerBackend.get_variable=get_variable
```

```
def get_conn_uri(self, key):
  return self._get_secret('airflow/connections2', key)

SecretsManagerBackend.get_conn_uri=get_conn_uri
```

```
def get_config(self, key):
  return self._get_secret('airflow/config2', key)

SecretsManagerBackend.get_config=get_config
```

To learn more about adding permissions to an execution role, see [Amazon MWAA Execution role](mwaa-create-role.md)\. To learn more about AWS Secrets Manager integrations, see [Using the Amazon MWAA console](configuring-env-variables.md) and [AWS Secrets Manager Backend](https://airflow.apache.org/docs/apache-airflow/1.10.12/howto/use-alternative-secrets-backend.html?highlight=secrets%20manager#aws-secrets-manager-backend) in the *Apache Airflow reference guide*\.

## Web server<a name="troubleshooting-web-server"></a>

The following topic describes the errors you may receive for your Apache Airflow web server in an environment\.

### I'm using the BigQueryOperator and it's causing my web server to crash<a name="operator-biquery"></a>

We recommend the following steps:
+ Apache Airflow operators such as the `BigQueryOperator` and `QuboleOperator` that contain `operator_extra_links` could cause your Apache Airflow web server to crash\. These operators attempt to load code to your web server, which is not permitted for security reasons\. We recommend patching the operators in your DAG by adding the following code after your import statements:

  ```
  BigQueryOperator.operator_extra_links = None
  ```

  To update your DAG code, see [](configuring-dag-folder.md)\.

## Tasks<a name="troubleshooting-tasks"></a>

The following topic describes the errors you may receive for Apache Airflow tasks in an environment\.

### I see my tasks stuck in the running state<a name="stranded-tasks"></a>

We recommend the following steps:
+ If you have a DAG with tasks stuck in the running state, you can try to clear the tasks or mark them as succeeded or failed\. This allows the autoscaling component for your environment to scale down the number of workers running in an environment\. The following image shows an example of a stranded task\.  
![\[This is an image with a stranded task.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-airflow-scaling.png)

  1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

  1. Choose an environment\.

  1. Choose **Open Airflow UI** to view your Apache Airflow UI\.
**Note**  
You may need to ask your account administrator to add `AmazonMWAAWebServerAccess` permissions for your account to view your Apache Airflow UI\. For more information, see [Managing access](https://docs.aws.amazon.com/mwaa/latest/userguide/manage-access.html)\.
+ Choose the circle for the stranded task, and then select **Clear** \(as shown in the following image\) so that Amazon MWAA can successfully scale down workers\. This is because Amazon MWAA can't determine which DAGs are enabled or disabled, and can't scale down, if there are still queued tasks\.  
![\[Apache Airflow Actions\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-airflow-scaling-menu.png)

## Logs<a name="troubleshooting-view-logs"></a>

The following topic describes the errors you may receive when viewing Apache Airflow logs\.

### I can’t see my task logs or I received a remote log error in the Airflow UI<a name="t-task-logs"></a>

We recommend the following steps:
+ If you see blank logs or the follow error when viewing Task logs in the Airflow UI

  ```
  *** Reading remote log from Cloudwatch log_group: airflow-{environmentName}-Task log_stream: {DAG_ID}/{TASK_ID}/{time}/{n}.log.Could not read remote logs from log_group: airflow-{environmentName}-Task log_stream: {DAG_ID}/{TASK_ID}/{time}/{n}.log.
  ```

  To resolve this:
  + Verify that you enabled task logs at the INFO level in your environment details view\.
  + Verify that your operator has the appropriate Python libraries to load correctly\. You can try eliminating imports until you find the one that is causing the issue\.
