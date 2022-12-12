# Container, queue, and database metrics for Amazon MWAA<a name="accessing-metrics-cw-container-queue-db"></a>

 In addition to Apache Airflow metrics, you can monitor the underlying components of your Amazon Managed Workflows for Apache Airflow \(MWAA\) environments using CloudWatch, which collects raw data and processes data into readable, near real\-time metrics\. With these environment metrics, you will have greater visibility into key performance indicators to help you appropriately size your environments and debug issues with your workflows\. These metrics apply to all supported Apache Airflow versions on Amazon MWAA\. 

 

 Amazon MWAA will provide CPU and memory utilization for each Amazon Elastic Container Service \(Amazon ECS\) container and Amazon Aurora PostgreSQL instance, and Amazon Simple Queue Service \(Amazon SQS\) metrics for the number of messages and the age of the oldest message, Amazon Relational Database Service \(Amazon RDS\) metrics for database connections, disk queue depth, write operations, latency, and throughput, and Amazon RDS Proxy metrics\. These metrics also include the number of base workers, additional workers, schedulers, and web servers\. 

 These statistics are kept for 15 months, so that you can access historical information and gain a better perspective on why a schedule is failing, and troubleshoot underlying issues\. You can also set alarms that watch for certain thresholds, and send notifications or take actions when those thresholds are met\. For more information, see the [Amazon CloudWatch User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/)\. 

**Topics**
+ [Terms](#accessing-metrics-cw-container-queue-db-terms)
+ [Dimensions](#accessing-metrics-cw-container-queue-db-dimensions)
+ [Accessing metrics in the CloudWatch console](#accessing-metrics-cw-container-queue-db-console)
+ [List of metrics](#accessing-metrics-cw-container-queue-db-list)

## Terms<a name="accessing-metrics-cw-container-queue-db-terms"></a>

**Namespace**  
A namespace is a container for the CloudWatch metrics of an AWS service\. For Amazon MWAA, the namespace is `AWS/MWAA`\.

**CloudWatch metrics**  
A CloudWatch metric represents a time\-ordered set of data points that are specific to CloudWatch\. 

**Dimension**  
A dimension is a name/value pair that is part of the identity of a metric\. 

**Unit**  
 A statistic has a unit of measure\. For Amazon MWAA, units include *Count*\. 

## Dimensions<a name="accessing-metrics-cw-container-queue-db-dimensions"></a>

This section describes the CloudWatch dimensions grouping for Amazon MWAA metrics in CloudWatch\.


| Dimension | Description | 
| --- | --- | 
|  Cluster  |  Metrics for the minimum three Amazon ECS container that an Amazon MWAA environemnt uses to run Apache Airflow components: scheduler, worker, and web server\.  | 
|  Queue  |  Metrics for the Amazon SQS queues that decouple the scheduler from workers\. When workers read the messages, they are considered in\-flight and not available for other workers\. Messages become available for other workers to read if they are not deleted before the 12 hours visibility timeout\.  | 
|  Database  |  Metrics the Aurora clusters used by Amazon MWAA\. This includes metrics for the primary database instance and a read replica to support the read operations\. Amazon MWAA publishes database metrics for both READER and WRITER instances\.  | 

## Accessing metrics in the CloudWatch console<a name="accessing-metrics-cw-container-queue-db-console"></a>

This section describes how to access your Amazon MWAA metrics in CloudWatch\.

**To view performance metrics for a dimension**

1. Open the [Metrics page](https://console.aws.amazon.com/cloudwatch/home#metricsV2:graph=~()) on the CloudWatch console\.

1. Use the AWS Region selector to select your region\.

1. Choose the **AWS/MWAA** namespace\.

1. In the **All metrics** tab, select a dimension\. For example, **Cluster**\.

1. Choose a CloudWatch metric for a dimension\. For example, *NumSchedulers* or *CPUUtilization*\. Then, choose **Graph all search results**\.

1. Choose the **Graphed metrics** tab to view performance metrics\.

## List of metrics<a name="accessing-metrics-cw-container-queue-db-list"></a>

 The following tables list the cluster, queue, and database service metrics for Amazon MWAA\. To see unit and descriptions for metrics directly emitted from Amazon ECS, Amazon SQS, or Amazon RDS, choose the respective documentation link\. 


**Cluster metrics**  

| Namespace | Metric | Count | Description | 
| --- | --- | --- | --- | 
|  `AWS/MWAA`  |  `CPUUtilization`  |  Percent  |  Applies to each scheduler, worker, web server, and database proxy until it is replaced with an Amazon RDS proxy\. For more information about this metric, see [Available metrics and dimensions](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cloudwatch-metrics.html#available_cloudwatch_metrics) in the *Amazon ECS Developer Guide*\.  | 
|  `AWS/MWAA`  |  `MemoryUtilization`  |  Percent  |  Applies to each scheduler, worker, web server, and database proxy until it is replaced with an Amazon RDS proxy\. For more information about this metric, see [Available metrics and dimensions](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cloudwatch-metrics.html#available_cloudwatch_metrics) in the *Amazon ECS Developer Guide*\.  | 
|  `AWS/MWAA`  |  `NumWorkers`  |  Count  |  The default worker \(`BaseWorker`\) that comes with every Amazon MWAA environment\. The value for this metric is always `1`\.  | 
|  `AWS/MWAA`  |  `NumAdditionalWorkers`  |  Count  |   The number of additional workers that your environment uses if you specify a minimum number of workers higher than one\. When you configure more than one minimum worker, Amazon MWAA creates additional clusters \(`AdditionalWorker`\) to host the workers from two up to the maximum number of workers you set for your environment\.   | 
|  `AWS/MWAA`  |  `NumSchedulers`  |  Count  |   The number of schedulers that your environment uses\.   | 
|  `AWS/MWAA`  |  `NumWebServers`  |  Count  |   The number of web servers that your environment uses to host the Apache Airflow UI\.   | 

For more information on units and descriptions for the following database metrics, see [CloudWatch metrics for Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-metrics.html) in the *Amazon Relational Database Service User Guide*\.


**Database metrics**  

| Namespace | Metric | 
| --- | --- | 
|  `AWS/MWAA`  |  `CPUUtilization`  | 
|  `AWS/MWAA`  |  `DatabaseConnections`  | 
|  `AWS/MWAA`  |  `DiskQueueDepth`  | 
|  `AWS/MWAA`  |  `FreeableMemory`  | 
|  `AWS/MWAA`  |  `VolumeWriteIOPS`  | 
|  `AWS/MWAA`  |  `WriteIOPS`  | 
|  `AWS/MWAA`  |  `WriteLatency`  | 
|  `AWS/MWAA`  |  `WriteThroughput`  | 

For more information on units and descriptions on the following database proxy metrics, see [Monitoring Amazon RDS Proxy metrics with CloudWatch](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy.monitoring.html) in the *Amazon Relational Database Service User Guide*\.


**Database metrics for Amazon RDS Proxy \(when available\)**  

| Namespace | Metric | 
| --- | --- | 
|  `AWS/MWAA`  |  `ClientConnections`  | 
|  `AWS/MWAA`  |  `ClientConnectionsClosed`  | 
|  `AWS/MWAA`  |  `ClientConnectionsReceived`  | 
|  `AWS/MWAA`  |  `AvailabilityPercentage`  | 
|  `AWS/MWAA`  |  `DatabaseConnectionsCurrentlyInTransaction`  | 
|  `AWS/MWAA`  |  `DatabaseConnectionsSetupFailed`  | 
|  `AWS/MWAA`  |  `DatabaseConnectionsSetupSucceeded`  | 
|  `AWS/MWAA`  |  `DatabaseConnectionRequests`  | 
|  `AWS/MWAA`  |  `DatabaseConnections`  | 
|  `AWS/MWAA`  |  `QueryDatabaseResponseLatency`  | 
|  `AWS/MWAA`  |  `QueryRequests`  | 
|  `AWS/MWAA`  |  `QueryResponseLatency`  | 

For more information on units and descriptions for the following queue metrics, see [Available CloudWatch metrics for Amazon SQS](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-available-cloudwatch-metrics.html) in the *Amazon Simple Queue Service Developer Guide*\. 


**Queue metrics**  

| Namespace | Metric | 
| --- | --- | 
|  `AWS/MWAA`  |  `ApproximateAgeOfOldestMessage`  | 
|  `AWS/MWAA`  |  `ApproximateNumberOfMessagesNotVisible` \(Running tasks\)  | 
|  `AWS/MWAA`  |  `ApproximateNumberOfMessagesVisible` \(Queued tasks\)  | 