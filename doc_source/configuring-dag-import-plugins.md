# Installing custom plugins<a name="configuring-dag-import-plugins"></a>

Amazon Managed Workflows for Apache Airflow \(MWAA\) supports Apache Airflow's built\-in plugin manager, allowing you to use custom Apache Airflow operators, hooks, sensors, or interfaces\. This page describes the steps to install Apache Airflow [custom plugins](https://airflow.incubator.apache.org/plugins.html) on your Amazon MWAA environment using a `plugins.zip` file\.

**Topics**
+ [Prerequisites](#configuring-dag-plugins-prereqs)
+ [How it works](#configuring-dag-plugins-how)
+ [Creating a plugins\.zip file](#configuring-dag-plugins-zip)
+ [Uploading `plugins.zip` to Amazon S3](#configuring-dag-plugins-upload)
+ [Specifying the path to `plugins.zip` on the Amazon MWAA console \(the first time\)](#configuring-dag-plugins-mwaa-first)
+ [Specifying the `plugins.zip` version on the Amazon MWAA console](#configuring-dag-plugins-s3-mwaaconsole)
+ [Viewing changes on your Apache Airflow UI](#configuring-dag-plugins-s3-mwaaconsole-view)

## Prerequisites<a name="configuring-dag-plugins-prereqs"></a>

**To use the steps on this page, you'll need:**

1. The required AWS resources configured for your environment as defined in [Get started with Amazon Managed Workflows for Apache Airflow \(MWAA\)](get-started.md)\.

1. An execution role with a permissions policy that grants Amazon MWAA access to the AWS resources used by your environment as defined in [Amazon MWAA Execution role](mwaa-create-role.md)\.

1. An AWS account with access in AWS Identity and Access Management \(IAM\) to the Amazon S3 console, or the AWS Command Line Interface \(AWS CLI\) as defined in [Accessing an Amazon MWAA environment](access-policies.md)\.

## How it works<a name="configuring-dag-plugins-how"></a>

To run custom plugins on your environment, you must do three things:

1. Create a `plugins.zip` file locally\.

1. Upload the local `plugins.zip` file to your Amazon S3 storage bucket\.

1. Specify the version of this file in the **Plugins file** field on the Amazon MWAA console\.

**Note**  
If this is the first time you're uploading a `plugins.zip` to your Amazon S3 bucket, you'll also need to specify the path to the file on the Amazon MWAA console\. You only need to complete this step once\.

Apache Airflow's built\-in plugin manager can integrate external features to its core by simply dropping files in an `$AIRFLOW_HOME/plugins` folder\. It allows you to use custom Apache Airflow operators, hooks, sensors, or interfaces\.

Amazon MWAA automatically detects and syncs changes from your Amazon S3 bucket to Apache Airflow every 30 seconds\. To run an Apache Airflow platform that uses custom plugins on an Amazon MWAA environment, you need to create and copy a `plugins.zip` file to your Amazon S3 storage bucket\. For example, the contents of the `plugins.zip` file in your storage bucket may look like this:

**Example plugins\.zip**  

```
__init__.py
my_airflow_plugin.py
hooks/
    |-- __init__.py
    |-- my_airflow_hook.py
operators/
    |-- __init__.py
    |-- my_airflow_operator.py
sensors/
    |-- __init__.py
    |-- my_airflow_sensor.py
```

This assumes that your DAG definition in `dag_def.py` of your [DAG folder](https://docs.aws.amazon.com/mwaa/latest/userguide/configuring-dag-folder.html#configuring-dag-folder-how) uses an import statement for each file\. For example:

**Example dag\_def\.py**  

```
from airflow.operators.my_airflow_operator import MyOperator
from airflow.sensors.my_airflow_sensor import MySensor
from airflow.hooks.my_airflow_hook import MyHook
...
```

It also assumes that `my_airflow_plugin.py` uses the `AirflowPlugin` class:

**Example my\_airflow\_plugin\.py**  

```
from airflow.plugins_manager import AirflowPlugin
from hooks.my_airflow_hook import *
from operators.my_airflow_operator import *
from utils.my_utils import *
                    
class PluginName(AirflowPlugin):
                    
    name = 'my_airflow_plugin'
                    
    hooks = [MyHook]
    operators = [MyOperator]
    sensors = [MySensor]
    ...
```

To refer to a given plugin from your DAG, import into your Python file\. For example, the Python code for these files may look like this:

**Example my\_airflow\_hook\.py**  

```
from hooks.my_airflow_hook import BaseHook

    class MyHook(BaseHook):
        ...
```

**Example my\_airflow\_sensor\.py**  

```
from sensors.my_airflow_sensor import BaseSensorOperator
from airflow.utils.decorators import apply_defaults
            
    class MySensor(BaseSensorOperator):
        ...
```

**Example my\_airflow\_operator\.py**  

```
from airflow.operators.bash_operator import BaseOperator
from airflow.utils.decorators import apply_defaults
from hooks.my_airflow_hook import MyHook
        
    class MyOperator(BaseOperator):
        ...
```

## Creating a plugins\.zip file<a name="configuring-dag-plugins-zip"></a>

You can use a built\-in ZIP archive utility, or any other ZIP utility \(such as [7zip](https://www.7-zip.org/download.html)\) to create a \.zip file\.

**Note**  
The built\-in zip utility for Windows OS may add subfolders when you create a \.zip file\. We recommend verifying the contents of the plugins\.zip file before uploading to your Amazon S3 bucket to ensure no additional directories were added\.

1. Change directories to your `$AIRFLOW_HOME/plugins` folder\.

   ```
   myproject$ cd $AIRFLOW_HOME/plugins
   ```

1. Run the following command to ensure that the contents have executable permissions \(macOS and Linux only\)\.

   ```
   $AIRFLOW_HOME/plugins$ chmod -R 755 .
   ```

1. Zip the contents within your `$AIRFLOW_HOME/plugins` folder\.

   ```
   $AIRFLOW_HOME/plugins$ zip -r plugins.zip .
   ```

1. The directory structure of your `plugins.zip` may look like this:

   ```
   __init__.py
   my_airflow_plugin.py
   hooks/
       |-- __init__.py
       |-- my_airflow_hook.py
   operators/
       |-- __init__.py
       |-- my_airflow_operator.py
   sensors/
       |-- __init__.py
       |-- my_airflow_sensor.py
   ```

## Uploading `plugins.zip` to Amazon S3<a name="configuring-dag-plugins-upload"></a>

You can use the Amazon S3 console or the AWS Command Line Interface \(AWS CLI\) to upload a `plugins.zip` file to your Amazon S3 bucket\.

### Using the AWS CLI<a name="configuring-dag-plugins-upload-cli"></a>

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

1. The following command uploads a `plugins.zip` file to an Amazon S3 bucket\.

   ```
   aws s3 cp plugins.zip s3://your-s3-bucket-any-name/plugins.zip
   ```

### Using the Amazon S3 console<a name="configuring-dag-plugins-upload-console"></a>

The Amazon S3 console is a web\-based user interface that allows you to create and manage the resources in your Amazon S3 bucket\.

**To upload using the Amazon S3 console**

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Select the **S3 bucket** link in the **DAG code in S3** pane to open your storage bucket on the Amazon S3 console\.

1. Choose **Upload**\.

1. Choose **Add file**\.

1. Select the local copy of your `plugins.zip`, choose **Upload**\.

## Specifying the path to `plugins.zip` on the Amazon MWAA console \(the first time\)<a name="configuring-dag-plugins-mwaa-first"></a>

If this is the first time you're uploading a `plugins.zip` to your Amazon S3 bucket, you'll also need to specify the path to the file on the Amazon MWAA console\. You only need to complete this step once\.

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose **Edit**\.

1. On the **DAG code in Amazon S3** pane, choose **Browse S3** next to the **Plugins file \- optional** field\.

1. Select the `plugins.zip` file on your Amazon S3 bucket\.

1. Choose **Choose**\.

1. Choose **Next**, **Update environment**\.

You can begin using the new plugins immediately after your environment finishes updating\.

## Specifying the `plugins.zip` version on the Amazon MWAA console<a name="configuring-dag-plugins-s3-mwaaconsole"></a>

You need to specify the version of your `plugins.zip` file on the Amazon MWAA console each time you upload a new version of your `plugins.zip` in your Amazon S3 bucket\. 

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose **Edit**\.

1. On the **DAG code in Amazon S3** pane, choose a `plugins.zip` version in the dropdown list\.

1. Choose **Next**, **Update environment**\.

You can begin using the new plugins immediately after your environment finishes updating\.

## Viewing changes on your Apache Airflow UI<a name="configuring-dag-plugins-s3-mwaaconsole-view"></a>

**To access your Apache Airflow UI**

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose **Open Airflow UI** to view your Apache Airflow UI\.

**Note**  
You may need to ask your account administrator to add `AmazonMWAAWebServerAccess` permissions for your account to view your Apache Airflow UI\. For more information, see [Managing access](https://docs.aws.amazon.com/mwaa/latest/userguide/manage-access.html)\.