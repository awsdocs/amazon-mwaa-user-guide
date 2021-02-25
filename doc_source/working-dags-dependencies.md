# Installing Python dependencies<a name="working-dags-dependencies"></a>

An extra package is a Python subpackage that is not included in the Apache Airflow base install on your Amazon Managed Workflows for Apache Airflow \(MWAA\) environment\. It is referred to throughout this page as a Python dependency\. This page describes the steps to install Apache Airflow [Extra packages](https://airflow.apache.org/docs/stable/installation.html#extra-packages) on your Amazon MWAA environment using a `requirements.txt` file\.

**Topics**
+ [Prerequisites](#working-dags-dependencies-prereqs)
+ [How it works](#working-dags-dependencies-how)
+ [Creating a `requirements.txt`](#working-dags-dependencies-syntax-create)
+ [Example requirements\.txt for Apache Hive](#working-dags-dependencies-example-install)
+ [Uploading `requirements.txt` to Amazon S3](#configuring-dag-dependencies-upload)
+ [Specifying the path to `requirements.txt` on the Amazon MWAA console \(the first time\)](#configuring-dag-dependencies-first)
+ [Specifying the `requirements.txt` version on the Amazon MWAA console](#working-dags-dependencies-mwaaconsole-version)
+ [Viewing logs for your `requirements.txt`](#working-dags-dependencies-logs)
+ [Viewing changes on your Apache Airflow UI](#configuring-dag-dependencies-mwaaconsole-view)

## Prerequisites<a name="working-dags-dependencies-prereqs"></a>

**To use the steps on this page, you'll need:**

1. The required AWS resources configured for your environment as defined in [Get started with Amazon Managed Workflows for Apache Airflow \(MWAA\)](get-started.md)\.

1. An execution role with a permissions policy that grants Amazon MWAA access to the AWS resources used by your environment as defined in [Amazon MWAA Execution role](mwaa-create-role.md)\.

1. An AWS account with access in AWS Identity and Access Management \(IAM\) to the Amazon S3 console, or the AWS Command Line Interface \(AWS CLI\) as defined in [Accessing an Amazon MWAA environment](access-policies.md)\.

## How it works<a name="working-dags-dependencies-how"></a>

Amazon MWAA runs `pip3 install -r requirements.txt` on the requirements file that you specify for your environment for each of the Apache Airflow Scheduler and Workers\.

To run Python dependencies on your environment, you must do three things:

1. Create a `requirements.txt` file locally\.

1. Upload the local `requirements.txt` to your Amazon S3 bucket\.

1. Specify the version of this file in the **Requirements file** field on the Amazon MWAA console\.

**Note**  
If this is the first time you're creating and uploading a `requirements.txt` to your Amazon S3 bucket, you'll also need to specify the path to the file on the Amazon MWAA console\. You only need to complete this step once\.

## Creating a `requirements.txt`<a name="working-dags-dependencies-syntax-create"></a>

If your Apache Airflow platform uses [Extra packages](https://airflow.apache.org/docs/stable/installation.html#extra-packages), specify the names of the packages and their required dependencies in your `requirements.txt`\.

We recommend always specifying either a specific version \(`==`\) or a maximum version \(`>=`\) for the dependencies in your `requirements.txt` file\. This helps to prevent a future breaking update from [PyPi\.org](http://pypi.org/) from being automatically applied\. We also recommend verifying whether you need to add any dependent packages\.

**To find the package version and its dependencies**

1. Open the [Documentation](https://airflow.apache.org/docs/) page in the *Apache Airflow reference guide*\.

1. Choose a package\.

1. Verify the required dependencies in **Pip requirements** are specified in your `requirements.txt` file\.

1. Apache Airflow provides a list of packages typically used with the current package in **Cross provider package dependencies**\. Identify any other dependencies you may want to specify in your `requirements.txt` file\.

## Example requirements\.txt for Apache Hive<a name="working-dags-dependencies-example-install"></a>

The following example `requirements.txt` file installs [Apache Hive](https://airflow.apache.org/docs/apache-airflow-providers-apache-hive/stable/index.html) and its required dependencies\.

Open the [Apache Hive](https://airflow.apache.org/docs/apache-airflow-providers-apache-hive/stable/index.html) page in the *Apache Airflow reference guide*\. The package lists three required dependencies in **PIP requirements** which we've added to our `requirements.txt` file\.

```
apache-airflow-providers-apache-hive[amazon]>=1.0.1
hmsclient>=0.1.0
pyhive[hive]>=0.6.0
thrift>=0.9.2
```

You may want to specify other dependencies listed in **Cross provider package dependencies**\. For example, if you want to use Presto with Apache Hive in your DAGs, you need to add [https://airflow.apache.org/docs/apache-airflow-providers-presto/stable/index.html](https://airflow.apache.org/docs/apache-airflow-providers-presto/stable/index.html) and its required dependencies in the **Pip requirements** list\.

```
apache-airflow-providers-apache-hive[amazon]>=1.0.1
hmsclient>=0.1.0
pyhive[hive]>=0.6.0
thrift>=0.9.2
apache-airflow-providers-presto>=1.0.1
presto-python-client>=0.7.0
```

## Uploading `requirements.txt` to Amazon S3<a name="configuring-dag-dependencies-upload"></a>

You can use the Amazon S3 console or the AWS Command Line Interface \(AWS CLI\) to upload a `requirements.txt` file to your Amazon S3 bucket\.

### Using the AWS CLI<a name="configuring-dag-dependencies-upload-cli"></a>

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

1. The following command uploads a `requirements.txt` file to an Amazon S3 bucket\.

   ```
   aws s3 cp requirements.txt s3://your-s3-bucket-any-name/requirements.txt
   ```

### Using the Amazon S3 console<a name="configuring-dag-dependencies-upload-console"></a>

The Amazon S3 console is a web\-based user interface that allows you to create and manage the resources in your Amazon S3 bucket\.

**To upload using the Amazon S3 console**

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Select the **S3 bucket** link in the **DAG code in S3** pane to open your storage bucket on the Amazon S3 console\.

1. Choose **Upload**\.

1. Choose **Add file**\.

1. Select the local copy of your `requirements.txt`, choose **Upload**\.

## Specifying the path to `requirements.txt` on the Amazon MWAA console \(the first time\)<a name="configuring-dag-dependencies-first"></a>

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose **Edit**\.

1. On the **DAG code in Amazon S3** pane, choose **Browse S3** next to the **Requirements file \- optional** field\.

1. Select the `requirements.txt` file on your Amazon S3 bucket\.

1. Choose **Choose**\.

1. Choose **Next**, **Update environment**\.

You can begin using the new packages immediately after your environment finishes updating\.

## Specifying the `requirements.txt` version on the Amazon MWAA console<a name="working-dags-dependencies-mwaaconsole-version"></a>

You need to specify the version of your `requirements.txt` file on the Amazon MWAA console each time you upload a new version of your `requirements.txt` in your Amazon S3 bucket\. 

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose **Edit**\.

1. On the **DAG code in Amazon S3** pane, choose a `requirements.txt` version in the dropdown list\.

1. Choose **Next**, **Update environment**\.

You can begin using the new packages immediately after your environment finishes updating\.

## Viewing logs for your `requirements.txt`<a name="working-dags-dependencies-logs"></a>

If you've enabled Apache Airflow logs for your environment, you can view logs in Amazon CloudWatch to ensure your packages were installed successfully\. 

1. Open the [Logs groups page](https://console.aws.amazon.com/cloudwatch/home#logsV2:log-groups) on the CloudWatch console\.

1. Choose the log group name for your scheduler\. For example, `airflow-YOUR_ENVIRONMENT_NAME-Scheduler`\.

1. Choose the `requirements_install_ip` log in **Log streams**\.

1. Review the list of packages and whether any of these encountered an error during installation\.

**Note**  
If a package in your `requirements.txt` is not available on [PyPi\.org](http://pypi.org/), then the installation will fail with no logging\.

## Viewing changes on your Apache Airflow UI<a name="configuring-dag-dependencies-mwaaconsole-view"></a>

**To access your Apache Airflow UI**

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose **Open Airflow UI** to view your Apache Airflow UI\.

**Note**  
You may need to ask your account administrator to add `AmazonMWAAWebServerAccess` permissions for your account to view your Apache Airflow UI\. For more information, see [Managing access](https://docs.aws.amazon.com/mwaa/latest/userguide/manage-access.html)\.