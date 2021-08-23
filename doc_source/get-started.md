# Get started with Amazon Managed Workflows for Apache Airflow \(MWAA\)<a name="get-started"></a>

Amazon Managed Workflows for Apache Airflow \(MWAA\) uses the Amazon VPC, DAG code and supporting files in your Amazon S3 storage bucket to create an environment\. This guide describes the prerequisites and the required AWS resources needed to get started with Amazon MWAA\.

**Topics**
+ [Prerequisites](#prerequisites)
+ [About this guide](#prerequisites-infra)
+ [Before you begin](#prerequisites-before)
+ [Available regions](#regions)
+ [Create an Amazon S3 bucket for Amazon MWAA](mwaa-s3-bucket.md)
+ [Create the VPC network](vpc-create.md)
+ [Create an Amazon MWAA environment](create-environment.md)
+ [What's next?](#mwaa-s3-bucket-next-up)

## Prerequisites<a name="prerequisites"></a>

To create an Amazon MWAA environment, you may want to take additional steps to ensure you have permission to the AWS resources you need to create\.
+ **AWS account** – An AWS account with permission to use Amazon MWAA and the AWS services and resources used by your environment\.

## About this guide<a name="prerequisites-infra"></a>

This section describes the AWS infrastructure and resources you'll create in this guide\.
+ **Amazon VPC** – The Amazon VPC networking components required by an Amazon MWAA environment\. You can configure an existing VPC that meets these requirements \(advanced\) as seen in [About networking on Amazon MWAA](networking-about.md), or create the VPC and networking components, as defined in [Create the VPC network](vpc-create.md)\.
+ **Amazon S3 bucket** – An Amazon S3 bucket to store your DAGs and associated files, such as `plugins.zip` and `requirements.txt`\. Your Amazon S3 bucket must be configured to **Block all public access**, with **Bucket Versioning** enabled, as defined in [Create an Amazon S3 bucket for Amazon MWAA](mwaa-s3-bucket.md)\. 
+ **Amazon MWAA environment** – An Amazon MWAA environment configured with the location of your Amazon S3 bucket, the path to your DAG code and any custom plugins or Python dependencies, and your Amazon VPC and its security group, as defined in [Create an Amazon MWAA environment](create-environment.md)\. 

## Before you begin<a name="prerequisites-before"></a>

To create an Amazon MWAA environment, you may want to take additional steps to create and configure other AWS resources before you create your environment\. 

To create an environment, you need the following:
+ **AWS KMS key** – An AWS KMS key for data encryption on your environment\. You can choose the default option on the Amazon MWAA console to create an [AWS owned key](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk) when you create an environment, or specify an existing [Customer managed key](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk) with permissions to other AWS services used by your environment configured \(advanced\)\. To learn more, see [Customer managed keys for Data Encryption](custom-keys-certs.md)\.
+ **Execution role** – An execution role that allows Amazon MWAA to access AWS resources in your environment\. You can choose the default option on the Amazon MWAA console to create an execution role when you create an environment\. To learn more, see [Amazon MWAA execution role](mwaa-create-role.md)\.
+ **VPC security group** – A VPC security group that allows Amazon MWAA to access other AWS resources in your VPC network\. You can choose the default option on the Amazon MWAA console to create a security group when you create an environment, or provide a security group with the appropriate inbound and outbound rules \(advanced\)\. To learn more, see [Security in your VPC on Amazon MWAA](vpc-security.md)\.

## Available regions<a name="regions"></a>

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

## What's next?<a name="mwaa-s3-bucket-next-up"></a>
+ Learn how to create an Amazon S3 bucket in [Create an Amazon S3 bucket for Amazon MWAA](mwaa-s3-bucket.md)\.