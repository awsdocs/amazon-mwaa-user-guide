# Using a DAG to write custom metrics in CloudWatch<a name="samples-custom-metrics"></a>

The following sample shows how to write a DAG that queries the Amazon Aurora PostgreSQL metadata database for an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment and publishes custom metrics to Amazon CloudWatch\.

**Topics**
+ [Version](#samples-custom-metrics-version)
+ [Prerequisites](#samples-custom-metrics-prereqs)
+ [Permissions](#samples-custom-metrics-permissions)
+ [Requirements](#samples-custom-metrics-dependencies)
+ [Code sample](#samples-custom-metrics-code)
+ [What's next?](#samples-custom-metrics-next-up)

## Version<a name="samples-custom-metrics-version"></a>
+ The sample code on this page can be used with **Apache Airflow v1\.10\.12** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.
+ The sample code on this page can be used with **Apache Airflow v2\.0\.2** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="samples-custom-metrics-prereqs"></a>

To use the sample code on this page, you'll need the following:
+ An [Amazon MWAA environment](get-started.md)\.

## Permissions<a name="samples-custom-metrics-permissions"></a>
+ No additional permissions are required to use the sample code on this page\.

## Requirements<a name="samples-custom-metrics-dependencies"></a>
+ No additional dependencies are required to use the sample code on this page\. The sample code uses the [Apache Airflow v1\.10\.12 base install](https://raw.githubusercontent.com/apache/airflow/constraints-1.10.12/constraints-3.7.txt) on your environment\.
+ No additional dependencies are required to use the sample code on this page\. The sample code uses the Apache Airflow v2\.0\.2 base install [https://github\.com/aws/aws\-mwaa\-local\-runner/blob/main/docker/config/requirements\.txt](https://github.com/aws/aws-mwaa-local-runner/blob/main/docker/config/requirements.txt) on your environment\.

## Code sample<a name="samples-custom-metrics-code"></a>

1. In your command prompt, navigate to the directory where your DAG code is stored\. For example:

   ```
   cd dags
   ```

1. Copy the contents of the following code sample and save locally as `dag-custom-metrics.py`\.

   ```
   from airflow import DAG
   from airflow.operators.python_operator import PythonOperator
   from airflow.utils.dates import days_ago
   from datetime import datetime
   import os,json,boto3,psutil,socket
   
   def publish_metric(client,name,value,cat,unit='None'):
       environment_name = os.getenv("AIRFLOW_ENV_NAME")
       value_number=float(value)
       hostname = socket.gethostname()
       ip_address = socket.gethostbyname(hostname)
       print('writing value',value_number,'to metric',name)
       response = client.put_metric_data(
           Namespace='MWAA-Custom',
           MetricData=[
               {
                   'MetricName': name,
                   'Dimensions': [
                       {
                           'Name': 'Environment',
                           'Value': environment_name
                       },
                       {
                           'Name': 'Category',
                           'Value': cat
                       },       
                       {
                           'Name': 'Host',
                           'Value': ip_address
                       },                                     
                   ],
                   'Timestamp': datetime.now(),
                   'Value': value_number,
                   'Unit': unit
               },
           ]
       )
       print(response)
       return response
   
   def python_fn(**kwargs):
       client = boto3.client('cloudwatch')
   
       cpu_stats = psutil.cpu_stats()
       print('cpu_stats', cpu_stats)
   
       virtual = psutil.virtual_memory()
       cpu_times_percent = psutil.cpu_times_percent(interval=0)
   
       publish_metric(client=client, name='virtual_memory_total', cat='virtual_memory', value=virtual.total, unit='Bytes')
       publish_metric(client=client, name='virtual_memory_available', cat='virtual_memory', value=virtual.available, unit='Bytes')
       publish_metric(client=client, name='virtual_memory_used', cat='virtual_memory', value=virtual.used, unit='Bytes')
       publish_metric(client=client, name='virtual_memory_free', cat='virtual_memory', value=virtual.free, unit='Bytes')
       publish_metric(client=client, name='virtual_memory_active', cat='virtual_memory', value=virtual.active, unit='Bytes')
       publish_metric(client=client, name='virtual_memory_inactive', cat='virtual_memory', value=virtual.inactive, unit='Bytes')
       publish_metric(client=client, name='virtual_memory_percent', cat='virtual_memory', value=virtual.percent, unit='Percent')
   
       publish_metric(client=client, name='cpu_times_percent_user', cat='cpu_times_percent', value=cpu_times_percent.user, unit='Percent')
       publish_metric(client=client, name='cpu_times_percent_system', cat='cpu_times_percent', value=cpu_times_percent.system, unit='Percent')
       publish_metric(client=client, name='cpu_times_percent_idle', cat='cpu_times_percent', value=cpu_times_percent.idle, unit='Percent')
   
       return "OK"
   
   
   with DAG(dag_id=os.path.basename(__file__).replace(".py", ""), schedule_interval='*/5 * * * *', catchup=False, start_date=days_ago(1)) as dag:
       t = PythonOperator(task_id="memory_test", python_callable=python_fn, provide_context=True)
   ```

## What's next?<a name="samples-custom-metrics-next-up"></a>
+ Learn how to upload the DAG code in this example to the `dags` folder in your Amazon S3 bucket in [Adding or updating DAGs](configuring-dag-folder.md)\.