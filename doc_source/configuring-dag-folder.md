# Adding or updating DAGs<a name="configuring-dag-folder"></a>

Directed Acyclic Graphs \(DAGs\) are defined within a Python file that defines the DAG's structure as code\. You can use the AWS CLI, or the Amazon S3 console to upload DAGs to your environment\. This page describes the steps to add or update Apache Airflow DAGs on your Amazon Managed Workflows for Apache Airflow \(MWAA\) environment using the `dags` folder in your Amazon S3 bucket\.

**Topics**
+ [Prerequisites](#configuring-dag-folder-prereqs)
+ [How it works](#configuring-dag-folder-how)
+ [What's changed in v2\.0\.2](#configuring-dag-folder-changed)
+ [Testing DAGs using the Amazon MWAA CLI utility](#working-dag-folder-cli-utility)
+ [Uploading DAG code to Amazon S3](#configuring-dag-folder-uploading)
+ [Specifying the path to your DAGs folder on the Amazon MWAA console \(the first time\)](#configuring-dag-folder-mwaaconsole)
+ [Viewing changes on your Apache Airflow UI](#configuring-dag-folder-mwaaconsole-view)
+ [What's next?](#configuring-dag-folder-next-up)

## Prerequisites<a name="configuring-dag-folder-prereqs"></a>

You'll need the following before you can complete the steps on this page\.

1. **Access**\. Your AWS account must have been granted access by your administrator to the [AmazonMWAAFullConsoleAccess](access-policies.md#console-full-access) access control policy for your environment\.

1. **Amazon S3 configurations**\. The [Amazon S3 bucket](mwaa-s3-bucket.md) used to store your DAGs, custom plugins in `plugins.zip`, and Python dependencies in `requirements.txt` must be configured with *Public Access Blocked* and *Versioning Enabled*\.

1. **Permissions**\. Your Amazon MWAA environment must be permitted by your [execution role](mwaa-create-role.md) to access the AWS resources used by your environment\.

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

**Note**  
You do not need to include the `airflow.cfg` configuration file in your DAG folder\. You can override the default Apache Airflow configurations from the Amazon MWAA console\. For more information, see [Apache Airflow configuration options](configuring-env-variables.md)\.

## What's changed in v2\.0\.2<a name="configuring-dag-folder-changed"></a>
+ **New: Operators, Hooks, and Executors**\. The import statements in your DAGs, and the custom plugins you specify in a `plugins.zip` on Amazon MWAA have changed between Apache Airflow v1\.10\.12 and Apache Airflow v2\.0\.2\. For example, `from airflow.contrib.hooks.aws_hook import AwsHook` in Apache Airflow v1\.10\.12 has changed to `from airflow.providers.amazon.aws.hooks.base_aws import AwsBaseHook` in Apache Airflow v2\.0\.2\. To learn more, see [Python API Reference](https://airflow.apache.org/docs/apache-airflow/2.0.2/python-api-ref.html) in the *Apache Airflow reference guide*\.

## Testing DAGs using the Amazon MWAA CLI utility<a name="working-dag-folder-cli-utility"></a>
+ The command line interface \(CLI\) utility replicates an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment locally\.
+ The CLI builds a Docker container image locally that’s similar to an Amazon MWAA production image\. This allows you to run a local Apache Airflow environment to develop and test DAGs, custom plugins, and dependencies before deploying to Amazon MWAA\.
+ To run the CLI, see the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

## Uploading DAG code to Amazon S3<a name="configuring-dag-folder-uploading"></a>

You can use the Amazon S3 console or the AWS Command Line Interface \(AWS CLI\) to upload DAG code to your Amazon S3 bucket\. The following steps assume you are uploading code \(`.py`\) to a folder named `dags` in your Amazon S3 bucket\.

### Using the AWS CLI<a name="configuring-dag-folder-cli"></a>

The AWS Command Line Interface \(AWS CLI\) is an open source tool that enables you to interact with AWS services using commands in your command\-line shell\. To complete the steps on this page, you need the following:
+ [AWS CLI – Install version 2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)\.
+ [AWS CLI – Quick configuration with `aws configure`](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)\.

**To upload using the AWS CLI**

1. Use the following command to list all of your Amazon S3 buckets\.

   ```
   aws s3 ls
   ```

1. Use the following command to list the files and folders in the Amazon S3 bucket for your environment\.

   ```
   aws s3 ls s3://YOUR_S3_BUCKET_NAME
   ```

1. The following command uploads a `dag_def.py` file to a `dags` folder\. 

   ```
   aws s3 cp dag_def.py s3://YOUR_S3_BUCKET_NAME/dags/
   ```

   If a folder named `dags` does not already exist on your Amazon S3 bucket, this command creates the `dags` folder and uploads the file named `dag_def.py` to the new folder\.

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

The following steps assume you are specifying the path to a folder on your Amazon S3 bucket named `dags`\.

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose the environment where you want to run DAGs\.

1. Choose **Edit**\.

1. On the **DAG code in Amazon S3** pane, choose **Browse S3** next to the **DAG folder** field\.

1. Select your `dags` folder\.

1. Choose **Choose**\.

1. Choose **Next**, **Update environment**\.

## Viewing changes on your Apache Airflow UI<a name="configuring-dag-folder-mwaaconsole-view"></a>

### Logging into Apache Airflow<a name="airflow-access-and-login"></a>

You need [Apache Airflow UI access policy: AmazonMWAAWebServerAccess](access-policies.md#web-ui-access) permissions for your AWS account in AWS Identity and Access Management \(IAM\) to view your Apache Airflow UI\. 

**To access your Apache Airflow UI**

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose **Open Airflow UI**\.

**To log\-in to your Apache Airflow UI**
+ Enter the AWS Identity and Access Management \(IAM\) user name and password for your account\.

![\[This image shows how to log-in to your Apache Airflow UI.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-aa-ui-login.png)

## What's next?<a name="configuring-dag-folder-next-up"></a>
+ Test your DAGs, custom plugins, and Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.