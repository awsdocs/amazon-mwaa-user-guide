# Installing custom plugins<a name="configuring-dag-import-plugins"></a>

Amazon Managed Workflows for Apache Airflow \(MWAA\) supports Apache Airflow's built\-in plugin manager, allowing you to use custom Apache Airflow operators, hooks, sensors, or interfaces\. This page describes the steps to install [Apache Airflow custom plugins](https://airflow.incubator.apache.org/plugins.html) on your Amazon MWAA environment using a `plugins.zip` file\.

**Contents**
+ [Prerequisites](#configuring-dag-plugins-prereqs)
+ [How it works](#configuring-dag-plugins-how)
+ [Custom plugins overview](#configuring-dag-plugins-overview)
  + [Custom plugins directory and size limits](#configuring-dag-plugins-quota)
  + [Example using a flat directory structure in plugins\.zip](#configuring-dag-plugins-overview-simple)
  + [Example using a nested directory structure in plugins\.zip](#configuring-dag-plugins-overview-complex)
  + [Testing custom plugins using the Amazon MWAA CLI utility](#configuring-dag-plugins-cli-utility)
  + [Creating a plugins\.zip file](#configuring-dag-plugins-zip)
+ [Uploading `plugins.zip` to Amazon S3](#configuring-dag-plugins-upload)
  + [Using the AWS CLI](#configuring-dag-plugins-upload-cli)
  + [Using the Amazon S3 console](#configuring-dag-plugins-upload-console)
+ [Installing custom plugins on your environment](#configuring-dag-plugins-mwaa-installing)
  + [Specifying the path to `plugins.zip` on the Amazon MWAA console \(the first time\)](#configuring-dag-plugins-mwaa-first)
  + [Specifying the `plugins.zip` version on the Amazon MWAA console](#configuring-dag-plugins-s3-mwaaconsole)
+ [Example use cases for plugins\.zip](#configuring-dag-plugins-examples)
+ [What's next?](#configuring-dag-plugins-next-up)

## Prerequisites<a name="configuring-dag-plugins-prereqs"></a>

You'll need the following before you can complete the steps on this page\.

1. An [AWS account with access](access-policies.md) to your environment\.

1. An [Amazon S3 bucket](mwaa-s3-bucket.md) with *Public Access Blocked* and *Versioning Enabled*\.

1. An [execution role](mwaa-create-role.md) that grants Amazon MWAA access to the AWS resources used by your environment\.

## How it works<a name="configuring-dag-plugins-how"></a>

To run custom plugins on your environment, you must do three things:

1. Create a `plugins.zip` file locally\.

1. Upload the local `plugins.zip` file to your Amazon S3 bucket\.

1. Specify the version of this file in the **Plugins file** field on the Amazon MWAA console\.

**Note**  
If this is the first time you're uploading a `plugins.zip` to your Amazon S3 bucket, you also need to specify the path to the file on the Amazon MWAA console\. You only need to complete this step once\.

## Custom plugins overview<a name="configuring-dag-plugins-overview"></a>

Apache Airflow's built\-in plugin manager can integrate external features to its core by simply dropping files in an `$AIRFLOW_HOME/plugins` folder\. It allows you to use custom Apache Airflow operators, hooks, sensors, or interfaces\. The following section provides an example of flat and nested directory structures in a local development environment and the resulting import statements, which determines the directory structure within a plugins\.zip\.

### Custom plugins directory and size limits<a name="configuring-dag-plugins-quota"></a>

The Apache Airflow *Scheduler* and the *Workers* look for custom plugins during startup on the AWS\-managed Fargate container for your environment at `/usr/local/airflow/plugins/*`\.
+ **Directory structure**\. The directory structure \(at `/*`\) is based on the contents of your `plugins.zip` file\. For example, if your `plugins.zip` contains the `operators` directory as a top\-level directory, then the directory will be extracted to `/usr/local/airflow/plugins/operators` on your environment\.
+ **Size limit**\. We recommend a `plugins.zip` file less than than 1 GB\. The larger the size of a `plugins.zip` file, the longer the startup time on an environment\. Although Amazon MWAA doesn't limit the size of a `plugins.zip` file explicitly, if dependencies can't be installed within ten minutes, the Fargate service will time\-out and attempt to rollback the environment to a stable state\. 

**Note**  
For security reasons, the Apache Airflow *Web server* on Amazon MWAA has limited network egress, and does not install plugins nor Python dependencies directly on the *Web server*\.

The following section uses sample code in the *Apache Airflow reference guide* to show how to structure your local development environment\.

### Example using a flat directory structure in plugins\.zip<a name="configuring-dag-plugins-overview-simple"></a>

------
#### [ Airflow v1\.10\.12 ]

The following example shows custom plugin files in a flat directory structure, and how to change the code to use a nested directory structure for Apache Airflow v1\.10\.12\.

**Example Oracle plugins\.zip**  
The following example shows the top\-level tree of a plugins\.zip file for the custom Oracle plugin in [Creating a custom plugin with Oracle](samples-oracle.md)\.   

```
├── env_var_plugin_oracle.py
└── instantclient_18_5/
```

**Example dags/oracle\.py**  
The following example shows the import statements in the DAG \([DAGs folder](https://docs.aws.amazon.com/mwaa/latest/userguide/configuring-dag-folder.html#configuring-dag-folder-how)\) that uses the custom plugin\.  

```
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from airflow.utils.dates import days_ago
import os
import cx_Oracle
...
```

**Example plugins/boto\_plugin\.py**  
If you want to use a nested directory structure, you can also create a custom plugin at the top\-level of your plugins\.zip file \(saved locally at the root in the `plugins/` directory\) that uses the custom Oracle plugin in [Creating a custom plugin with Oracle](samples-oracle.md)\.  

```
from airflow.plugins_manager import AirflowPlugin
from airflow.hooks.base_hook import BaseHook

import boto3

class BotoHook(BaseHook):
    def get_version(self):
        return boto3.__version__                 

class BotoPlugin(AirflowPlugin):                
    name = 'boto_plugin'          
    hooks = [BotoHook]
```

**Example dags/oracle\.py**  
The following example shows the import statements in the DAG \([DAGs folder](https://docs.aws.amazon.com/mwaa/latest/userguide/configuring-dag-folder.html#configuring-dag-folder-how)\) that uses the custom plugin\.  

```
from airflow import DAG
from boto_plugin import BotoHook
...
```

------

### Example using a nested directory structure in plugins\.zip<a name="configuring-dag-plugins-overview-complex"></a>

------
#### [ Airflow v1\.10\.12 ]

The following example shows custom plugin files in separate directories in the `hooks`, `operators`, and `sensors` directory for Apache Airflow v1\.10\.12\.

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
    |-- hello_operator.py
sensors/
    |-- __init__.py
    |-- my_airflow_sensor.py
```

The following example shows the import statements in the DAG \([DAGs folder](https://docs.aws.amazon.com/mwaa/latest/userguide/configuring-dag-folder.html#configuring-dag-folder-how)\) that uses the custom plugins\.

**Example dags/your\_dag\.py**  

```
from airflow import DAG
from datetime import datetime, timedelta
from operators.my_operator import MyOperator
from sensors.my_sensor import MySensor
from operators.hello_operator import HelloOperator

default_args = {
	'owner': 'airflow',
	'depends_on_past': False,
	'start_date': datetime(2018, 1, 1),
	'email_on_failure': False,
	'email_on_retry': False,
	'retries': 1,
	'retry_delay': timedelta(minutes=5),
}


with DAG('customdag',
		 max_active_runs=3,
		 schedule_interval='@once',
		 default_args=default_args) as dag:

	sens = MySensor(
		task_id='taskA'
	)

	op = MyOperator(
		task_id='taskB',
		my_field='some text'
	)

	hello_task = HelloOperator(task_id='sample-task', name='foo_bar')



	sens >> op >> hello_task
```

**Example plugins/my\_airflow\_plugin\.py**  

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
```

The following examples show each of the import statements needed in the custom plugin files\.

**Example hooks/my\_airflow\_hook\.py**  

```
from airflow.hooks.base_hook import BaseHook


class MyHook(BaseHook):

    def my_method(self):
        print("Hello World")
```

**Example sensors/my\_airflow\_sensor\.py**  

```
from airflow.sensors.base_sensor_operator import BaseSensorOperator
from airflow.utils.decorators import apply_defaults


class MySensor(BaseSensorOperator):

    @apply_defaults
    def __init__(self,
                 *args,
                 **kwargs):
        super(MySensor, self).__init__(*args, **kwargs)

    def poke(self, context):
        return True
```

**Example operators/my\_airflow\_operator\.py**  

```
from airflow.operators.bash_operator import BaseOperator
from airflow.utils.decorators import apply_defaults
from hooks.my_hook import MyHook


class MyOperator(BaseOperator):

    @apply_defaults
    def __init__(self,
                 my_field,
                 *args,
                 **kwargs):
        super(MyOperator, self).__init__(*args, **kwargs)
        self.my_field = my_field

    def execute(self, context):
        hook = MyHook('my_conn')
        hook.my_method()
```

**Example operators/hello\_operator\.py**  

```
from airflow.models.baseoperator import BaseOperator
from airflow.utils.decorators import apply_defaults

class HelloOperator(BaseOperator):

    @apply_defaults
    def __init__(
            self,
            name: str,
            **kwargs) -> None:
        super().__init__(**kwargs)
        self.name = name

    def execute(self, context):
        message = "Hello {}".format(self.name)
        print(message)
        return message
```

------

### Testing custom plugins using the Amazon MWAA CLI utility<a name="configuring-dag-plugins-cli-utility"></a>
+ The command line interface \(CLI\) utility replicates an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment locally\.
+ The CLI builds a Docker container image locally that’s similar to an Amazon MWAA production image\. This allows you to run a local Apache Airflow environment to develop and test DAGs, custom plugins, and dependencies before deploying to Amazon MWAA\.
+ To run the CLI, see the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

### Creating a plugins\.zip file<a name="configuring-dag-plugins-zip"></a>

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

## Uploading `plugins.zip` to Amazon S3<a name="configuring-dag-plugins-upload"></a>

You can use the Amazon S3 console or the AWS Command Line Interface \(AWS CLI\) to upload a `plugins.zip` file to your Amazon S3 bucket\.

### Using the AWS CLI<a name="configuring-dag-plugins-upload-cli"></a>

The AWS Command Line Interface \(AWS CLI\) is an open source tool that enables you to interact with AWS services using commands in your command\-line shell\. To complete the steps on this page, you need the following:
+ [AWS CLI – Install version 2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
+ [AWS CLI – Quick configuration with `aws configure`](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)

**To upload using the AWS CLI**

1. In your command prompt, navigate to the directory where your `plugins.zip` file is stored\. For example:

   ```
   cd plugins
   ```

1. Use the following command to list all of your Amazon S3 buckets\.

   ```
   aws s3 ls
   ```

1. Use the following command to list the files and folders in the Amazon S3 bucket for your environment\.

   ```
   aws s3 ls s3://YOUR_S3_BUCKET_NAME
   ```

1. Use the following command to upload the `plugins.zip` file to the Amazon S3 bucket for your environment\.

   ```
   aws s3 cp plugins.zip s3://YOUR_S3_BUCKET_NAME/plugins.zip
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

## Installing custom plugins on your environment<a name="configuring-dag-plugins-mwaa-installing"></a>

This section describes how to install the custom plugins you uploaded to your Amazon S3 bucket by specifying the path to the plugins\.zip file, and specifying the version of the plugins\.zip file each time the zip file is updated\.

### Specifying the path to `plugins.zip` on the Amazon MWAA console \(the first time\)<a name="configuring-dag-plugins-mwaa-first"></a>

If this is the first time you're uploading a `plugins.zip` to your Amazon S3 bucket, you also need to specify the path to the file on the Amazon MWAA console\. You only need to complete this step once\.

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose **Edit**\.

1. On the **DAG code in Amazon S3** pane, choose **Browse S3** next to the **Plugins file \- optional** field\.

1. Select the `plugins.zip` file on your Amazon S3 bucket\.

1. Choose **Choose**\.

1. Choose **Next**, **Update environment**\.

### Specifying the `plugins.zip` version on the Amazon MWAA console<a name="configuring-dag-plugins-s3-mwaaconsole"></a>

You need to specify the version of your `plugins.zip` file on the Amazon MWAA console each time you upload a new version of your `plugins.zip` in your Amazon S3 bucket\. 

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose **Edit**\.

1. On the **DAG code in Amazon S3** pane, choose a `plugins.zip` version in the dropdown list\.

1. Choose **Next**, **Update environment**\.

## Example use cases for plugins\.zip<a name="configuring-dag-plugins-examples"></a>
+ Learn how to create a custom plugin in [Custom plugin with Apache Hive and Hadoop](samples-hive.md)\.
+ Learn how to create a custom plugin in [Custom plugin to patch PythonVirtualenvOperator ](samples-virtualenv.md)\.
+ Learn how to create a custom plugin in [Custom plugin with Oracle](samples-oracle.md)\.

## What's next?<a name="configuring-dag-plugins-next-up"></a>
+ Test your DAGs, custom plugins, and Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.