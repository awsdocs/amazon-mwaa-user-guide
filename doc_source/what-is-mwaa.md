# What Is Amazon Managed Workflows for Apache Airflow \(MWAA\)?<a name="what-is-mwaa"></a>

Amazon Managed Workflows for Apache Airflow \(MWAA\) is a managed orchestration service for [Apache Airflow](https://airflow.apache.org/) that makes it easier to set up and operate end\-to\-end data pipelines in the cloud at scale\. Apache Airflow is an open\-source tool used to programmatically author, schedule, and monitor sequences of processes and tasks referred to as "workflows\." With Amazon MWAA, you can use Airflow and Python to create workflows without having to manage the underlying infrastructure for scalability, availability, and security\. Amazon MWAA automatically scales its workflow execution capacity to meet your needs, and is integrated with AWS security services to help provide you with fast and secure access to data\.

**Topics**
+ [Integrated with AWS](#integrations-mwaa)
+ [Amazon MWAA regions](#regions-mwaa)
+ [Apache Airflow versions](#regions-aa-versions)
+ [Apache Airflow components](#components-aa)
+ [Amazon MWAA architecture](#architecture-mwaa)
+ [Amazon MWAA benefits](#benefits-mwaa)
+ [What's next?](#whatis-next-up)

## Integrated with AWS<a name="integrations-mwaa"></a>

The active and growing Apache Airflow open source community provides operators \(plugins that simplify connections to services\) for Apache Airflow to integrate with AWS services\. This includes services such as Amazon S3, Amazon Redshift, Amazon EMR, AWS Batch, and Amazon SageMaker, as well as services on other cloud platforms\. Using Apache Airflow with Amazon MWAA fully supports integration with AWS services and popular third\-party tools such as Apache Hadoop, Presto, Hive, and Spark to perform data processing tasks\. Amazon MWAA is committed to maintaining compatibility with the Amazon MWAA API, and Amazon MWAA intends to provide reliable integrations to AWS services and make them available to the community, and be involved in community feature development\.

Amazon MWAA automatically sends environment performance metrics—and if enabled—Apache Airflow logs to CloudWatch\. You can view logs and performance metrics for multiple environments from a single location to easily identify task delays or workflow errors without the need for additional third\-party tools\.

## Amazon MWAA regions<a name="regions-mwaa"></a>

Amazon MWAA is available in the following AWS Regions\.
+ Europe \(Stockholm\) \- eu\-north\-1
+ Europe \(Ireland\) \- eu\-west\-1
+ Asia Pacific \(Singapore\) \- ap\-southeast\-1
+ Asia Pacific \(Sydney\) \- ap\-southeast\-2
+ Europe \(Frankfurt\) \- eu\-central\-1
+ Asia Pacific \(Tokyo\) \- ap\-northeast\-1
+ US East \(N\. Virginia\) \- us\-east\-1
+ US East \(Ohio\) \- us\-east\-2
+ US West \(Oregon\) \- us\-west\-2

## Apache Airflow versions<a name="regions-aa-versions"></a>

The following Apache Airflow versions are supported on Amazon Managed Workflows for Apache Airflow \(MWAA\)\.


| Airflow version | Airflow guide | Airflow constraints | Python version | 
| --- | --- | --- | --- | 
|  v2\.0\.2  |  [Apache Airflow v2\.0\.2 reference guide](http://airflow.apache.org/docs/apache-airflow/2.0.2/index.html)  |  [https://raw\.githubusercontent\.com/apache/airflow/constraints\-2\.0\.2/constraints\-3\.7\.txt](https://raw.githubusercontent.com/apache/airflow/constraints-2.0.2/constraints-3.7.txt)  |  [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)  | 
|  v1\.10\.12  |  [Apache Airflow v1\.10\.12 reference guide](https://airflow.apache.org/docs/apache-airflow/1.10.12/)  |  [https://raw\.githubusercontent\.com/apache/airflow/constraints\-1\.10\.12/constraints\-3\.7\.txt](https://raw.githubusercontent.com/apache/airflow/constraints-1.10.12/constraints-3.7.txt)  |  [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)  | 

## Apache Airflow components<a name="components-aa"></a>

This section describes the number of Apache Airflow *Scheduler\(s\)* and *Workers* available for each Apache Airflow version on Amazon MWAA\.

### Schedulers<a name="airflow-versions-components-schedulers"></a>


| Airflow version | Scheduler \(default\) | Scheduler \(min\) | Scheduler \(max\) | 
| --- | --- | --- | --- | 
|  Apache Airflow v2\.0\.2  |  2  |  2  |  5  | 
|  Apache Airflow v1\.10\.12  |  1  |  1  |  1  | 

### Workers<a name="airflow-versions-components-workers"></a>


| Airflow version | Workers \(min\) | Workers \(max\) | Workers \(default\) | 
| --- | --- | --- | --- | 
|  Apache Airflow v2\.0\.2  |  1  |  25  |  10  | 
|  Apache Airflow v1\.10\.12  |  1  |  25  |  10  | 

## Amazon MWAA architecture<a name="architecture-mwaa"></a>

All of the components contained in the outer box \(below\) appear as a single Amazon MWAA environment in your account\. The Apache Airflow *Scheduler* and *Workers* are AWS Fargate \(Fargate\) containers that connect to the private subnets in the Amazon VPC that you select when you create an environment\. Amazon CloudWatch, Amazon S3, Amazon SQS, Amazon ECR, and AWS KMS are separate from Amazon MWAA and need to be accessible from the *Scheduler* and *Workers* in the Fargate containers\. Each environment has its own Apache Airflow metadatabase managed by AWS that is accessible to the *Scheduler* and *Workers* Fargate containers via a privately\-secured VPC endpoint\. 

The Apache Airflow *Web server* can be accessed either *over the Internet* by selecting the **Public network** Apache Airflow access mode, or *within your VPC* by selecting the **Private network** Apache Airflow access mode\. In both cases, access for your Apache Airflow users is controlled by the access control policy you create in AWS Identity and Access Management \(IAM\)\.

**Note**  
Multiple Apache Airflow *Schedulers* are only available with Apache Airflow v2\.0\.2 and above\. Learn more about the Apache Airflow task lifecycle at [Concepts](https://airflow.apache.org/docs/apache-airflow/stable/concepts.html#task-lifecycle) in the *Apache Airflow reference guide*\.

![\[This image shows the architecture of a Amazon MWAA environment.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-architecture.png)

## Amazon MWAA benefits<a name="benefits-mwaa"></a>

Amazon MWAA offers the following benefits:
+ **Setup** – Amazon MWAA sets up Apache Airflow for you when you create an environment using the same open\-source Airflow and user interface available from Apache\. You don't need to perform a manual setup or use custom tools to create an environment\. Amazon MWAA is not a "branch" of Airflow, nor is it just "compatible with"\. It is the exact same Apache Airflow that you can download on your own\.
+ **Scaling** – Amazon MWAA uses the Apache [Celery Executor](https://airflow.apache.org/docs/stable/executor/celery.html) to automatically scale workers as needed for your environment\. Amazon MWAA monitors the workers in your environment, and as demand increases, Amazon MWAA adds additional worker containers\. As workers free up, Amazon MWAA removes them\.
+ **Security** – Integrated support with AWS Identity and Access Management \(IAM\), including role\-based authentication and authorization for access to the Airflow user interface\. Workers assume IAM roles for easy and secure access to AWS services\. Workers and Scheduler run in your VPC for access to your resources\.
+ **Public or private access modes** – the Apache Airflow *Web server* can be exposed through a publicly\-accessible VPC endpoint over the Internet using the **Public network** Apache Airflow access mode, or through a privately\-accessible VPC endpoint in your VPC using the **Private network** Apache Airflow access mode\. In each case, these endpoints are secured by IAM and AWS SSO\.
+ **Upgrades and patches** – Amazon MWAA will provide new versions of Apache Airflow periodically\. The images for these versions will be updated and patched by the Amazon MWAA team\.
+ **Monitoring** – Amazon MWAA is integrated with CloudWatch\. The Apache Airflow logs and performance metrics data for your environment are available in a single location\. This lets you easily identify workflow errors or task delays\.
+ **Integration** – Amazon MWAA supports open\-source integrations with: Amazon Athena, AWS Batch, Amazon CloudWatch, Amazon DynamoDB, AWS DataSync, Amazon EMR, AWS Fargate, Amazon EKS, Amazon Kinesis Data Firehose, AWS Glue, AWS Lambda, Amazon Redshift, Amazon SQS, Amazon SNS, Amazon SageMaker, and Amazon S3, as well as hundreds of built\-in and community\-created operators and sensors\.
+ **Containers** – Amazon MWAA offers support for using containers to scale the worker fleet on demand and reduce scheduler outages, through AWS Fargate\. To learn more about Fargate, see [Amazon ECS on AWS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html)\. Operators that execute tasks on Amazon ECS containers, as well as Kubernetes operators that create and run pods on a Kubernetes cluster, are supported\.

## What's next?<a name="whatis-next-up"></a>
+ Learn more about the Apache Airflow versions we support and Apache Airflow components included with each version in [Apache Airflow versions on Amazon Managed Workflows for Apache Airflow \(MWAA\)](airflow-versions.md)\.