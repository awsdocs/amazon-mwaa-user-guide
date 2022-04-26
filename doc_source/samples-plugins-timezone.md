# Changing a DAG's timezone on Amazon MWAA<a name="samples-plugins-timezone"></a>

 Apache Airflow schedules your DAGs in UTC\+0 by default\. The following steps show how you can change the timezone in which Amazon MWAA runs your DAGs with [Pendulum](https://pypi.org/project/pendulum/), and optionally create a custom plugin to change the timezone for your environment' Apache Airflow logs\. 

**Topics**
+ [Version](#samples-plugins-timezone-version)
+ [Prerequisites](#samples-plugins-timezone-prerequisites)
+ [Permissions](#samples-plugins-timezone-permissions)
+ [Using `pendulum` in a DAG](#samples-plugins-timezone-dag)
+ [Create a plugin to change the timezone in Airflow logs](#samples-plugins-timezone-custom-plugin)
+ [Create a `plugins.zip`](#samples-plugins-timezone-plugins-zip)
+ [What's next?](#samples-plugins-timezone-plugins-next-up)

## Version<a name="samples-plugins-timezone-version"></a>
+ The sample code on this page can be used with **Apache Airflow v1** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.
+ The sample code on this page can be used with **Apache Airflow v2 and above** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="samples-plugins-timezone-prerequisites"></a>

To use the sample code on this page, you'll need the following:
+ An [Amazon MWAA environment](get-started.md)\.

## Permissions<a name="samples-plugins-timezone-permissions"></a>
+ No additional permissions are required to use the sample code on this page\.

## Using `pendulum` in a DAG<a name="samples-plugins-timezone-dag"></a>

1.  In your command promt, navigate to the directory where your DAGs are sotred\. 

   ```
   $ cd dags
   ```

1.  Copy the content of the following example and save as `tz-aware-dag.py`\. This code sample uses [Pendulum](https://pypi.org/project/pendulum/), a Python library for working with timezone\-aware datetime, to change the default timezone \(UTC\+0\) in which the DAG runs\. 

   ```
   from airflow import DAG
   from airflow.operators.bash_operator import BashOperator
   from datetime import datetime, timedelta
   # Import the Pendulum library.
   import pendulum
   
   # Instantiate Pendulum and set your timezone.
   local_tz = pendulum.timezone("America/Los_Angeles")
   
   default_args = {
       "owner": "airflow",
       "depends_on_past": False,
       # Set the DAGs timezone by passing the timezone-aware Pendulum object.
       "start_date": datetime(2022, 1, 1, tzinfo=local_tz),
       "email": ["airflow@airflow.com"],
       "email_on_failure": False,
       "email_on_retry": False,
       "retries": 1,
       "retry_delay": timedelta(minutes=5)
   }
   
   # The following cron runs the DAG at 12PM every day in the specified timezone.
   dag = DAG("tz_test", default_args=default_args, schedule_interval="* 12 * * *")
   
   t1 = BashOperator(task_id="print_date", bash_command="echo 'Test DAG is running...'", dag=dag)
   ```

1.  Upload the new timezone\-aware DAG to your environment's Amazon S3 bucket\. 

   ```
   $ aws s3 cp tz-aware-dag.py s3://<your-mwaa-bucket>/<your-dag-folder>/
   ```

## Create a plugin to change the timezone in Airflow logs<a name="samples-plugins-timezone-custom-plugin"></a>

Apache Airflow will run the Python files in the `plugins` directory at start\-up\. With the following plugin, you can override the executor's timezone, which modifies the timezone in which Apache Airflow writes logs\.

1. Create a directory names `plugins` for your custom plugin, and navigate to the directory\. For example:

   ```
   $ mkdir plugins
   $ cd plugins
   ```

1. Copy the contents of the following code sample and save locally as `dag-timezone-plugin.py` in the `plugins` folder\.

   ```
   import time
   import os
   
   os.environ['TZ'] = 'America/Los_Angeles'
   time.tzset()
   ```

1.  In the `plugins` directory, create an empty Python file named `__init__.py`\. Your `plugins` directory should be similar to the following: 

   ```
   plugins/
   |-- __init__.py
   |-- dag-timezone-plugin.py
   ```

## Create a `plugins.zip`<a name="samples-plugins-timezone-plugins-zip"></a>

The following steps show how to create `plugins.zip`\. The content of this example can be combined with other plugins and binaries into a single `plugins.zip` file\.

1. In your command prompt, navigate to the `plugins` directory from the previous step\. For example:

   ```
   cd plugins
   ```

1. Zip the contents within your `plugins` directory\.

   ```
   zip -r ../plugins.zip ./
   ```

## What's next?<a name="samples-plugins-timezone-plugins-next-up"></a>
+ Learn more about how to upload the `plugins.zip` file in this example to your Amazon S3 bucket in [Installing custom plugins](configuring-dag-import-plugins.md)\.