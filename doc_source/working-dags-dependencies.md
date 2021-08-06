# Installing Python dependencies<a name="working-dags-dependencies"></a>

A Python dependency is any package or distribution that is not included in the Apache Airflow base install for your Apache Airflow version on your Amazon Managed Workflows for Apache Airflow \(MWAA\) environment\. This page describes the steps to install Apache Airflow Python dependencies on your Amazon MWAA environment using a `requirements.txt` file in your Amazon S3 bucket\.

**Contents**
+ [Prerequisites](#working-dags-dependencies-prereqs)
+ [How it works](#working-dags-dependencies-how)
+ [What's changed in v2\.0\.2](#working-dags-dependencies-changed)
+ [Python dependencies overview](#working-dags-dependencies-overview)
  + [Python dependencies location and size limits](#working-dags-dependencies-quota)
+ [Creating a requirements\.txt file](#working-dags-dependencies-test-create)
  + [Step one: Test Python dependencies using the Amazon MWAA CLI utility](#working-dags-dependencies-cli-utility)
  + [Step two: Create the `requirements.txt`](#working-dags-dependencies-syntax-create)
+ [Uploading `requirements.txt` to Amazon S3](#configuring-dag-dependencies-upload)
  + [Using the AWS CLI](#configuring-dag-dependencies-upload-cli)
  + [Using the Amazon S3 console](#configuring-dag-dependencies-upload-console)
+ [Installing Python dependencies on your environment](#configuring-dag-dependencies-installing)
  + [Specifying the path to `requirements.txt` on the Amazon MWAA console \(the first time\)](#configuring-dag-dependencies-first)
  + [Specifying the `requirements.txt` version on the Amazon MWAA console](#working-dags-dependencies-mwaaconsole-version)
+ [Viewing logs for your `requirements.txt`](#working-dags-dependencies-logs)
+ [What's next?](#working-dags-dependencies-next-up)

## Prerequisites<a name="working-dags-dependencies-prereqs"></a>

You'll need the following before you can complete the steps on this page\.

1. **Access**\. Your AWS account must have been granted access by your administrator to the [AmazonMWAAFullConsoleAccess](access-policies.md#console-full-access) access control policy for your environment\.

1. **Amazon S3 configurations**\. The [Amazon S3 bucket](mwaa-s3-bucket.md) used to store your DAGs, custom plugins in `plugins.zip`, and Python dependencies in `requirements.txt` must be configured with *Public Access Blocked* and *Versioning Enabled*\.

1. **Permissions**\. Your Amazon MWAA environment must be permitted by your [execution role](mwaa-create-role.md) to access the AWS resources used by your environment\.

## How it works<a name="working-dags-dependencies-how"></a>

On Amazon MWAA, you install all Python dependencies by uploading a `requirements.txt` file to your Amazon S3 bucket, then specifying the version of the file on the Amazon MWAA console each time you update the file\. Amazon MWAA runs `pip3 install -r requirements.txt` to install the Python dependencies on the Apache Airflow scheduler and each of the workers\.

To run Python dependencies on your environment, you must do three things:

1. Create a `requirements.txt` file locally\.

1. Upload the local `requirements.txt` to your Amazon S3 bucket\.

1. Specify the version of this file in the **Requirements file** field on the Amazon MWAA console\.

**Note**  
If this is the first time you're creating and uploading a `requirements.txt` to your Amazon S3 bucket, you also need to specify the path to the file on the Amazon MWAA console\. You only need to complete this step once\.

## What's changed in v2\.0\.2<a name="working-dags-dependencies-changed"></a>
+ **New: Airflow package extras**\. The Python dependencies that you specify in a `requirements.txt` on Amazon MWAA have changed in Apache Airflow v2\.0\.2\. For example, the [core extras](http://airflow.apache.org/docs/apache-airflow/2.0.2/extra-packages-ref.html#core-airflow-extras), [provider extras](http://airflow.apache.org/docs/apache-airflow/2.0.2/extra-packages-ref.html#providers-extras), [locally installed software extras](http://airflow.apache.org/docs/apache-airflow/2.0.2/extra-packages-ref.html#locally-installed-software-extras), [external service extras](http://airflow.apache.org/docs/apache-airflow/2.0.2/extra-packages-ref.html#external-services-extras), ["other" extras](http://airflow.apache.org/docs/apache-airflow/2.0.2/extra-packages-ref.html#other-extras), [bundle extras](http://airflow.apache.org/docs/apache-airflow/2.0.2/extra-packages-ref.html#bundle-extras), [doc extras](http://airflow.apache.org/docs/apache-airflow/2.0.2/extra-packages-ref.html#doc-extras), and [software extras](http://airflow.apache.org/docs/apache-airflow/2.0.2/extra-packages-ref.html#apache-software-extras) have changed\. To view a list of the packages installed for Apache Airflow v2\.0\.2 on Amazon MWAA, see [https://github\.com/aws/aws\-mwaa\-local\-runner/blob/main/docker/config/requirements\.txt](https://github.com/aws/aws-mwaa-local-runner/blob/main/docker/config/requirements.txt)\.

## Python dependencies overview<a name="working-dags-dependencies-overview"></a>

You can install Apache Airflow extras and other Python dependencies from the Python Package Index \(PyPi\.org\), Python wheels \(`.whl`\), or Python dependencies hosted on a private PyPi/PEP\-503 Compliant Repo on your environment\. 

### Python dependencies location and size limits<a name="working-dags-dependencies-quota"></a>

The Apache Airflow *Scheduler* and the *Workers* look for custom plugins during startup on the AWS\-managed Fargate container for your environment at `/usr/local/airflow/requirements/requirements.txt`\. 
+ **Size limit**\. We recommend a `requirements.txt` file that references libraries whose combined size is less than than 1 GB\. The more libraries Amazon MWAA needs to install, the longer the *startup* time on an environment\. Although Amazon MWAA doesn't limit the size of installed libraries explicitly, if dependencies can't be installed within ten minutes, the Fargate service will time\-out and attempt to rollback the environment to a stable state\.

**Note**  
For security reasons, the Apache Airflow *Web server* on Amazon MWAA has limited network egress, and does not install plugins nor Python dependencies directly on the *Web server*\.

## Creating a requirements\.txt file<a name="working-dags-dependencies-test-create"></a>

The following steps describe the steps we recommend to create a requirements\.txt file locally\.

### Step one: Test Python dependencies using the Amazon MWAA CLI utility<a name="working-dags-dependencies-cli-utility"></a>
+ The command line interface \(CLI\) utility replicates an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment locally\.
+ The CLI builds a Docker container image locally that’s similar to an Amazon MWAA production image\. This allows you to run a local Apache Airflow environment to develop and test DAGs, custom plugins, and dependencies before deploying to Amazon MWAA\.
+ To run the CLI, see the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

### Step two: Create the `requirements.txt`<a name="working-dags-dependencies-syntax-create"></a>

The following section describes how to specify Python dependencies from the [Python Package Index](https://pypi.org/) in a `requirements.txt` file\.

------
#### [ Airflow v2\.0\.2 ]

1. **Test locally**\. Add additional libraries iteratively to find the right combination of packages and their versions, before creating a `requirements.txt` file\. To run the Amazon MWAA CLI utility, see the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

1. **Review the Airflow package extras**\. For example, the [core extras](http://airflow.apache.org/docs/apache-airflow/2.0.2/extra-packages-ref.html#core-airflow-extras), [provider extras](http://airflow.apache.org/docs/apache-airflow/2.0.2/extra-packages-ref.html#providers-extras), [locally installed software extras](http://airflow.apache.org/docs/apache-airflow/2.0.2/extra-packages-ref.html#locally-installed-software-extras), [external service extras](http://airflow.apache.org/docs/apache-airflow/2.0.2/extra-packages-ref.html#external-services-extras), ["other" extras](http://airflow.apache.org/docs/apache-airflow/2.0.2/extra-packages-ref.html#other-extras), [bundle extras](http://airflow.apache.org/docs/apache-airflow/2.0.2/extra-packages-ref.html#bundle-extras), [doc extras](http://airflow.apache.org/docs/apache-airflow/2.0.2/extra-packages-ref.html#doc-extras), and [software extras](http://airflow.apache.org/docs/apache-airflow/2.0.2/extra-packages-ref.html#apache-software-extras) have changed\. To view a list of the packages installed for Apache Airflow v2\.0\.2 on Amazon MWAA, see [https://github\.com/aws/aws\-mwaa\-local\-runner/blob/main/docker/config/requirements\.txt](https://github.com/aws/aws-mwaa-local-runner/blob/main/docker/config/requirements.txt)\.

1. **Add the constraints file**\. Add the constraints file for Apache Airflow v2\.0\.2 to the top of your `requirements.txt` file\. If the constraints file determines that `xyz==1.0` package is not compatible with other packages on your environment, the `pip3 install` will fail to prevent incompatible libraries from being installed to your environment\. 

   ```
   --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-2.0.2/constraints-3.7.txt"
   ```

1. **Apache Airflow packages**\. Add the [Apache Airflow v2\.0\.2 package extras](http://airflow.apache.org/docs/apache-airflow/2.0.2/extra-packages-ref.html) and the version \(`==`\)\. This helps to prevent packages of the same name, but different version, from being installed on your environment\.

   ```
   apache-airflow[package-extra]==2.0.2
   ```

1. **Python libraries**\. Add the package name and the version \(`==`\) in your `requirements.txt` file\. This helps to prevent a future breaking update from [PyPi\.org](https://pypi.org) from being automatically applied\.

   ```
   library == version
   ```  
**Example Boto3 and psycopg2\-binary**  

   This example is provided for demonstration purposes\. The boto and psycopg2\-binary libraries are included with the Apache Airflow v2\.0\.2 base install and don't need to be specified in a `requirements.txt` file\.

   ```
   boto3==1.17.54
   boto==2.49.0
   botocore==1.20.54
   psycopg2-binary==2.8.6
   ```

   If a package is specified without a version, Amazon MWAA installs the latest version of the package from [PyPi\.org](https://pypi.org)\. This version may conflict with other packages in your `requirements.txt`\.

------
#### [ Airflow v1\.10\.12 ]

1. **Test locally**\. Add additional libraries iteratively to find the right combination of packages and their versions, before creating a `requirements.txt` file\. To run the Amazon MWAA CLI utility, see the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

1. **Review the Airflow package extras**\. Review the list of packages available for Apache Airflow v1\.10\.12 at [https://raw\.githubusercontent\.com/apache/airflow/constraints\-1\.10\.12/constraints\-3\.7\.txt](https://raw.githubusercontent.com/apache/airflow/constraints-1.10.12/constraints-3.7.txt)\.

1. **Add the constraints file**\. Add the constraints file for Apache Airflow v1\.10\.12 to the top of your `requirements.txt` file\. If the constraints file determines that `xyz==1.0` package is not compatible with other packages on your environment, the `pip3 install` will fail to prevent incompatible libraries from being installed to your environment\.

   ```
   --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-1.10.12/constraints-3.7.txt"
   ```

1. **Apache Airflow v1\.10\.12 packages**\. Add the [Airflow package extras](https://airflow.apache.org/docs/apache-airflow/1.10.12/installation.html#extra-packages) and the Apache Airflow v1\.10\.12 version \(`==`\)\. This helps to prevent packages of the same name, but different version, from being installed on your environment\.

   ```
   apache-airflow[package]==1.10.12
   ```  
**Example Secure Shell \(SSH\)**  

   The following example `requirements.txt` file installs SSH for Apache Airflow v1\.10\.12\. 

   ```
   apache-airflow[ssh]==1.10.12
   ```

1. **Python libraries**\. Add the package name and the version \(`==`\) in your `requirements.txt` file\. This helps to prevent a future breaking update from [PyPi\.org](https://pypi.org) from being automatically applied\.

   ```
   library == version
   ```  
**Example Boto3**  

   The following example `requirements.txt` file installs the Boto3 library for Apache Airflow v1\.10\.12\.

   ```
   boto3 == 1.17.4
   ```

   If a package is specified without a version, Amazon MWAA installs the latest version of the package from [PyPi\.org](https://pypi.org)\. This version may conflict with other packages in your `requirements.txt`\.

------

## Uploading `requirements.txt` to Amazon S3<a name="configuring-dag-dependencies-upload"></a>

You can use the Amazon S3 console or the AWS Command Line Interface \(AWS CLI\) to upload a `requirements.txt` file to your Amazon S3 bucket\.

### Using the AWS CLI<a name="configuring-dag-dependencies-upload-cli"></a>

The AWS Command Line Interface \(AWS CLI\) is an open source tool that enables you to interact with AWS services using commands in your command\-line shell\. To complete the steps on this page, you need the following:
+ [AWS CLI – Install version 2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
+ [AWS CLI – Quick configuration with `aws configure`](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)

**To upload using the AWS CLI**

1. Use the following command to list all of your Amazon S3 buckets\.

   ```
   aws s3 ls
   ```

1. Use the following command to list the files and folders in the Amazon S3 bucket for your environment\.

   ```
   aws s3 ls s3://YOUR_S3_BUCKET_NAME
   ```

1. The following command uploads a `requirements.txt` file to an Amazon S3 bucket\.

   ```
   aws s3 cp requirements.txt s3://YOUR_S3_BUCKET_NAME/requirements.txt
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

## Installing Python dependencies on your environment<a name="configuring-dag-dependencies-installing"></a>

This section describes how to install the dependencies you uploaded to your Amazon S3 bucket by specifying the path to the requirements\.txt file, and specifying the version of the requirements\.txt file each time it's updated\.

### Specifying the path to `requirements.txt` on the Amazon MWAA console \(the first time\)<a name="configuring-dag-dependencies-first"></a>

If this is the first time you're creating and uploading a `requirements.txt` to your Amazon S3 bucket, you also need to specify the path to the file on the Amazon MWAA console\. You only need to complete this step once\.

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose **Edit**\.

1. On the **DAG code in Amazon S3** pane, choose **Browse S3** next to the **Requirements file \- optional** field\.

1. Select the `requirements.txt` file on your Amazon S3 bucket\.

1. Choose **Choose**\.

1. Choose **Next**, **Update environment**\.

You can begin using the new packages immediately after your environment finishes updating\.

### Specifying the `requirements.txt` version on the Amazon MWAA console<a name="working-dags-dependencies-mwaaconsole-version"></a>

You need to specify the version of your `requirements.txt` file on the Amazon MWAA console each time you upload a new version of your `requirements.txt` in your Amazon S3 bucket\. 

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose **Edit**\.

1. On the **DAG code in Amazon S3** pane, choose a `requirements.txt` version in the dropdown list\.

1. Choose **Next**, **Update environment**\.

You can begin using the new packages immediately after your environment finishes updating\.

## Viewing logs for your `requirements.txt`<a name="working-dags-dependencies-logs"></a>

You can view Apache Airflow logs for the *Scheduler* scheduling your workflows and parsing your `dags` folder\. The following steps describe how to open the log group for the *Scheduler* on the Amazon MWAA console, and view Apache Airflow logs on the CloudWatch Logs console\.

**To view logs for a `requirements.txt`**

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose the **Airflow scheduler log group** on the **Monitoring** pane\.

1. Choose the `requirements_install_ip` log in **Log streams**\.

1. You should see the list of packages that were installed on the environment at `/usr/local/airflow/requirements/requirements.txt`\. For example:

   ```
   Collecting appdirs==1.4.4 (from -r /usr/local/airflow/requirements/requirements.txt (line 1))
   Downloading https://files.pythonhosted.org/packages/3b/00/2344469e2084fb28kjdsfiuyweb47389789vxbmnbjhsdgf5463acd6cf5e3db69324/appdirs-1.4.4-py2.py3-none-any.whl  
   Collecting astroid==2.4.2 (from -r /usr/local/airflow/requirements/requirements.txt (line 2))
   ```

1. Review the list of packages and whether any of these encountered an error during installation\. If something went wrong, you may see an error similar to the following:

   ```
   2021-03-05T14:34:42.731-07:00
   No matching distribution found for LibraryName==1.0.0 (from -r /usr/local/airflow/requirements/requirements.txt (line 4))
   No matching distribution found for LibraryName==1.0.0 (from -r /usr/local/airflow/requirements/requirements.txt (line 4))
   ```

## What's next?<a name="working-dags-dependencies-next-up"></a>
+ Test your DAGs, custom plugins, and Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.