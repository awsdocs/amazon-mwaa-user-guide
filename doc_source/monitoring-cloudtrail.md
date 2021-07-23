# Viewing audit logs in AWS CloudTrail<a name="monitoring-cloudtrail"></a>

AWS CloudTrail is enabled on your AWS account when you create it\. CloudTrail logs the activity taken by a user, role, or an AWS service, such as Amazon Managed Workflows for Apache Airflow \(MWAA\), which is recorded as a CloudTrail event\. You can view, search, and download the past 90 days of event history in the CloudTrail console\. CloudTrail captures all events on the Amazon MWAA console and all calls to Amazon MWAA APIs\. It doesn't capture read\-only actions, such as `GetEnvironment`, or the `PublishMetrics` action\. This page describes how to use CloudTrail to monitor events for Amazon MWAA\.

**Contents**
+ [Creating a trail in CloudTrail](#monitoring-cloudtrail-create)
+ [Viewing events with CloudTrail Event History](#monitoring-cloudtrail-view)
+ [Example trail for `CreateEnvironment`](#monitoring-cloudtrail-logs-ex)
+ [What's next?](#monitoring-cloudtrail-next-up)

## Creating a trail in CloudTrail<a name="monitoring-cloudtrail-create"></a>

You need to create a trail to view an ongoing record of events in your AWS account, including events for Amazon MWAA\. A trail enables CloudTrail to deliver log files to an Amazon S3 bucket\. If you do not create a trail, you can still view available event history in the CloudTrail console\. For example, using the information collected by CloudTrail, you can determine the request that was made to Amazon MWAA, the IP address from which the request was made, who made the request, when it was made, and additional details\. To learn more, see the [Creating a trail for your AWS account](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-create-and-update-a-trail.html)\.

## Viewing events with CloudTrail Event History<a name="monitoring-cloudtrail-view"></a>

You can troubleshoot operational and security incidents over the past 90 days in the CloudTrail console by viewing event history\. For example, you can view events related to the creation, modification, or deletion of resources \(such as IAM users or other AWS resources\) in your AWS account on a per\-region basis\. To learn more, see the [Viewing Events with CloudTrail Event History](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/view-cloudtrail-events.html)\.

1. Open the [CloudTrail](https://console.aws.amazon.com/cloudtrail/home#) console\.

1. Choose **Event history**\.

1. Select the events you want to view, and then choose **Compare event details**\.

## Example trail for `CreateEnvironment`<a name="monitoring-cloudtrail-logs-ex"></a>

A trail is a configuration that enables delivery of events as log files to an Amazon S3 bucket that you specify\. 

CloudTrail log files contain one or more log entries\. An event represents a single request from any source and includes information about the requested action, such as the date and time of the action, or request parameters\. CloudTrail log files are not an ordered stack trace of the public API calls, and don't appear in any specific order\. The following example is a log entry for the `CreateEnvironment` action that is denied due to lacking permissions\. The values in `AirflowConfigurationOptions` have been redacted for privacy\.

```
{
    "eventVersion": "1.05",
    "userIdentity": {
        "type": "AssumedRole",
        "principalId": "00123456ABC7DEF8HIJK",
        "arn": "arn:aws:sts::012345678901:assumed-role/root/myuser",
        "accountId": "012345678901",
        "accessKeyId": "",
        "sessionContext": {
            "sessionIssuer": {
                "type": "Role",
                "principalId": "00123456ABC7DEF8HIJK",
                "arn": "arn:aws:iam::012345678901:role/user",
                "accountId": "012345678901",
                "userName": "user"
            },
            "webIdFederationData": {},
            "attributes": {
                "mfaAuthenticated": "false",
                "creationDate": "2020-10-07T15:51:52Z"
            }
        }
    },
    "eventTime": "2020-10-07T15:52:58Z",
    "eventSource": "airflow.amazonaws.com",
    "eventName": "CreateEnvironment",
    "awsRegion": "us-west-2",
    "sourceIPAddress": "205.251.233.178",
    "userAgent": "PostmanRuntime/7.26.5",
    "errorCode": "AccessDenied",
    "requestParameters": {
        "SourceBucketArn": "arn:aws:s3:::my-bucket",
        "ExecutionRoleArn": "arn:aws:iam::012345678901:role/AirflowTaskRole",
        "AirflowConfigurationOptions": "***",
        "DagS3Path": "sample_dag.py",
        "NetworkConfiguration": {
            "SecurityGroupIds": [
                "sg-01234567890123456"
            ],
            "SubnetIds": [
                "subnet-01234567890123456",
                "subnet-65432112345665431"
            ]
        },
        "Name": "test-cloudtrail"
    },
    "responseElements": {
        "message": "Access denied."
    },
    "requestID": "RequestID",
    "eventID": "EventID",
    "readOnly": false,
    "eventType": "AwsApiCall",
    "recipientAccountId": "012345678901"
}
```

## What's next?<a name="monitoring-cloudtrail-next-up"></a>
+ Learn how to configure other AWS services for the event data collected in CloudTrail logs in [CloudTrail Supported Services and Integrations](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-aws-service-specific-topics.html#cloudtrail-aws-service-specific-topics-integrations)\.
+ Learn how to be notified when CloudTrail publishes new log files to an Amazon S3 bucket in [Configuring Amazon SNS Notifications for CloudTrail](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/getting_notifications_top_level.html)\.