# Installing custom plugins<a name="configuring-dag-import-plugins"></a>

Amazon Managed Workflows for Apache Airflow \(MWAA\) supports Apache Airflow's built\-in plugin manager, allowing you to use custom Apache Airflow operators, hooks, sensors, or interfaces\. This page describes the steps to install [Apache Airflow custom plugins](https://airflow.incubator.apache.org/plugins.html) on your Amazon MWAA environment using a `plugins.zip` file\.

**Contents**
+ [Prerequisites](#configuring-dag-plugins-prereqs)
+ [How it works](#configuring-dag-plugins-how)
+ [What's changed in v2\.0\.2](#configuring-dag-plugins-changed)
+ [Custom plugins overview](#configuring-dag-plugins-overview)
  + [Custom plugins directory and size limits](#configuring-dag-plugins-quota)
+ [Examples of custom plugins](#configuring-dag-plugins-airflow-ex)
  + [Example using a flat directory structure in plugins\.zip](#configuring-dag-plugins-overview-simple)
  + [Example using a nested directory structure in plugins\.zip](#configuring-dag-plugins-overview-complex)
+ [Creating a plugins\.zip file](#configuring-dag-plugins-test-create)
  + [Step one: Test custom plugins using the Amazon MWAA CLI utility](#configuring-dag-plugins-cli-utility)
  + [Step two: Create the plugins\.zip file](#configuring-dag-plugins-zip)
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

1. **Access**\. Your AWS account must have been granted access by your administrator to the [AmazonMWAAFullConsoleAccess](access-policies.md#console-full-access) access control policy for your environment\.

1. **Amazon S3 configurations**\. The [Amazon S3 bucket](mwaa-s3-bucket.md) used to store your DAGs, custom plugins in `plugins.zip`, and Python dependencies in `requirements.txt` must be configured with *Public Access Blocked* and *Versioning Enabled*\.

1. **Permissions**\. Your Amazon MWAA environment must be permitted by your [execution role](mwaa-create-role.md) to access the AWS resources used by your environment\.

## How it works<a name="configuring-dag-plugins-how"></a>

To run custom plugins on your environment, you must do three things:

1. Create a `plugins.zip` file locally\.

1. Upload the local `plugins.zip` file to your Amazon S3 bucket\.

1. Specify the version of this file in the **Plugins file** field on the Amazon MWAA console\.

**Note**  
If this is the first time you're uploading a `plugins.zip` to your Amazon S3 bucket, you also need to specify the path to the file on the Amazon MWAA console\. You only need to complete this step once\.

## What's changed in v2\.0\.2<a name="configuring-dag-plugins-changed"></a>
+ **New: Operators, Hooks, and Executors**\. The import statements in your DAGs, and the custom plugins you specify in a `plugins.zip` on Amazon MWAA have changed between Apache Airflow v1\.10\.12 and Apache Airflow v2\.0\.2\. For example, `from airflow.contrib.hooks.aws_hook import AwsHook` in Apache Airflow v1\.10\.12 has changed to `from airflow.providers.amazon.aws.hooks.base_aws import AwsBaseHook` in Apache Airflow v2\.0\.2\. To learn more, see [Python API Reference](https://airflow.apache.org/docs/apache-airflow/2.0.2/python-api-ref.html) in the *Apache Airflow reference guide*\.
+ **New: Imports in plugins**\. Importing operators, sensors, hooks added in plugins using `airflow.{operators,sensors,hooks}.<plugin_name>` is no longer supported\. These extensions should be imported as regular Python modules\. In v2\.0\+, the recommended approach is to place them in the DAGs directory and create and use an *\.airflowignore* file to exclude them from being parsed as DAGs\. To learn more, see [Modules Management](https://airflow.apache.org/docs/apache-airflow/stable/modules_management.html) and [Creating a custom Operator](https://airflow.apache.org/docs/apache-airflow/stable/howto/custom-operator.html) in the *Apache Airflow reference guide*\.

## Custom plugins overview<a name="configuring-dag-plugins-overview"></a>

Apache Airflow's built\-in plugin manager can integrate external features to its core by simply dropping files in an `$AIRFLOW_HOME/plugins` folder\. It allows you to use custom Apache Airflow operators, hooks, sensors, or interfaces\. The following section provides an example of flat and nested directory structures in a local development environment and the resulting import statements, which determines the directory structure within a plugins\.zip\.

### Custom plugins directory and size limits<a name="configuring-dag-plugins-quota"></a>

The Apache Airflow *Scheduler* and the *Workers* look for custom plugins during startup on the AWS\-managed Fargate container for your environment at `/usr/local/airflow/plugins/*`\.
+ **Directory structure**\. The directory structure \(at `/*`\) is based on the contents of your `plugins.zip` file\. For example, if your `plugins.zip` contains the `operators` directory as a top\-level directory, then the directory will be extracted to `/usr/local/airflow/plugins/operators` on your environment\.
+ **Size limit**\. We recommend a `plugins.zip` file less than than 1 GB\. The larger the size of a `plugins.zip` file, the longer the startup time on an environment\. Although Amazon MWAA doesn't limit the size of a `plugins.zip` file explicitly, if dependencies can't be installed within ten minutes, the Fargate service will time\-out and attempt to rollback the environment to a stable state\. 

**Note**  
For security reasons, the Apache Airflow *Web server* on Amazon MWAA has limited network egress, and does not install plugins nor Python dependencies directly on the *Web server*\.

## Examples of custom plugins<a name="configuring-dag-plugins-airflow-ex"></a>

The following section uses sample code in the *Apache Airflow reference guide* to show how to structure your local development environment\.

### Example using a flat directory structure in plugins\.zip<a name="configuring-dag-plugins-overview-simple"></a>

------
#### [ Airflow v2\.0\.2 ]

The following example shows a `plugins.zip` file with a flat directory structure for Apache Airflow v2\.0\.2\.

**Example flat directory with PythonVirtualenvOperator plugins\.zip**  
The following example shows the top\-level tree of a plugins\.zip file for the PythonVirtualenvOperator custom plugin in [Creating a custom plugin for Apache Airflow PythonVirtualenvOperator](samples-virtualenv.md)\.   

```
├── virtual_python_plugin.py
```

**Example plugins/virtual\_python\_plugin\.py**  
The following example shows the PythonVirtualenvOperator custom plugin\.  

```
"""
Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.
 
Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
the Software, and to permit persons to whom the Software is furnished to do so.
 
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
"""
from airflow.plugins_manager import AirflowPlugin
import airflow.utils.python_virtualenv 
from typing import List

def _generate_virtualenv_cmd(tmp_dir: str, python_bin: str, system_site_packages: bool) -> List[str]:
    cmd = ['python3','/usr/local/airflow/.local/lib/python3.7/site-packages/virtualenv', tmp_dir]
    if system_site_packages:
        cmd.append('--system-site-packages')
    if python_bin is not None:
        cmd.append(f'--python={python_bin}')
    return cmd

airflow.utils.python_virtualenv._generate_virtualenv_cmd=_generate_virtualenv_cmd

class VirtualPythonPlugin(AirflowPlugin):                
    name = 'virtual_python_plugin'
```

------
#### [ Airflow v1\.10\.12 ]

The following example shows a `plugins.zip` file with a flat directory structure for Apache Airflow v1\.10\.12\.

**Example flat directory with PythonVirtualenvOperator plugins\.zip**  
The following example shows the top\-level tree of a plugins\.zip file for the PythonVirtualenvOperator custom plugin in [Creating a custom plugin for Apache Airflow PythonVirtualenvOperator](samples-virtualenv.md)\.   

```
├── virtual_python_plugin.py
```

**Example plugins/virtual\_python\_plugin\.py**  
The following example shows the PythonVirtualenvOperator custom plugin\.  

```
from airflow.plugins_manager import AirflowPlugin
from airflow.operators.python_operator import PythonVirtualenvOperator

def _generate_virtualenv_cmd(self, tmp_dir):
    cmd = ['python3','/usr/local/airflow/.local/lib/python3.7/site-packages/virtualenv', tmp_dir]
    if self.system_site_packages:
        cmd.append('--system-site-packages')
    if self.python_version is not None:
        cmd.append('--python=python{}'.format(self.python_version))
    return cmd
PythonVirtualenvOperator._generate_virtualenv_cmd=_generate_virtualenv_cmd

class EnvVarPlugin(AirflowPlugin):                
    name = 'virtual_python_plugin'
```

------

### Example using a nested directory structure in plugins\.zip<a name="configuring-dag-plugins-overview-complex"></a>

------
#### [ Airflow v2\.0\.2 ]

The following example shows a `plugins.zip` file with separate directories for `hooks`, `operators`, and a `sensors` directory for Apache Airflow v2\.0\.2\.

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
from operators.my_airflow_operator import MyOperator
from sensors.my_airflow_sensor import MySensor
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
                    
class PluginName(AirflowPlugin):
                    
    name = 'my_airflow_plugin'
                    
    hooks = [MyHook]
    operators = [MyOperator]
    sensors = [MySensor]
```

The following examples show each of the import statements needed in the custom plugin files\.

**Example hooks/my\_airflow\_hook\.py**  

```
from airflow.hooks.base import BaseHook


class MyHook(BaseHook):

    def my_method(self):
        print("Hello World")
```

**Example sensors/my\_airflow\_sensor\.py**  

```
from airflow.sensors.base import BaseSensorOperator
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
from airflow.operators.bash import BaseOperator
from airflow.utils.decorators import apply_defaults
from hooks.my_airflow_hook import MyHook


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

Follow the steps in [Testing custom plugins using the Amazon MWAA CLI utility](#configuring-dag-plugins-cli-utility), and then [Creating a plugins\.zip file](#configuring-dag-plugins-zip) to zip the contents **within** your `plugins` directory\. For example, `cd plugins`\.

------
#### [ Airflow v1\.10\.12 ]

The following example shows a `plugins.zip` file with separate directories for `hooks`, `operators`, and a `sensors` directory for Apache Airflow v1\.10\.12\.

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

Follow the steps in [Testing custom plugins using the Amazon MWAA CLI utility](#configuring-dag-plugins-cli-utility), and then [Creating a plugins\.zip file](#configuring-dag-plugins-zip) to zip the contents **within** your `plugins` directory\. For example, `cd plugins`\.

------

## Creating a plugins\.zip file<a name="configuring-dag-plugins-test-create"></a>

The following steps describe the steps we recommend to create a plugins\.zip file locally\.

### Step one: Test custom plugins using the Amazon MWAA CLI utility<a name="configuring-dag-plugins-cli-utility"></a>
+ The command line interface \(CLI\) utility replicates an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment locally\.
+ The CLI builds a Docker container image locally that’s similar to an Amazon MWAA production image\. This allows you to run a local Apache Airflow environment to develop and test DAGs, custom plugins, and dependencies before deploying to Amazon MWAA\.
+ To run the CLI, see the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.

### Step two: Create the plugins\.zip file<a name="configuring-dag-plugins-zip"></a>

You can use a built\-in ZIP archive utility, or any other ZIP utility \(such as [7zip](https://www.7-zip.org/download.html)\) to create a \.zip file\.

**Note**  
The built\-in zip utility for Windows OS may add subfolders when you create a \.zip file\. We recommend verifying the contents of the plugins\.zip file before uploading to your Amazon S3 bucket to ensure no additional directories were added\.

1. Change directories to your local Airflow plugins directory\. For example:

   ```
   myproject$ cd plugins
   ```

1. Run the following command to ensure that the contents have executable permissions \(macOS and Linux only\)\.

   ```
   plugins$ chmod -R 755 .
   ```

1. Zip the contents **within** your `plugins` folder\.

   ```
   plugins$ zip -r plugins.zip .
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

1. Choose **Next**\.

## Example use cases for plugins\.zip<a name="configuring-dag-plugins-examples"></a>
+ Learn how to create a custom plugin in [Custom plugin with Apache Hive and Hadoop](samples-hive.md)\.
+ Learn how to create a custom plugin in [Custom plugin to patch PythonVirtualenvOperator ](samples-virtualenv.md)\.
+ Learn how to create a custom plugin in [Custom plugin with Oracle](samples-oracle.md)\.

## What's next?<a name="configuring-dag-plugins-next-up"></a>
+ Test your DAGs, custom plugins, and Python dependencies locally using the [aws\-mwaa\-local\-runner](https://github.com/aws/aws-mwaa-local-runner) on GitHub\.