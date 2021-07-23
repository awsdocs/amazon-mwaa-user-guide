# Monitoring overview on Amazon MWAA<a name="monitoring-overview"></a>

This page describes the AWS services used to monitor an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment\.

**Contents**
+ [Amazon CloudWatch overview](#monitoring-metrics-cw-about)
+ [AWS CloudTrail overview](#monitoring-metrics-ct-about)

## Amazon CloudWatch overview<a name="monitoring-metrics-cw-about"></a>

CloudWatch is a metrics repository for AWS services that allows you to retrieve statistics based on the [metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_concepts.html#Metric) and [dimensions](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_concepts.html#Dimension) published by a service\. You can use these metrics to configure [alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_concepts.html#CloudWatchAlarms), calculate statistics and then present the data in a [dashboard](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Dashboards.html) that helps you assess the health of your environment in the Amazon CloudWatch console\. 

Apache Airflow is already set\-up to send [StatsD](https://github.com/etsy/statsd) metrics for an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment to Amazon CloudWatch\. 

To learn more, see [What is Amazon CloudWatch?](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)\.

## AWS CloudTrail overview<a name="monitoring-metrics-ct-about"></a>

CloudTrail is an auditing service that provides a record of actions taken by a user, role, or an AWS service in Amazon MWAA\. Using the information collected by CloudTrail, you can determine the request that was made to Amazon MWAA, the IP address from which the request was made, who made the request, when it was made, and additional details available in audit logs\.

To learn more, see [What is AWS CloudTrail?](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-user-guide.html)\.