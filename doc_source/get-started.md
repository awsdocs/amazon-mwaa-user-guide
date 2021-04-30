# Get started with Amazon Managed Workflows for Apache Airflow \(MWAA\)<a name="get-started"></a>

Amazon Managed Workflows for Apache Airflow \(MWAA\) uses the Amazon VPC, DAG code and supporting files in your Amazon S3 storage bucket to create an environment\. You specify the location of your Amazon S3 bucket, the path to your DAG code, and any custom plugins or dependencies on the Amazon MWAA console when you create an environment\. This guide describes the prerequisites and the required AWS resources needed to get started with Amazon Managed Workflows for Apache Airflow \(MWAA\)\.

**Topics**
+ [Before you begin](#prerequisites)
+ [Regions](#regions)
+ [Create an Amazon S3 bucket for Amazon MWAA](mwaa-s3-bucket.md)
+ [Create the VPC network](vpc-create.md)
+ [Create an Amazon MWAA environment](create-environment.md)

## Before you begin<a name="prerequisites"></a>

To use Amazon MWAA, you may need to take additional steps to ensure you have the necessary permissions to use Amazon MWAA, and that you have the required AWS resources configured for your environment\. 

To create an environment, you'll need:
+ **AWS account** – An AWS account with permission to use Amazon MWAA and the AWS services and resources used by your environment\.
+ **Amazon S3 bucket** – An Amazon S3 bucket with versioning enabled, and public access blocked\. An Amazon S3 bucket is used to store your DAGs and associated files, such as `plugins.zip` and `requirements.txt`\.
+ **Amazon VPC** – The Amazon VPC networking components required by an Amazon MWAA environment\. You can use an existing VPC that meets these requirements \(advanced\), or create the VPC and networking components as defined in [Create the VPC network](vpc-create.md)\.
+ **Customer master key \(CMK\)** – A Customer master key \(CMK\) for data encryption on your environment\. You can choose the default option on the Amazon MWAA console to create an [AWS owned CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk) when you create an environment, or specify an existing [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk) with permissions to other AWS services used by your environment configured \(advanced\)\.
+ **Execution role** – An execution role that allows Amazon MWAA to access AWS resources in your environment\. You can choose the default option on the Amazon MWAA console to create an execution role when you create an environment\.
+ **VPC security group** – A VPC security group that allows Amazon MWAA to access other AWS resources in your VPC network\. You can choose the default option on the Amazon MWAA console to create a security group when you create an environment, or provide a security group with the appropriate inbound and outbound rules \(advanced\)\.

You may \(optionally\) want to create other AWS resources before you create an environment:
+ **Customer master key \(CMK\)** – If you are unable to use the default option on the Amazon MWAA console to use an [AWS owned CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk), you may want to create a [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk) and configure permissions to other AWS services used by your environment \(advanced\)\. To learn more, see [Customer managed CMKs for Data Encryption](custom-keys-certs.md)\.

## Regions<a name="regions"></a>

Amazon MWAA is available in the following AWS Regions\.
+ Europe \(Stockholm\) \- eu\-north\-1
+ Europe \(Ireland\) \- eu\-west\-1
+ Asia Pacific \(Singapore\) \- ap\-southeast\-1
+ Asia Pacific \(Sydney\) \- ap\-southeast\-2
+ Europe \(Frankfurt\) \- eu\-central\-1
+ Asia Pacific \(Tokyo\) \- ap\-northeast\-1
+ US East \(N\. Virginia\) \- us\-east\-1
+ US East \(Ohio\) \- us\-east\-1
+ US West \(Oregon\) \- us\-west\-2