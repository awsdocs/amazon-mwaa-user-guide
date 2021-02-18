# Adding or updating DAGs<a name="configuring-dag-folder"></a>

Directed Acyclic Graphs \(DAGs\) are defined within a Python file that defines the DAG's structure as code\. You can use the AWS CLI, or the Amazon S3 console to upload DAGs to your environment\. This page describes the steps to add or update Apache Airflow DAGs on your Amazon Managed Workflows for Apache Airflow \(MWAA\) environment using a `dags` folder\.

**Topics**
+ [Prerequisites](#configuring-dag-folder-prereqs)
+ [How it works](#configuring-dag-folder-how)
+ [Uploading DAG code to Amazon S3](#configuring-dag-folder-uploading)
+ [Specifying the path to your DAGs folder on the Amazon MWAA console \(the first time\)](#configuring-dag-folder-mwaaconsole)
+ [Viewing changes on your Apache Airflow UI](#configuring-dag-folder-mwaaconsole-view)

## Prerequisites<a name="configuring-dag-folder-prereqs"></a>

**To use the steps on this page, you'll need:**

1. The required AWS resources configured for your environment as defined in [Get started with Amazon Managed Workflows for Apache Airflow \(MWAA\)](get-started.md)\.

1. An execution role with a permissions policy that grants Amazon MWAA access to the AWS services used by your environment as defined in [Amazon MWAA Execution role](mwaa-create-role.md)\.

1. An AWS account with access in AWS Identity and Access Management \(IAM\) to the Amazon S3 console, or the AWS Command Line Interface \(AWS CLI\) as defined in [Accessing an Amazon MWAA environment](access-policies.md)\.

## How it works<a name="configuring-dag-folder-how"></a>

A Directed Acyclic Graph \(DAG\) is defined within a single Python file that defines the DAG's structure as code\. It consists of the following:
+ A [DAG](https://airflow.apache.org/docs/stable/concepts.html#dags) definition\.
+ [Operators](https://airflow.apache.org/concepts.html#operators) that describe how to run the DAG and the [tasks](https://airflow.apache.org/docs/stable/concepts.html#tasks) to run\.
+ [Operator relationships](https://airflow.apache.org/concepts.html#bitshift-composition) that describe the order in which to run the tasks\.

Amazon MWAA automatically detects and syncs changes from your Amazon S3 bucket to Apache Airflow every 30 seconds\. To run an Apache Airflow platform on an Amazon MWAA environment, you need to copy your DAG definition to the `dags` folder in your storage bucket\. For example, the DAG folder in your storage bucket may look like this:

**Example DAG folder**  

```
dags/
    └ dag_def.py
```

If this is the first time you're adding the folder to your Amazon S3 bucket, you'll also need to specify the path to the folder on the Amazon MWAA console\. You only need to complete this step once\.

**Note**  
You do not need to include the `airflow.cfg` configuration file in your DAG folder\. You can override the default Apache Airflow configurations from the Amazon MWAA console\. For more information, see [Amazon MWAA Apache Airflow configuration options](configuring-env-variables.md)\.

## Uploading DAG code to Amazon S3<a name="configuring-dag-folder-uploading"></a>

You can use the Amazon S3 console or the AWS Command Line Interface \(AWS CLI\) to upload DAG code to your Amazon S3 bucket\. The following steps assume you are uploading code \(`.py`\) to a folder named `dags` in your Amazon S3 bucket\.

### Using the AWS CLI<a name="configuring-dag-folder-cli"></a>

The AWS Command Line Interface \(AWS CLI\) is an open source tool that enables you to interact with AWS services using commands in your command\-line shell\. To complete the steps in this section, you need the following:
+ [AWS CLI – Install version 2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
+ [AWS CLI – Quick configuration with `aws configure`](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)

**To upload using the AWS CLI**

1. The following command lists all Amazon S3 buckets\.

   ```
   aws s3 ls
   ```

1. The following command lists the files and folders in an Amazon S3 bucket\.

   ```
   aws s3 ls s3://your-s3-bucket-any-name
   ```

1. The following command uploads a `dag_def.py` file to a `dags` folder\. 

   ```
   aws s3 cp dag_def.py s3://your-s3-bucket-any-name/dags/
   ```

   If a folder named `dags` does not already exist on your Amazon S3 bucket, this command creates the folder and uploads the file named `dag_def.py` to the folder\.

### Using the Amazon S3 console<a name="configuring-dag-folder-console"></a>

The Amazon S3 console is a web\-based user interface that allows you to create and manage the resources in your Amazon S3 bucket\. The following steps assume you have a DAGs folder named `dags`\.

**To upload using the Amazon S3 console**

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Select the **S3 bucket** link in the **DAG code in S3** pane to open your storage bucket on the Amazon S3 console\.

1. Choose the `dags` folder\.

1. Choose **Upload**\.

1. Choose **Add file**\.

1. Select the local copy of your `dag_def.py`, choose **Upload**\.

## Specifying the path to your DAGs folder on the Amazon MWAA console \(the first time\)<a name="configuring-dag-folder-mwaaconsole"></a>

If this is the first time you're adding the folder to your Amazon S3 bucket, you'll also need to specify the path to the folder on the Amazon MWAA console\. You only need to complete this step once\.

The following steps assume you are specifying the path to a folder on your Amazon S3 bucket named `dags`\.

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose the environment where you want to run DAGs\.

1. Choose **Edit**\.

1. On the **DAG code in Amazon S3** pane, choose **Browse S3** next to the **DAG folder** field\.

1. Select your `dags` folder\.

1. Choose **Choose**\.

1. Choose **Next**, **Update environment**\.

You can begin using the new DAG immediately after your environment finishes updating\.

## Viewing changes on your Apache Airflow UI<a name="configuring-dag-folder-mwaaconsole-view"></a>

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose **Open Airflow UI** to view your Apache Airflow UI\.

**Note**  
You may need to ask your account administrator to add `AmazonMWAAWebServerAccess` permissions for your account to view your Apache Airflow UI\. For more information, see [Managing access](https://docs.aws.amazon.com/mwaa/latest/userguide/manage-access.html)\.