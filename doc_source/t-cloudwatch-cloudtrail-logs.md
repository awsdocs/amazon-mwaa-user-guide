# Troubleshooting: CloudWatch Logs and CloudTrail errors<a name="t-cloudwatch-cloudtrail-logs"></a>

The topics on this page contains resolutions to Amazon CloudWatch Logs and AWS CloudTrail errors you may encounter on an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment\.

**Contents**
+ [Logs](#troubleshooting-view-logs)
  + [I can’t see my task logs or I received a remote log error in the Airflow UI](#t-task-logs)
  + [I keep seeing a ResourceAlreadyExistsException error in CloudTrail](#t-cloudtrail)
  + [I keep seeing an Invalid request error in CloudTrail](#t-cloudtrail-bucket)

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