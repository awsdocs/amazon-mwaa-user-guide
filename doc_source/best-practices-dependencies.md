# Managing Python dependencies in requirements\.txt<a name="best-practices-dependencies"></a>

This page describes the best practices we recommend to install and manage Python dependencies in a `requirements.txt` file for an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment\.

**Contents**
+ [Testing DAGs using the Amazon MWAA CLI utility](#best-practices-dependencies-cli-utility)
+ [Installing Python dependencies using PyPi\.org Requirements File Format](#best-practices-dependencies-different-ways)
  + [Option one: Python dependencies from the Python Package Index](#best-practices-dependencies-pip-extras)
  + [Option two: Python wheels \(\.whl\)](#best-practices-dependencies-python-wheels)
    + [Using the `plugins.zip` file on an Amazon S3 bucket](#best-practices-dependencies-python-wheels-s3)
    + [Using a WHL file hosted on a URL](#best-practices-dependencies-python-wheels-url)
    + [Creating a WHL files from a DAG](#best-practices-dependencies-python-wheels-dag)
  + [Option three: Python dependencies hosted on a private PyPi/PEP\-503 Compliant Repo](#best-practices-dependencies-custom-auth-url)
+ [Enabling logs on the Amazon MWAA console](#best-practices-dependencies-troubleshooting-enable)
+ [Viewing logs on the CloudWatch Logs console](#best-practices-dependencies-troubleshooting-view)
+ [Viewing errors in the Apache Airflow UI](#best-practices-dependencies-troubleshooting-aa)
  + [Logging into Apache Airflow](#airflow-access-and-login)
+ [Example `requirements.txt` scenarios](#best-practices-dependencies-ex-mix-match)

## Testing DAGs using the Amazon MWAA CLI utility<a name="best-practices-dependencies-cli-utility"></a>
+ The command line interface \(CLI\) utility replicates an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment locally\.
+ The CLI builds a Docker container image locally that’s similar to an Amazon MWAA production image\. This allows you to run a local Apache Airflow environment to develop and test DAGs, custom plugins, and dependencies before deploying to Amazon MWAA\.
+ To run the CLI, see the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

## Installing Python dependencies using PyPi\.org Requirements File Format<a name="best-practices-dependencies-different-ways"></a>

The following section describes the different ways to install Python dependencies according to the PyPi\.org [Requirements File Format](https://pip.pypa.io/en/stable/reference/pip_install/#requirements-file-format)\.

### Option one: Python dependencies from the Python Package Index<a name="best-practices-dependencies-pip-extras"></a>

The following section describes how to specify Python dependencies from the [Python Package Index](https://pypi.org/) in a `requirements.txt` file\.

------
#### [ Apache Airflow v2 ]

1. **Test locally**\. Add additional libraries iteratively to find the right combination of packages and their versions, before creating a `requirements.txt` file\. To run the Amazon MWAA CLI utility, see the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

1. **Review the Apache Airflow package extras**\. For example, the [core extras](http://airflow.apache.org/docs/apache-airflow/2.2.2/extra-packages-ref.html#core-airflow-extras), [provider extras](http://airflow.apache.org/docs/apache-airflow/2.2.2/extra-packages-ref.html#providers-extras), [locally installed software extras](http://airflow.apache.org/docs/apache-airflow/2.2.2/extra-packages-ref.html#locally-installed-software-extras), [external service extras](http://airflow.apache.org/docs/apache-airflow/2.2.2/extra-packages-ref.html#external-services-extras), ["other" extras](http://airflow.apache.org/docs/apache-airflow/2.2.2/extra-packages-ref.html#other-extras), [bundle extras](http://airflow.apache.org/docs/apache-airflow/2.2.2/extra-packages-ref.html#bundle-extras), [doc extras](http://airflow.apache.org/docs/apache-airflow/2.2.2/extra-packages-ref.html#doc-extras), and [software extras](http://airflow.apache.org/docs/apache-airflow/2.2.2/extra-packages-ref.html#apache-software-extras) have changed\. To view a list of the packages installed for Apache Airflow v2 on Amazon MWAA, see [Amazon MWAA local runner `requirements.txt`](https://github.com/aws/aws-mwaa-local-runner/blob/main/docker/config/requirements.txt) on the GitHub website\.

1. **Add the constraints file**\. Add the constraints file for your Apache Airflow v2 environment to the top of your `requirements.txt` file\. If the constraints file determines that `xyz==1.0` package is not compatible with other packages on your environment, the `pip3 install` will fail to prevent incompatible libraries from being installed to your environment\. In the following example, replace `{Airflow-version}` with your environment's version number\.

   ```
   --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-{Airflow-version}/constraints-3.7.txt"
   ```

1. **Apache Airflow packages**\. Add the [Apache Airflow v2 package extras](http://airflow.apache.org/docs/apache-airflow/2.2.2/extra-packages-ref.html) and the version \(`==`\)\. This helps to prevent packages of the same name, but different version, from being installed on your environment\.

   ```
   apache-airflow[package-extra]==2.0.2
   ```

1. **Python libraries**\. Add the package name and the version \(`==`\) in your `requirements.txt` file\. This helps to prevent a future breaking update from [PyPi\.org](https://pypi.org) from being automatically applied\.

   ```
   library == version
   ```  
**Example Boto3 and psycopg2\-binary**  

   This example is provided for demonstration purposes\. The boto and psycopg2\-binary libraries are included with the Apache Airflow v2 base install and don't need to be specified in a `requirements.txt` file\.

   ```
   boto3==1.17.54
   boto==2.49.0
   botocore==1.20.54
   psycopg2-binary==2.8.6
   ```

   If a package is specified without a version, Amazon MWAA installs the latest version of the package from [PyPi\.org](https://pypi.org)\. This version may conflict with other packages in your `requirements.txt`\.

------
#### [ Apache Airflow v1 ]

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

### Option two: Python wheels \(\.whl\)<a name="best-practices-dependencies-python-wheels"></a>

 A Python wheel is a package format designed to ship libraries with compiled artifacts\. There are several benefits to wheel packages as a method to install dependencies in Amazon MWAA: 
+  **Faster installation** – the WHL files are copied to the container as a single ZIP, and then installed locally, without having to download each one\. 
+  **Fewer conflicts** – You can determine version compatibility for your packages in advance\. As a result, there is no need for `pip` to recursively work out compatible versions\. 
+  **More resilience** – With externally hosted libraries, downstream requirements can change, resulting in version incompatibility between containers on a Amazon MWAA environment\. By not depending on an external source for dependencies, every container on has have the same libraries regardless of when the each container is instantiated\. 

 We recommend the following methods to install Python dependencies from a Python wheel archive \(`.whl`\) in your `requirements.txt`\. 

**Topics**
+ [Using the `plugins.zip` file on an Amazon S3 bucket](#best-practices-dependencies-python-wheels-s3)
+ [Using a WHL file hosted on a URL](#best-practices-dependencies-python-wheels-url)
+ [Creating a WHL files from a DAG](#best-practices-dependencies-python-wheels-dag)

#### Using the `plugins.zip` file on an Amazon S3 bucket<a name="best-practices-dependencies-python-wheels-s3"></a>

The Apache Airflow scheduler, workers, and web server \(for Apache Airflow v2\.2\.2 and later\) look for custom plugins during startup on the AWS\-managed Fargate container for your environment at `/usr/local/airflow/plugins/*`\. This process begins prior to Amazon MWAA's `pip3 install -r requirements.txt` for Python dependencies and Apache Airflow service startup\. A `plugins.zip` file be used for any files that you don't want continuously changed during environment execution, or that you may not want to grant access to users that write DAGs\. For example, Python library wheel files, certificate PEM files, and configuration YAML files\.

The following section describes how to install a wheel that's in the `plugins.zip` file on your Amazon S3 bucket\. 

1.  **Download the necessary WHL files** You can use [https://pip.pypa.io/en/stable/cli/pip_download/](https://pip.pypa.io/en/stable/cli/pip_download/) with your existing `requirements.txt` on the Amazon MWAA [local\-runner](https://github.com/aws/aws-mwaa-local-runner) or another [Amazon Linux 2](http://aws.amazon.com/https://aws.amazon.com/amazon-linux-2) container to resolve and download the necessary Python wheel files\. 

   ```
   $ pip3 download -r "$AIRFLOW_HOME/dags/requirements.txt" -d "$AIRFLOW_HOME/plugins"
   $ cd "$AIRFLOW_HOME/plugins"
   $ zip "$AIRFLOW_HOME/plugins.zip" *
   ```

1. **Specify the path in your `requirements.txt`**\. Specify the plugins directory at the top of your requirements\.txt using [https://pip.pypa.io/en/stable/cli/pip_install/#install-find-links](https://pip.pypa.io/en/stable/cli/pip_install/#install-find-links) and instruct `pip` not to install from other sources using [https://pip.pypa.io/en/stable/cli/pip_install/#install-no-index](https://pip.pypa.io/en/stable/cli/pip_install/#install-no-index), as shown in the following 

   ```
   --find-links /usr/local/airflow/plugins
   --no-index
   ```  
**Example wheel in requirements\.txt**  

   The following example assumes you've uploaded the wheel in a `plugins.zip` file at the root of your Amazon S3 bucket\. For example:

   ```
   --find-links /usr/local/airflow/plugins
   --no-index
   
   numpy
   ```

   Amazon MWAA fetches the `numpy-1.20.1-cp37-cp37m-manylinux1_x86_64.whl` wheel from the `plugins` folder and installs it on your environment\.

#### Using a WHL file hosted on a URL<a name="best-practices-dependencies-python-wheels-url"></a>

 The following section describes how to install a wheel that's hosted on a URL\. The URL must either be publicly accessible, or accessible from within the custom Amazon VPC you specified for your Amazon MWAA environment\. 
+ **Provide a URL**\. Provide the URL to a wheel in your `requirements.txt`\.  
**Example wheel archive on a public URL**  

  The following example downloads a wheel from a public site\.

  ```
  --find-links https://files.pythonhosted.org/packages/
  --no-index
  ```

  Amazon MWAA fetches the wheel from the URL you specified and installs them on your environment\.
**Note**  
 URLs are not accessible from private web servers installing requirements in Amazon MWAA v2\.2\.2 and later\. 

#### Creating a WHL files from a DAG<a name="best-practices-dependencies-python-wheels-dag"></a>

If you have a private web server using Apache Airflow v2\.2\.2 or later and you're unable to install requirements because your environment does not have access to external repositories, you can use the following DAG to take your existing MAmazon MWAA requirements and package them on Amazon S3: 

```
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.utils.dates import days_ago

S3_BUCKET = 'my-s3-bucket'
S3_KEY = 'backup/plugins_whl.zip' 

with DAG(dag_id="create_whl_file", schedule_interval=None, catchup=False, start_date=days_ago(1)) as dag:
    cli_command = BashOperator(
        task_id="bash_command",
        bash_command=f"mkdir /tmp/whls;pip3 download -r /usr/local/airflow/requirements/requirements.txt -d /tmp/whls;zip -j /tmp/plugins.zip /tmp/whls/*;aws s3 cp /tmp/plugins.zip s3://{S3_BUCKET}/{S3_KEY}"
    )
```

 After running the DAG, use this new file as your Amazon MWAA `plugins.zip`, optionally, packaged with other plugins\. Then, update your `requirements.txt` preceeded by `--find-links /usr/local/airflow/plugins` and `--no-index` without adding `--constraint`\. 

 This method allows you to use the same libraries offline\. 

### Option three: Python dependencies hosted on a private PyPi/PEP\-503 Compliant Repo<a name="best-practices-dependencies-custom-auth-url"></a>

The following section describes how to install an Apache Airflow extra that's hosted on a private URL with authentication\.

1. Add your username and password as [Apache Airflow configuration options](configuring-env-variables.md)\. For example:
   + `foo.user` : `YOUR_USER_NAME`
   + `foo.pass` : `YOUR_PASSWORD`

1. Create your `requirements.txt` file\. Substitute the placeholders in the following example with your private URL, and the username and password you've added as [Apache Airflow configuration options](configuring-env-variables.md)\. For example:

   ```
   --index-url https://${AIRFLOW__FOO__USER}:${AIRFLOW__FOO__PASS}@my.privatepypi.com
   ```

1. Add any additional libraries to your `requirements.txt` file\. For example:

   ```
   --index-url https://${AIRFLOW__FOO__USER}:${AIRFLOW__FOO__PASS}@my.privatepypi.com
   my-private-package==1.2.3
   ```

## Enabling logs on the Amazon MWAA console<a name="best-practices-dependencies-troubleshooting-enable"></a>

The [execution role](mwaa-create-role.md) for your Amazon MWAA environment needs permission to send logs to CloudWatch Logs\. To update the permissions of an execution role, see [Amazon MWAA execution role](mwaa-create-role.md)\.

You can enable Apache Airflow logs at the `INFO`, `WARNING`, `ERROR`, or `CRITICAL` level\. When you choose a log level, Amazon MWAA sends logs for that level and all higher levels of severity\. For example, if you enable logs at the `INFO` level, Amazon MWAA sends `INFO` logs and `WARNING`, `ERROR`, and `CRITICAL` log levels to CloudWatch Logs\. We recommend enabling Apache Airflow logs at the `INFO` level for the *Scheduler* to view logs received for the `requirements.txt`\. 

![\[This image shows how to enable logs at the INFO level.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-logs-info.png)

## Viewing logs on the CloudWatch Logs console<a name="best-practices-dependencies-troubleshooting-view"></a>

You can view Apache Airflow logs for the *Scheduler* scheduling your workflows and parsing your `dags` folder\. The following steps describe how to open the log group for the *Scheduler* on the Amazon MWAA console, and view Apache Airflow logs on the CloudWatch Logs console\.

**To view logs for a `requirements.txt`**

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose the **Airflow scheduler log group** on the **Monitoring** pane\.

1. Choose the `requirements_install_ip` log in **Log streams**\.

1. You should see the list of packages that were installed on the environment at `/usr/local/airflow/.local/bin`\. For example:

   ```
   Collecting appdirs==1.4.4 (from -r /usr/local/airflow/.local/bin (line 1))
   Downloading https://files.pythonhosted.org/packages/3b/00/2344469e2084fb28kjdsfiuyweb47389789vxbmnbjhsdgf5463acd6cf5e3db69324/appdirs-1.4.4-py2.py3-none-any.whl  
   Collecting astroid==2.4.2 (from -r /usr/local/airflow/.local/bin (line 2))
   ```

1. Review the list of packages and whether any of these encountered an error during installation\. If something went wrong, you may see an error similar to the following:

   ```
   2021-03-05T14:34:42.731-07:00
   No matching distribution found for LibraryName==1.0.0 (from -r /usr/local/airflow/.local/bin (line 4))
   No matching distribution found for LibraryName==1.0.0 (from -r /usr/local/airflow/.local/bin (line 4))
   ```

## Viewing errors in the Apache Airflow UI<a name="best-practices-dependencies-troubleshooting-aa"></a>

You may also want to check your Apache Airflow UI to identify whether an error may be related to another issue\. The most common error you may encounter with Apache Airflow on Amazon MWAA is:

```
Broken DAG: No module named x
```

If you see this error in your Apache Airflow UI, you're likely missing a required dependency in your `requirements.txt` file\.

### Logging into Apache Airflow<a name="airflow-access-and-login"></a>

You need [Apache Airflow UI access policy: AmazonMWAAWebServerAccess](access-policies.md#web-ui-access) permissions for your AWS account in AWS Identity and Access Management \(IAM\) to view your Apache Airflow UI\. 

**To access your Apache Airflow UI**

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose **Open Airflow UI**\.

**To log\-in to your Apache Airflow UI**
+ Enter the AWS Identity and Access Management \(IAM\) user name and password for your account\.

![\[This image shows how to log-in to your Apache Airflow UI.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-aa-ui-login.png)

## Example `requirements.txt` scenarios<a name="best-practices-dependencies-ex-mix-match"></a>

You can mix and match different formats in your `requirements.txt`\. The following example uses a combination of the different ways to install extras\.

**Example Extras on PyPi\.org and a public URL**  
You need to use the `--index-url` option when specifying packages from PyPi\.org, in addition to packages on a public URL, such as custom PEP 503 compliant repo URLs\.  

```
aws-batch == 0.6
phoenix-letter >= 0.3
    
--index-url http://dist.repoze.org/zope2/2.10/simple
    zopelib
```