# Get started with Amazon Managed Workflows for Apache Airflow \(MWAA\)<a name="get-started"></a>

Amazon Managed Workflows for Apache Airflow \(MWAA\) uses the Amazon VPC, DAG code and supporting files in your Amazon S3 storage bucket to create an environment\. You specify the location of your Amazon S3 bucket, the path to your DAG code, and any custom plugins or dependencies on the Amazon MWAA console when you create an environment\. This guide describes the prerequisites and the steps to create an Amazon MWAA environment\.

**Topics**
+ [Prerequisites](#prerequisites)
+ [Create an Amazon S3 bucket for Amazon MWAA](mwaa-s3-bucket.md)
+ [Create the VPC network](vpc-create.md)
+ [Create an Amazon MWAA environment](create-environment.md)

## Prerequisites<a name="prerequisites"></a>

To use Amazon MWAA, you may need to take additional steps to ensure you have the necessary permissions to use Amazon MWAA, and that you have the required AWS resources configured for your environment\. 

To create an environment, you'll need:
+ **AWS account** – An AWS account with permission to use Amazon MWAA and the AWS services and resources used by your environment\.
+ **Amazon S3 bucket** – An Amazon S3 bucket with versioning enabled\. An Amazon S3 bucket is used to store your DAGs and associated files, such as `plugins.zip` and `requirements.txt`\.
+ **Amazon VPC** – The Amazon VPC networking components required by an Amazon MWAA environment\. You can use an existing VPC that meets these requirements, or create the VPC and networking components as defined in [Create the VPC network](vpc-create.md)\.
+ **Customer master key \(CMK\)** – A Customer master key \(CMK\) for data encryption on your environment\. You can choose the default option on the Amazon MWAA console to create an [AWS owned CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk) when you create an environment\.
+ **Execution role** – An execution role that allows Amazon MWAA to access AWS resources in your environment\. You can choose the default option on the Amazon MWAA console to create an execution role when you create an environment

You may \(optionally\) want to create other AWS resources before you create an environment:
+ **Customer master key \(CMK\)** – If you are unable to use the default option on the Amazon MWAA console to use an AWS owned CMK, you want want to create a [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk)\. To learn more, see [Customer managed CMKs for Data Encryption](custom-keys-certs.md)\.