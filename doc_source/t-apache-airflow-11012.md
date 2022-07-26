# Troubleshooting: DAGs, Operators, Connections, and other issues in Apache Airflow v1<a name="t-apache-airflow-11012"></a>

The topics on this page contains resolutions to Apache Airflow v1\.10\.12 Python dependencies, custom plugins, DAGs, Operators, Connections, tasks, and *Web server* issues you may encounter on an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment\.

**Contents**
+ [Updating requirements\.txt](#troubleshooting-dependencies)
  + [Adding `apache-airflow-providers-amazon` causes my environment to fail](#t-requirements)
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
  + [I see a 'The scheduler does not appear to be running' error](#error-scheduler-11012)
+ [Tasks](#troubleshooting-tasks)
  + [I see my tasks stuck or not completing](#stranded-tasks)
+ [CLI](#troubleshooting-cli-11012)
  + [I see a '503' error when triggering a DAG in the CLI](#cli-toomany-11012)

## Updating requirements\.txt<a name="troubleshooting-dependencies"></a>

The following topic describes the errors you may receive when updating your `requirements.txt`\.

### Adding `apache-airflow-providers-amazon` causes my environment to fail<a name="t-requirements"></a>

`apache-airflow-providers-xyz` is only compatible with Apache Airflow v2\. `apache-airflow-backport-providers-xyz` is compatible with Apache Airflow 1\.10\.12\.

## Broken DAG<a name="troubleshooting-broken-dags"></a>

The following topic describes the errors you may receive when running DAGs\.

### I received a 'Broken DAG' error when using Amazon DynamoDB operators<a name="missing-boto"></a>

We recommend the following steps:

1. Test your DAGs, custom plugins, and Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

1. Add the following package to your `requirements.txt`\.

   ```
   boto
   ```

1. Explore ways to specify Python dependencies in a `requirements.txt` file, see [Managing Python dependencies in requirements\.txt](best-practices-dependencies.md)\.

### I received 'Broken DAG: No module named psycopg2' error<a name="missing-postgres-library"></a>

We recommend the following steps:

1. Test your DAGs, custom plugins, and Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

1. Add the following to your `requirements.txt` with your Apache Airflow version\. For example:

   ```
   apache-airflow[postgres]==1.10.12
   ```

1. Explore ways to specify Python dependencies in a `requirements.txt` file, see [Managing Python dependencies in requirements\.txt](best-practices-dependencies.md)\.

### I received a 'Broken DAG' error when using the Slack operators<a name="missing-slack"></a>

We recommend the following steps:

1. Test your DAGs, custom plugins, and Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

1. Add the following package to your `requirements.txt` and specify your Apache Airflow version\. For example:

   ```
   apache-airflow[slack]==1.10.12
   ```

1. Explore ways to specify Python dependencies in a `requirements.txt` file, see [Managing Python dependencies in requirements\.txt](best-practices-dependencies.md)\.

### I received various errors installing Google/GCP/BigQuery<a name="missing-bigquery-cython"></a>

Amazon MWAA uses Amazon Linux which requires a specific version of Cython and cryptograpy libraries\. We recommend the following steps:

1. Test your DAGs, custom plugins, and Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

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

1. Test your DAGs, custom plugins, and Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

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

1. Test your DAGs, custom plugins, and Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

1. A workaround is to override the extension by adding a line in the DAG to set `<operator name>.operator_extra_links = None` after importing the problem operators\. For example:

   ```
   from airflow.contrib.operators.bigquery_operator import BigQueryOperator
   BigQueryOperator.operator_extra_links = None
   ```

1. You can use this approach for all DAGs by adding the above to a plugin\. For an example, see [Creating a custom plugin for Apache Airflow PythonVirtualenvOperator](samples-virtualenv.md)\.

## Connections<a name="troubleshooting-connections"></a>

The following topic describes the errors you may receive when using an Apache Airflow connection, or using another AWS database\.

### I can't connect to Snowflake<a name="missing-snowflake"></a>

We recommend the following steps:

1. Test your DAGs, custom plugins, and Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

1. Add the following entries to the requirements\.txt for your environment\.

   ```
   asn1crypto == 0.24.0
   snowflake-connector-python == 1.7.2
   ```

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
...    extra=json.dumps(dict(account='YOUR_ACCOUNT', warehouse='YOUR_WAREHOUSE', database='YOUR_DB_OPTION', region='YOUR_REGION')),
... )
```

### I can't connect to Secrets Manager<a name="access-secrets-manager"></a>

We recommend the following steps:

1. Learn how to create secret keys for your Apache Airflow connection and variables in [Configuring an Apache Airflow connection using a Secrets Manager secret](connections-secrets-manager.md)\.

1. Learn how to use the secret key for an Apache Airflow variable \(`test-variable`\) in [Using a secret key in AWS Secrets Manager for an Apache Airflow variable](samples-secrets-manager-var.md)\.

1. Learn how to use the secret key for an Apache Airflow connection \(`myconn`\) in [Using a secret key in AWS Secrets Manager for an Apache Airflow connection](samples-secrets-manager.md)\.

### I can't connect to my MySQL server on '<DB\-identifier\-name>\.cluster\-id\.<region>\.rds\.amazonaws\.com'<a name="mysql-server"></a>

Amazon MWAA's security group and the RDS security group need an ingress rule to allow traffic to and from one another\. We recommend the following steps:

1. Modify the RDS security group to allow all traffic from Amazon MWAA's VPC security group\.

1. Modify Amazon MWAA's VPC security group to allow all traffic from the RDS security group\.

1. Rerun your tasks again and verify whether the SQL query succeeded by checking Apache Airflow logs in CloudWatch Logs\.

## Web server<a name="troubleshooting-web-server"></a>

The following topic describes the errors you may receive for your Apache Airflow *Web server* on Amazon MWAA\.

### I'm using the BigQueryOperator and it's causing my web server to crash<a name="operator-biquery"></a>

We recommend the following steps:

1. Apache Airflow operators such as the `BigQueryOperator` and `QuboleOperator` that contain `operator_extra_links` could cause your Apache Airflow web server to crash\. These operators attempt to load code to your web server, which is not permitted for security reasons\. We recommend patching the operators in your DAG by adding the following code after your import statements:

   ```
   BigQueryOperator.operator_extra_links = None
   ```

1. Test your DAGs, custom plugins, and Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

### I see a 5xx error accessing the web server<a name="5xx-webserver"></a>

We recommend the following steps:

1. Check Apache Airflow configuration options\. Verify that the key\-value pairs you specified as an Apache Airflow configuration option, such as AWS Secrets Manager, were configured correctly\. To learn more, see [I can't connect to Secrets Manager](#access-secrets-manager)\.

1. Check the `requirements.txt`\. Verify the Airflow "extras" package and other libraries listed in your `requirements.txt` are compatible with your Apache Airflow version\.

1. Explore ways to specify Python dependencies in a `requirements.txt` file, see [Managing Python dependencies in requirements\.txt](best-practices-dependencies.md)\.

### I see a 'The scheduler does not appear to be running' error<a name="error-scheduler-11012"></a>

If the scheduler doesn't appear to be running, or the last "heart beat" was received several hours ago, your DAGs may not appear in Apache Airflow, and new tasks will not be scheduled\.

We recommend the following steps:

1. Confirm that your VPC security group allows inbound access to port `5432`\. This port is needed to connect to the Amazon Aurora PostgreSQL metadata database for your environment\. After this rule is added, give Amazon MWAA a few minutes, and the error should disappear\. To learn more, see [Security in your VPC on Amazon MWAA](vpc-security.md)\. 
**Note**  
 The Aurora PostgreSQL metadatabase is part of the [Amazon MWAA service architecture](what-is-mwaa.md#architecture-mwaa) and is not visible in your AWS account\. 
 Database\-related errors are usually a symptom of scheduler failure and not the root cause\. 

1.  If the scheduler is not running, it might be due to a number of factors such as [dependency installation failures](best-practices-dependencies.md), or an [overloaded scheduler](best-practices-tuning.md)\. Confirm that your DAGs, plugins, and requirements are working correctly by viewing the corresponding log groups in CloudWatch Logs\. To learn more, see [Monitoring and metrics for Amazon Managed Workflows for Apache Airflow \(MWAA\)](cw-metrics.md)\.

## Tasks<a name="troubleshooting-tasks"></a>

The following topic describes the errors you may receive for Apache Airflow tasks in an environment\.

### I see my tasks stuck or not completing<a name="stranded-tasks"></a>

If your Apache Airflow tasks are "stuck" or not completing, we recommend the following steps:

1. There may be a large number of DAGs defined\. Reduce the number of DAGs and perform an update of the environment \(such as changing a log level\) to force a reset\.

   1. Airflow parses DAGs whether they are enabled or not\. If you're using greater than 50% of your environment's capacity you may start overwhelming the Apache Airflow *Scheduler*\. This leads to large *Total Parse Time* in CloudWatch Metrics or long DAG processing times in CloudWatch Logs\. There are other ways to optimize Apache Airflow configurations which are outside the scope of this guide\.

   1. To learn more about the best practices we recommend to tune the performance of your environment, see [Performance tuning for Apache Airflow on Amazon MWAA](best-practices-tuning.md)\.

1. There may be a large number of tasks in the queue\. This often appears as a large—and growing—number of tasks in the "None" state, or as a large number in *Queued Tasks* and/or *Tasks Pending* in CloudWatch\. This can occur for the following reasons:

   1. If there are more tasks to run than the environment has the capacity to run, and/or a large number of tasks that were queued before autoscaling has time to detect the tasks and deploy additional *Workers*\. 

   1. If there are more tasks to run than an environment has the capacity to run, we recommend **reducing** the number of tasks that your DAGs run concurrently, and/or increasing the minimum Apache Airflow *Workers*\.

   1. If there are a large number of tasks that were queued before autoscaling has had time to detect and deploy additional workers, we recommend **staggering** task deployment and/or increasing the minimum Apache Airflow *Workers*\.

   1. You can use the [update\-environment](https://docs.aws.amazon.com/cli/latest/reference/mwaa/update-environment.html) command in the AWS Command Line Interface \(AWS CLI\) to change the minimum or maximum number of *Workers* that run on your environment\.

      ```
      aws mwaa update-environment --name MyEnvironmentName --min-workers 2 --max-workers 10
      ```

   1. To learn more about the best practices we recommend to tune the performance of your environment, see [Performance tuning for Apache Airflow on Amazon MWAA](best-practices-tuning.md)\.

1. There may be tasks being deleted mid\-execution that appear as task logs which stop with no further indication in Apache Airflow\. This can occur for the following reasons:

   1. If there is a brief moment where 1\) the current tasks exceed current environment capacity, followed by 2\) a few minutes of no tasks executing or being queued, then 3\) new tasks being queued\. 

   1. Amazon MWAA autoscaling reacts to the first scenario by adding additional workers\. In the second scenario, it removes the additional workers\. Some of the tasks being queued may result with the workers in the process of being removed, and will end when the container is deleted\.

   1. We recommend increasing the minimum number of workers on your environment\. Another option is to adjust the timing of your DAGs and tasks to ensure that that these scenarios don't occur\.

   1. You can also set the minimum workers equal to the maximum workers on your environment, effectively disabling autoscaling\. Use the [update\-environment](https://docs.aws.amazon.com/cli/latest/reference/mwaa/update-environment.html) command in the AWS Command Line Interface \(AWS CLI\) to **disable autoscaling** by setting the minimum and maximum number of workers to be the same\.

      ```
      aws mwaa update-environment --name MyEnvironmentName --min-workers 5 --max-workers 5
      ```

   1. To learn more about the best practices we recommend to tune the performance of your environment, see [Performance tuning for Apache Airflow on Amazon MWAA](best-practices-tuning.md)\.

1. If your tasks are stuck in the "running" state, you can also clear the tasks or mark them as succeeded or failed\. This allows the autoscaling component for your environment to scale down the number of workers running on your environment\. The following image shows an example of a stranded task\.  
![\[This is an image with a stranded task.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-airflow-scaling.png)

   1. Choose the circle for the stranded task, and then select **Clear** \(as shown\)\. This allows Amazon MWAA to scale down workers; otherwise, Amazon MWAA can't determine which DAGs are enabled or disabled, and can't scale down, if there are still queued tasks\.  
![\[Apache Airflow Actions\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-airflow-scaling-menu.png)

1. Learn more about the Apache Airflow task lifecycle at [Concepts](https://airflow.apache.org/docs/apache-airflow/stable/concepts.html#task-lifecycle) in the *Apache Airflow reference guide*\.

## CLI<a name="troubleshooting-cli-11012"></a>

The following topic describes the errors you may receive when running Airflow CLI commands in the AWS Command Line Interface\.

### I see a '503' error when triggering a DAG in the CLI<a name="cli-toomany-11012"></a>

The Airflow CLI runs on the Apache Airflow *Web server*, which has limited concurrency\. Typically a maximum of 4 CLI commands can run simultaneously\.