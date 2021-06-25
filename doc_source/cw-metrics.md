# Amazon MWAA performance metrics in Amazon CloudWatch<a name="cw-metrics"></a>

Amazon CloudWatch is basically a metrics repository for AWS services that allows you to retrieve statistics based on the [metrics](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_concepts.html#Metric) and [dimensions](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_concepts.html#Dimension) published by a service\. You can use these metrics to configure [alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch_concepts.html#CloudWatchAlarms), calculate statistics and then present the data in a [dashboard](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Dashboards.html) that helps you assess the health of your environment in the Amazon CloudWatch console\. 

Apache Airflow is already set\-up to send [StatsD](https://github.com/etsy/statsd) metrics for an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment to Amazon CloudWatch\. This section describes the performance metrics and dimensions published to CloudWatch for your Amazon Managed Workflows for Apache Airflow \(MWAA\) environment\.

**Topics**
+ [Accessing Apache Airflow v2\.0\.2 metrics for an Amazon MWAA environment in CloudWatch](access-metrics-cw-202.md)
+ [Accessing Apache Airflow v1\.10\.12 metrics for an Amazon MWAA environment in CloudWatch](access-metrics-cw-110.md)