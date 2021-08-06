# Troubleshooting: CloudWatch Logs and CloudTrail errors<a name="t-cloudwatch-cloudtrail-logs"></a>

The topics on this page contains resolutions to Amazon CloudWatch Logs and AWS CloudTrail errors you may encounter on an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment\.

**Contents**
+ [Logs](#troubleshooting-view-logs)
  + [I can't see my task logs or I received a 'remote log' error in the Airflow UI](#t-task-logs)
  + [I see a 'ResourceAlreadyExistsException' error in CloudTrail](#t-cloudtrail)
  + [I see an 'Invalid request' error in CloudTrail](#t-cloudtrail-bucket)
  + [I see a 'Cannot locate a 64\-bit Oracle Client library: "libclntsh\.so: cannot open shared object file: No such file or directory' in Apache Airflow logs](#t-plugins-logs)
  + [I see psycopg2 'server closed the connection unexpectedly' in my Scheduler logs](#scheduler-postgres-library)
  + [I see 'Executor reports task instance %s finished \(%s\) although the task says its %s' in my DAG processing logs](#long-running-tasks)
  + [I see 'Could not read remote logs from log\_group: airflow\-\*\{\*environmentName\}\-Task log\_stream:\* \{\*DAG\_ID\}/\*\{\*TASK\_ID\}/\*\{\*time\}/\*\{\*n\}\.log\.' in my task logs](#t-task-fail-permission)

## Logs<a name="troubleshooting-view-logs"></a>

The following topic describes the errors you may receive when viewing Apache Airflow logs\.

### I can't see my task logs or I received a 'remote log' error in the Airflow UI<a name="t-task-logs"></a>

If you see blank logs, or the follow error when viewing *Task logs* in the Airflow UI:

```
*** Reading remote log from Cloudwatch log_group: airflow-{environmentName}-Task log_stream: {DAG_ID}/{TASK_ID}/{time}/{n}.log.Could not read remote logs from log_group: airflow-{environmentName}-Task log_stream: {DAG_ID}/{TASK_ID}/{time}/{n}.log.
```
+ We recommend the following steps:

  1. Verify that you enabled task logs at the INFO level in your environment details view\.

  1. Verify that your operator has the appropriate Python libraries to load correctly\. You can try eliminating imports until you find the one that is causing the issue\.

### I see a 'ResourceAlreadyExistsException' error in CloudTrail<a name="t-cloudtrail"></a>

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

### I see an 'Invalid request' error in CloudTrail<a name="t-cloudtrail-bucket"></a>

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

### I see a 'Cannot locate a 64\-bit Oracle Client library: "libclntsh\.so: cannot open shared object file: No such file or directory' in Apache Airflow logs<a name="t-plugins-logs"></a>
+ We recommend the following steps:

  1. If you're using Apache Airflow v2\.0\.2, add `core.lazy_load_plugins : False` as an Airflow configuration option\. To learn more, see [Using configuration options to load plugins in 2\.0](configuring-env-variables.md#configuring-2.0-airflow-override)\.

### I see psycopg2 'server closed the connection unexpectedly' in my Scheduler logs<a name="scheduler-postgres-library"></a>

If you see an error similar to the following, your Apache Airflow *Scheduler* may have run out of resources\.

```
2021-06-14T10:20:24.581-05:00    sqlalchemy.exc.OperationalError: (psycopg2.OperationalError) server closed the connection unexpectedly
2021-06-14T10:20:24.633-05:00    This probably means the server terminated abnormally
2021-06-14T10:20:24.686-05:00    before or while processing the request.
```

We recommend the following steps:
+ Consider upgrading to Apache Airflow v2\.0\.2, which allows you to specify up to 5 *Schedulers*\.

### I see 'Executor reports task instance %s finished \(%s\) although the task says its %s' in my DAG processing logs<a name="long-running-tasks"></a>

If you see an error similar to the following, your long\-running tasks may have reached the task time limit on Amazon MWAA\. Amazon MWAA has a limit of 12 hours for any one Airflow task, to prevent tasks from getting stuck in the queue and blocking activities like autoscaling\. 

```
Executor reports task instance %s finished (%s) although the task says its %s. (Info: %s) Was the task killed externally
```

We recommend the following steps:
+ Consider breaking up the task into multiple, shorter running tasks\. Airflow typically has a model whereby operators are asynchronous\. It invokes activities on external systems, and Apache Airflow Sensors poll to see when its complete\. If a Sensor fails, it can be safely retried without impacting the Operator's functionality\.

### I see 'Could not read remote logs from log\_group: airflow\-\*\{\*environmentName\}\-Task log\_stream:\* \{\*DAG\_ID\}/\*\{\*TASK\_ID\}/\*\{\*time\}/\*\{\*n\}\.log\.' in my task logs<a name="t-task-fail-permission"></a>

If you see an error similar to the following, the execution role for your environment may not contain a permissions policy to create log streams for task logs\. 

```
Could not read remote logs from log_group: airflow-*{*environmentName}-Task log_stream:* {*DAG_ID}/*{*TASK_ID}/*{*time}/*{*n}.log.
```

We recommend the following steps:
+ Modify the execution role for your environment using one of the sample policies at [Amazon MWAA execution role](mwaa-create-role.md)\.

You may have also specified a provider package in your `requirements.txt` file that is incompatible with your Apache Airflow version\. For example, if you're using Apache Airflow v2\.0\.2, you may have specified a package, such as the [apache\-airflow\-providers\-databricks](https://airflow.apache.org/docs/apache-airflow-providers-databricks/stable/index.html) package, which is only compatible with Airflow 2\.1\+\.

We recommend the following steps:

1. If you're using Apache Airflow v2\.0\.2, modify the `requirements.txt` file and add `apache-airflow[databricks]`\. This installs the correct version of the Databricks package that is compatible with Apache Airflow v2\.0\.2\.

1. Test your DAGs, custom plugins, and Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.