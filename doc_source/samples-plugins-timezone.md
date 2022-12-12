# Changing a DAG's timezone on Amazon MWAA<a name="samples-plugins-timezone"></a>

 Apache Airflow schedules your directed acyclic graph \(DAG\) in UTC\+0 by default\. The following steps show how you can change the timezone in which Amazon MWAA runs your DAGs with [Pendulum](https://pypi.org/project/pendulum/)\. Optionally, this topic demonstrates how you can create a custom plugin to change the timezone for your environment's Apache Airflow logs\. 

**Topics**
+ [Version](#samples-plugins-timezone-version)
+ [Prerequisites](#samples-plugins-timezone-prerequisites)
+ [Permissions](#samples-plugins-timezone-permissions)
+ [Create a plugin to change the timezone in Airflow logs](#samples-plugins-timezone-custom-plugin)
+ [Create a `plugins.zip`](#samples-plugins-timezone-plugins-zip)
+ [Code sample](#samples-plugins-timezone-dag)
+ [What's next?](#samples-plugins-timezone-plugins-next-up)

## Version<a name="samples-plugins-timezone-version"></a>
+ You can use the code example on this page with **Apache Airflow v2 and above** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="samples-plugins-timezone-prerequisites"></a>

To use the sample code on this page, you'll need the following:
+ An [Amazon MWAA environment](get-started.md)\.

## Permissions<a name="samples-plugins-timezone-permissions"></a>
+ No additional permissions are required to use the code example on this page\.

## Create a plugin to change the timezone in Airflow logs<a name="samples-plugins-timezone-custom-plugin"></a>

Apache Airflow will run the Python files in the `plugins` directory at start\-up\. With the following plugin, you can override the executor's timezone, which modifies the timezone in which Apache Airflow writes logs\.

1. Create a directory named `plugins` for your custom plugin, and navigate to the directory\. For example:

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

1. Upload `plugins.zip` to your S3 bucket

   ```
   $ aws s3 cp plugins.zip s3://your-mwaa-bucket/
   ```

## Code sample<a name="samples-plugins-timezone-dag"></a>

To change the default timezone \(UTC\+0\) in which the DAG runs, we'll use a library called [Pendulum](https://pypi.org/project/pendulum/), a Python library for working with timezone\-aware datetime\.

1.  In your command prompt, navigate to the directory where your DAGs are stored\. For example: 

   ```
   $ cd dags
   ```

1.  Copy the content of the following example and save as `tz-aware-dag.py`\. 

   ```
   from airflow import DAG
   from airflow.operators.bash_operator import BashOperator
   from datetime import datetime, timedelta
   # Import the Pendulum library.
   import pendulum
   
   # Instantiate Pendulum and set your timezone.
   local_tz = pendulum.timezone("America/Los_Angeles")
   
   with DAG(
       dag_id = "tz_test",
       schedule_interval="0 12 * * *",
       catchup=False,
       start_date=datetime(2022, 1, 1, tzinfo=local_tz)
   ) as dag:
       bash_operator_task = BashOperator(
           task_id="tz_aware_task",
           dag=dag,
           bash_command="date"
       )
   ```

1.  Run the following AWS CLI command to copy the DAG to your environment's bucket, then trigger the DAG using the Apache Airflow UI\. 

   ```
   $ aws s3 cp your-dag.py s3://your-environment-bucket/dags/
   ```

1. If successful, you'll output similar to the following in the task logs for the `tz_aware_task` in the `tz_test` DAG:

   ```
   [2022-08-01, 12:00:00 PDT] {{subprocess.py:74}} INFO - Running command: ['bash', '-c', 'date']
   [2022-08-01, 12:00:00 PDT] {{subprocess.py:85}} INFO - Output:
   [2022-08-01, 12:00:00 PDT] {{subprocess.py:89}} INFO - Mon Aug  1 12:00:00 PDT 2022
   [2022-08-01, 12:00:00 PDT] {{subprocess.py:93}} INFO - Command exited with return code 0
   [2022-08-01, 12:00:00 PDT] {{taskinstance.py:1280}} INFO - Marking task as SUCCESS. dag_id=tz_test, task_id=tz_aware_task, execution_date=20220801T190033, start_date=20220801T190035, end_date=20220801T190035
   [2022-08-01, 12:00:00 PDT] {{local_task_job.py:154}} INFO - Task exited with return code 0
   [2022-08-01, 12:00:00 PDT] {{local_task_job.py:264}} INFO - 0 downstream tasks scheduled from follow-on schedule check
   ```

## What's next?<a name="samples-plugins-timezone-plugins-next-up"></a>
+ Learn more about how to upload the `plugins.zip` file in this example to your Amazon S3 bucket in [Installing custom plugins](configuring-dag-import-plugins.md)\.