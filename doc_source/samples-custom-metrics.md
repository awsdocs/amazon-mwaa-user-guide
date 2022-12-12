# Using a DAG to write custom metrics in CloudWatch<a name="samples-custom-metrics"></a>

You can use the following code example to write a directed acyclic graph \(DAG\) that runs a `PythonOperator` to retrieve OS\-level metrics for an Amazon MWAA environment\. The DAG then publishes the data as custom metrics to Amazon CloudWatch\.

 Custom OS\-level metrics provide you with additional visibility about how your environment workers are utilizing resources such as virtual memory and CPU\. You can use this information to select the [environment class](environment-class.md) that best suits your workload\. 

**Topics**
+ [Version](#samples-custom-metrics-version)
+ [Prerequisites](#samples-custom-metrics-prereqs)
+ [Permissions](#samples-custom-metrics-permissions)
+ [Dependencies](#samples-custom-metrics-dependencies)
+ [Code example](#samples-custom-metrics-code)

## Version<a name="samples-custom-metrics-version"></a>
+ You can use the code example on this page with **Apache Airflow v2 and above** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="samples-custom-metrics-prereqs"></a>

To use the code example on this page, you need the following:
+ An [Amazon MWAA environment](get-started.md)\.

## Permissions<a name="samples-custom-metrics-permissions"></a>
+ No additional permissions are required to use the code example on this page\.

## Dependencies<a name="samples-custom-metrics-dependencies"></a>
+ No additional dependencies are required to use the code example on this page\.

## Code example<a name="samples-custom-metrics-code"></a>

1. In your command prompt, navigate to the folder where your DAG code is stored\. For example:

   ```
   cd dags
   ```

1.  Copy the contents of the following code example and save it locally as `dag-custom-metrics.py`\. Replace `MWAA-ENV-NAME` with your environment name\. 

   ```
   from airflow import DAG
   from airflow.operators.python_operator import PythonOperator
   from airflow.utils.dates import days_ago
   from datetime import datetime
   import os,json,boto3,psutil,socket
   
   def publish_metric(client,name,value,cat,unit='None'):
       environment_name = os.getenv("MWAA_ENV_NAME")
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

1.  Run the following AWS CLI command to copy the DAG to your environment's bucket, then trigger the DAG using the Apache Airflow UI\. 

   ```
   $ aws s3 cp your-dag.py s3://your-environment-bucket/dags/
   ```

1. If the DAG runs successfully, you should see something similar to the following in your Apache Airflow logs:

   ```
   [2022-08-16, 10:54:46 UTC] {{logging_mixin.py:109}} INFO - cpu_stats scpustats(ctx_switches=3253992384, interrupts=1964237163, soft_interrupts=492328209, syscalls=0)
   [2022-08-16, 10:54:46 UTC] {{logging_mixin.py:109}} INFO - writing value 16024199168.0 to metric virtual_memory_total
   [2022-08-16, 10:54:46 UTC] {{logging_mixin.py:109}} INFO - {'ResponseMetadata': {'RequestId': 'fad289ac-aa51-46a9-8b18-24e4e4063f4d', 'HTTPStatusCode': 200, 'HTTPHeaders': {'x-amzn-requestid': 'fad289ac-aa51-46a9-8b18-24e4e4063f4d', 'content-type': 'text/xml', 'content-length': '212', 'date': 'Tue, 16 Aug 2022 17:54:45 GMT'}, 'RetryAttempts': 0}}
   [2022-08-16, 10:54:46 UTC] {{logging_mixin.py:109}} INFO - writing value 14356287488.0 to metric virtual_memory_available
   [2022-08-16, 10:54:46 UTC] {{logging_mixin.py:109}} INFO - {'ResponseMetadata': {'RequestId': '6ef60085-07ab-4865-8abf-dc94f90cab46', 'HTTPStatusCode': 200, 'HTTPHeaders': {'x-amzn-requestid': '6ef60085-07ab-4865-8abf-dc94f90cab46', 'content-type': 'text/xml', 'content-length': '212', 'date': 'Tue, 16 Aug 2022 17:54:45 GMT'}, 'RetryAttempts': 0}}
   [2022-08-16, 10:54:46 UTC] {{logging_mixin.py:109}} INFO - writing value 1342296064.0 to metric virtual_memory_used
   [2022-08-16, 10:54:46 UTC] {{logging_mixin.py:109}} INFO - {'ResponseMetadata': {'RequestId': 'd5331438-5d3c-4df2-bc42-52dcf8d60a00', 'HTTPStatusCode': 200, 'HTTPHeaders': {'x-amzn-requestid': 'd5331438-5d3c-4df2-bc42-52dcf8d60a00', 'content-type': 'text/xml', 'content-length': '212', 'date': 'Tue, 16 Aug 2022 17:54:45 GMT'}, 'RetryAttempts': 0}}
   ...
   [2022-08-16, 10:54:46 UTC] {{python.py:152}} INFO - Done. Returned value was: OK
   [2022-08-16, 10:54:46 UTC] {{taskinstance.py:1280}} INFO - Marking task as SUCCESS. dag_id=dag-custom-metrics, task_id=memory_test, execution_date=20220816T175444, start_date=20220816T175445, end_date=20220816T175446
   [2022-08-16, 10:54:46 UTC] {{local_task_job.py:154}} INFO - Task exited with return code 0
   ```