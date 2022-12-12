# What Is Amazon Managed Workflows for Apache Airflow \(MWAA\)?<a name="what-is-mwaa"></a>

Amazon Managed Workflows for Apache Airflow \(MWAA\) is a managed orchestration service for [Apache Airflow](https://airflow.apache.org/) that makes it easier to setup and operate end\-to\-end data pipelines in the cloud at scale\. Apache Airflow is an open\-source tool used to programmatically author, schedule, and monitor sequences of processes and tasks referred to as "workflows\." With Amazon MWAA, you can use Airflow and Python to create workflows without having to manage the underlying infrastructure for scalability, availability, and security\. Amazon MWAA automatically scales its workflow execution capacity to meet your needs, and is integrated with AWS security services to help provide you with fast and secure access to your data\.

**Topics**
+ [Features](#benefits-mwaa)
+ [Architecture](#architecture-mwaa)
+ [Integration](#integrations-mwaa)
+ [Region availability](#regions-mwaa)
+ [Supported versions](#versions-support)
+ [What's next?](#whatis-next-up)

## Features<a name="benefits-mwaa"></a>
+ **Automatic Airflow setup** – Quickly setup Apache Airflow by choosing an [Apache Airflow version](airflow-versions.md) when you create an Amazon MWAA environment\. Amazon MWAA sets up Apache Airflow for you using the same Apache Airflow user interface and open\-source code that you can download on the Internet\.
+ **Automatic scaling** – Automatically scale Apache Airflow *Workers* by setting the minimum and maximum number of *Workers* that run in your environment\. Amazon MWAA monitors the *Workers* in your environment and uses its [autoscaling component](mwaa-autoscaling.md) to add *Workers* to meet demand, up to and until it reaches the maximum number of *Workers* you defined\.
+ **Built\-in authentication** – Enable role\-based authentication and authorization for your Apache Airflow *Web server* by defining the [access control policies](environment-class.md) in AWS Identity and Access Management \(IAM\)\. The Apache Airflow *Workers* assume these policies for secure access to AWS services\.
+ **Built\-in security** – The Apache Airflow *Workers* and *Schedulers* run in [Amazon MWAA's Amazon VPC](vpc-vpe-access.md)\. Data is also automatically encrypted using AWS Key Management Service, so your environment is secure by default\.
+ **Public or private access modes** – Access your Apache Airflow *Web server* using a private, or public [access mode](configuring-networking.md)\. The **Public network** access mode uses a VPC endpoint for your Apache Airflow *Web server* that is accessible *over the Internet*\. The **Private network** access mode uses a VPC endpoint for your Apache Airflow *Web server* that is accessible *in your VPC*\. In both cases, access for your Apache Airflow users is controlled by the access control policy you define in AWS Identity and Access Management \(IAM\), and AWS SSO\.
+ **Streamlined upgrades and patches** – Amazon MWAA provides new versions of Apache Airflow periodically\. The Amazon MWAA team will update and patch the images for these versions\.
+ **Workflow monitoring** – View Apache Airflow logs and [Apache Airflow metrics](cw-metrics.md) in Amazon CloudWatch to identify Apache Airflow task delays or workflow errors without the need for additional third\-party tools\. Amazon MWAA automatically sends environment metrics—and if enabled—Apache Airflow logs to CloudWatch\.
+ **AWS integration** – Amazon MWAA supports open\-source integrations with Amazon Athena, AWS Batch, Amazon CloudWatch, Amazon DynamoDB, AWS DataSync, Amazon EMR, AWS Fargate, Amazon EKS, Amazon Kinesis Data Firehose, AWS Glue, AWS Lambda, Amazon Redshift, Amazon SQS, Amazon SNS, Amazon SageMaker, and Amazon S3, as well as hundreds of built\-in and community\-created operators and sensors\.
+ **Worker fleets** – Amazon MWAA offers support for using containers to scale the worker fleet on demand and reduce scheduler outages using [Amazon ECS on AWS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html)\. Operators that invoke tasks on Amazon ECS containers, and Kubernetes operators that create and run pods on a Kubernetes cluster are supported\.

## Architecture<a name="architecture-mwaa"></a>

All of the components contained in the outer box \(in the image below\) appear as a single Amazon MWAA environment in your account\. The Apache Airflow *Scheduler* and **Workers** are AWS Fargate \(Fargate\) containers that connect to the private subnets in the Amazon VPC for your environment\. Each environment has its own Apache Airflow metadatabase managed by AWS that is accessible to the *Scheduler* and **Workers** Fargate containers via a privately\-secured VPC endpoint\.

Amazon CloudWatch, Amazon S3, Amazon SQS, Amazon ECR, and AWS KMS are separate from Amazon MWAA and need to be accessible from the Apache Airflow *Scheduler\(s\)* and **Workers** in the Fargate containers\. 

The Apache Airflow *Web server* can be accessed either *over the Internet* by selecting the **Public network** Apache Airflow access mode, or *within your VPC* by selecting the **Private network** Apache Airflow access mode\. In both cases, access for your Apache Airflow users is controlled by the access control policy you define in AWS Identity and Access Management \(IAM\)\.

**Note**  
Multiple Apache Airflow *Schedulers* are only available with Apache Airflow v2 and above\. Learn more about the Apache Airflow task lifecycle at [Concepts](https://airflow.apache.org/docs/apache-airflow/stable/concepts.html#task-lifecycle) in the *Apache Airflow reference guide*\.

![\[This image shows the architecture of an Amazon MWAA environment.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-architecture.png)

## Integration<a name="integrations-mwaa"></a>

The active and growing Apache Airflow open\-source community provides operators \(plugins that simplify connections to services\) for Apache Airflow to integrate with AWS services\. This includes services such as Amazon S3, Amazon Redshift, Amazon EMR, AWS Batch, and Amazon SageMaker, as well as services on other cloud platforms\. 

Using Apache Airflow with Amazon MWAA fully supports integration with AWS services and popular third\-party tools such as Apache Hadoop, Presto, Hive, and Spark to perform data processing tasks\. Amazon MWAA is committed to maintaining compatibility with the Amazon MWAA API, and Amazon MWAA intends to provide reliable integrations to AWS services and make them available to the community, and be involved in community feature development\.

For sample code, see [Code examples for Amazon Managed Workflows for Apache Airflow \(MWAA\)](sample-code.md)\.

## Region availability<a name="regions-mwaa"></a>

Amazon MWAA is available in the following AWS Regions\.
+ Europe \(Stockholm\) \- eu\-north\-1
+ Europe \(Frankfurt\) \- eu\-central\-1
+ Europe \(Ireland\) \- eu\-west\-1
+ Europe \(London\) \- eu\-west\-2
+ Europe \(Paris\) \- eu\-west\-3
+ Asia Pacific \(Mumbai\) \- ap\-south\-1
+ Asia Pacific \(Singapore\) \- ap\-southeast\-1
+ Asia Pacific \(Sydney\) \- ap\-southeast\-2
+ Asia Pacific \(Tokyo\) \- ap\-northeast\-1
+ Asia Pacific \(Seoul\) \- ap\-northeast\-2
+ US East \(N\. Virginia\) \- us\-east\-1
+ US East \(Ohio\) \- us\-east\-2
+ US West \(Oregon\) \- us\-west\-2
+ Canada \(Central\) \- ca\-central\-1
+ South America \(São Paulo\) \- sa\-east\-1

## Supported versions<a name="versions-support"></a>

Amazon MWAA supports multiple versions of Apache Airflow\. To learn more about the Apache Airflow versions we support and the Apache Airflow components included with each version, see [Apache Airflow versions on Amazon Managed Workflows for Apache Airflow \(MWAA\)](airflow-versions.md)\.

## What's next?<a name="whatis-next-up"></a>
+ Get started quickly with a single AWS CloudFormation template that creates an Amazon S3 bucket for your Airflow DAGs and supporting files, an Amazon VPC with public routing, and an Amazon MWAA environment in [Quick start tutorial for Amazon Managed Workflows for Apache Airflow \(MWAA\)](quick-start.md)\.
+ Get started incrementally by creating an Amazon S3 bucket for your Airflow DAGs and supporting files, choosing from one of three Amazon VPC networking options, and creating an Amazon MWAA environment in [Get started with Amazon Managed Workflows for Apache Airflow \(MWAA\)](get-started.md)\.