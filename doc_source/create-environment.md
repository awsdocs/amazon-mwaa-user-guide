# Create an Amazon MWAA environment<a name="create-environment"></a>

Amazon Managed Workflows for Apache Airflow \(MWAA\) sets up Apache Airflow on an environment in your chosen version using the same open\-source Airflow and user interface available from Apache\. This guide describes the steps to create an Amazon MWAA environment\.

**Contents**
+ [Before you begin](#create-environment-before)
+ [Apache Airflow versions](#create-environment-regions-aa-versions)
+ [Create an environment](#create-environment-start)
  + [Step one: Specify details](#create-environment-start-details)
  + [Step two: Configure advanced settings](#create-environment-start-advanced)
  + [Step three: Review and create](#create-environment-start-review)
+ [What's next?](#mwaa-env-next-up)

## Before you begin<a name="create-environment-before"></a>
+ The [VPC network](vpc-create.md) you specify for your environment can't be changed after the environment is created\.
+ You need an Amazon S3 bucket configured to **Block all public access**, with **Bucket Versioning** enabled\.
+ You need an AWS account with [permissions to use Amazon MWAA](manage-access.md), and permission in AWS Identity and Access Management \(IAM\) to create IAM roles\. If you choose the **Private network ** access mode for the Apache Airflow *Web server* which limits Apache Airflow access *within your Amazon VPC*, you'll need permission in IAM to create Amazon VPC endpoints\.

## Apache Airflow versions<a name="create-environment-regions-aa-versions"></a>

The following Apache Airflow versions are supported on Amazon Managed Workflows for Apache Airflow \(MWAA\)\.


| Airflow version | Airflow guide | Airflow constraints | Python version | 
| --- | --- | --- | --- | 
|  v2\.0\.2  |  [Apache Airflow v2\.0\.2 reference guide](http://airflow.apache.org/docs/apache-airflow/2.0.2/index.html)  |  [https://raw\.githubusercontent\.com/apache/airflow/constraints\-2\.0\.2/constraints\-3\.7\.txt](https://raw.githubusercontent.com/apache/airflow/constraints-2.0.2/constraints-3.7.txt)  |  [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)  | 
|  v1\.10\.12  |  [Apache Airflow v1\.10\.12 reference guide](https://airflow.apache.org/docs/apache-airflow/1.10.12/)  |  [https://raw\.githubusercontent\.com/apache/airflow/constraints\-1\.10\.12/constraints\-3\.7\.txt](https://raw.githubusercontent.com/apache/airflow/constraints-1.10.12/constraints-3.7.txt)  |  [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)  | 

## Create an environment<a name="create-environment-start"></a>

The following section describes the steps to create an Amazon MWAA environment\.

### Step one: Specify details<a name="create-environment-start-details"></a>

**To specify details for the environment**

1. Open the [Amazon MWAA](https://console.aws.amazon.com/mwaa/home/) console\.

1. Use the AWS Region selector to select your region\.

1. Choose **Create environment**\.

1. On the **Specify details** page, under **Environment details**:

   1. Type a unique name for your environment in **Name**\.

   1. Choose the Apache Airflow version in **Airflow version**\. 
**Note**  
If no value is specified, defaults to the latest Airflow version\.

1. Under **DAG code in Amazon S3**:

   1. **S3 Bucket**\. Choose **Browse S3** and select your Amazon S3 bucket, or enter the Amazon S3 URI\.

   1. **DAGs folder**\. Choose **Browse S3** and select the `dags` folder in your Amazon S3 bucket, or enter the Amazon S3 URI\.

   1. **Plugins file \- optional**\. Choose **Browse S3** and select the `plugins.zip` file on your Amazon S3 bucket, or enter the Amazon S3 URI\.

   1. **Requirements file \- optional**\. Choose **Browse S3** and select the `requirements.txt` file on your Amazon S3 bucket, or enter the Amazon S3 URI\.

1. Choose **Next**\. 

### Step two: Configure advanced settings<a name="create-environment-start-advanced"></a>

**To configure advanced settings**

1. On the **Configure advanced settings** page, under **Networking**:

   1. Choose your [Amazon VPC](vpc-create.md)\.

     This step populates two of the private subnets in your Amazon VPC\.

1. Under **Web server access**, select your preferred [Apache Airflow access mode](configuring-networking.md):

   1. **Private network**\. This limits access of the Apache Airflow UI to users *within your Amazon VPC* that have been granted access to the [IAM policy for your environment](access-policies.md)\. You need permission to create Amazon VPC endpoints for this step\.
**Note**  
If you choose the **Private network** Apache Airflow access mode, you need to create a mechanism to access your Apache Airflow *Web server* in your Amazon VPC\. To learn more, see [Accessing the VPC endpoint for your Apache Airflow Web server \(private network access\)](vpc-vpe-access.md#vpc-vpe-access-endpoints)\.

   1. **Public network**\. This allows the Apache Airflow UI to be accessed *over the Internet* by users granted access to the [IAM policy for your environment](access-policies.md)\.

1. Under **Security group\(s\)**, choose the security group used to secure your [Amazon VPC](vpc-create.md):

   1. By default, Amazon MWAA creates a security group in your Amazon VPC with specific inbound and outbound rules in **Create new security group**\.

   1. **Optional**\. Deselect the check box in **Create new security group** to select up to 5 security groups\.
**Note**  
An existing Amazon VPC security group must be configured with specific inbound and outbound rules to allow network traffic\. To learn more, see [Security in your VPC on Amazon MWAA](vpc-security.md)\.

1. Under **Environment class**, choose an [environment class](environment-class.md)\.

   We recommend choosing the smallest size necessary to support your workload\. You can change the environment class at any time\.

1. For **Maximum worker count**, specify the maximum number of Apache Airflow workers to run in the environment\.

   To learn more, see [Example high performance use case](mwaa-autoscaling.md#mwaa-autoscaling-high-volume)\.

1. Under **Encryption**, choose a data encryption option:

   1. By default, Amazon MWAA uses an AWS owned key to encrypt your data\.

   1. **Optional**\. Choose **Customize encryption settings \(advanced\)** to choose a different AWS KMS key \(such as the Amazon S3 key you're using for server\-side encryption on your bucket\), or enter the ARN\.
**Note**  
You must have permissions to the key to select it on the Amazon MWAA console\. You must also grant permissions for Amazon MWAA to use the key by attaching the policy described in [Attach key policy](custom-keys-certs.md#custom-keys-certs-grant-policies-attach)\.

1. **Recommended**\. Under **Monitoring**, choose one or more log categories for **Airflow logging configuration** to send Apache Airflow logs to CloudWatch Logs:

   1. **Airflow task logs**\. Choose the type of Apache Airflow task logs to send to CloudWatch Logs in **Log level**\.

   1. **Airflow web server logs**\. Choose the type of Apache Airflow web server logs to send to CloudWatch Logs in **Log level**\.

   1. **Airflow scheduler logs**\. Choose the type of Apache Airflow scheduler logs to send to CloudWatch Logs in **Log level**\.

   1. **Airflow worker logs**\. Choose the type of Apache Airflow worker logs to send to CloudWatch Logs in **Log level**\.

   1. **Airflow DAG processing logs**\. Choose the type of Apache Airflow DAG processing logs to send to CloudWatch Logs in **Log level**\.

1. **Optional**\. For **Airflow configuration options**, choose **Add custom configuration option**\.

   You can choose from the suggested dropdown list of [Apache Airflow configuration options](configuring-env-variables.md) for your Apache Airflow version, or specify custom configuration options\. For example, `core.default_task_retries` : `3`\.

1. **Optional**\. Under **Tags**, choose **Add new tag** to associate tags to your environment\. For example, `Environment`: `Staging`\.

1. Under **Permissions**, choose an execution role:

   1. By default, Amazon MWAA creates an [execution role](mwaa-create-role.md) in **Create a new role**\. You must have permission to create IAM roles to use this option\.

   1. **Optional**\. Choose **Enter role ARN** to enter the Amazon Resource Name \(ARN\) of an existing execution role\.

1. Choose **Next**\.

### Step three: Review and create<a name="create-environment-start-review"></a>

**To review an environment summary**
+ Review the environment summary, choose **Create environment**\.
**Note**  
It takes about twenty to thirty minutes to create an environment\.

## What's next?<a name="mwaa-env-next-up"></a>
+ Learn how to grant users access to your Apache Airflow *Web server* and Amazon MWAA environment in [Managing access to an Amazon MWAA environment](manage-access.md)\.
+ Learn how to access the VPC endpoint for your Apache Airflow *Web server* \(private network access\) in [Managing access to VPC endpoints on Amazon MWAA](vpc-vpe-access.md)\.