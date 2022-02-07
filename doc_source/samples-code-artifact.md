# Using an AWS CodeArtifact token at runtime<a name="samples-code-artifact"></a>

An AWS CodeArtifact token expires after a maximum expiration limit of twelve hours\. If you're using CodeArtifact, Amazon MWAA requires an active token to install Python dependencies\. To allow Amazon Managed Workflows for Apache Airflow \(MWAA\) to access Python dependencies stored in a Code Artifact repository at runtime, a `requirements.txt` needs to contain an `index url` with the token\. This would require you to update the token in the `requirements.txt` every twelve hours\. A DAG is uploaded to the DAGs folder on your Amazon S3 bucket and doesn't require an environment update\. The following sample walks you through the steps to create a DAG that uses the token in your AWS CodeArtifact repository, rather than in a `requirements.txt` on Amazon MWAA\.

**Topics**
+ [Version](#samples-code-artifact-version)
+ [Prerequisites](#samples-code-artifact-prereqs)
+ [Permissions](#samples-code-artifact-permissions)
+ [Requirements](#samples-hive-dependencies)
+ [Code sample](#samples-code-artifact-code)
+ [What's next?](#samples-code-artifact-next-up)

## Version<a name="samples-code-artifact-version"></a>
+ The sample code on this page can be used with **Apache Airflow v1** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="samples-code-artifact-prereqs"></a>

To use the sample code on this page, you'll need the following:
+ An [Amazon MWAA environment](get-started.md)\.

## Permissions<a name="samples-code-artifact-permissions"></a>
+ No additional permissions are required to use the sample code on this page\.

## Requirements<a name="samples-hive-dependencies"></a>

To use the sample code on this page, add the following dependencies\. To learn more, see [Installing Python dependencies](working-dags-dependencies.md)\.

You'll need to add the following files to your Amazon S3 bucket\.

1. Add a `codeartifact.txt` file to the `dags` folder in your Amazon S3 bucket\. For example:

   ```
   --index-url https://aws:auth_token@your_domain-111122223333.d.codeartifact.us-west-2.amazonaws.com/pypi/your_repo/simple/
   ```

   

1. Add a `requirements.txt` file in your Amazon S3 bucket with the following line\. For example:

   ```
   -r /usr/local/airflow/dags/codeartifact.txt
   ```

## Code sample<a name="samples-code-artifact-code"></a>

The following steps describe how to create the DAG code that updates the token\.

1. In your command prompt, navigate to the directory where your DAG code is stored\. For example:

   ```
   cd dags
   ```

1. Copy the contents of the following code sample and save locally as `code_artifact.py`\.

   ```
   from airflow import DAG
   import boto3
   from airflow.utils.dates import datetime,timedelta
   from airflow.operators.python_operator import PythonOperator
   from airflow.hooks.base_hook import BaseHook
   
   def fn_get_authorization_token(**kwargs):
       ti = kwargs['ti']
       client = boto3.client('codeartifact')
       response = client.get_authorization_token(
           domain='your_domain',
           domainOwner='111122223333',
           durationSeconds=43200 #(12 hours)
       )
       ti.xcom_push(key='authorizationToken', value=response['authorizationToken'])
   
   def fn_update_index_url(**kwargs):
       ti = kwargs['ti']
       s3_conn_id = 's3_conn'
       token=ti.xcom_pull(key='authorizationToken', task_ids='get_authorization_token')
       conn = BaseHook.get_connection(s3_conn_id)
       s3_client = boto3.client('s3',
                             aws_access_key_id=conn.login,
                             aws_secret_access_key=conn.password
                             )
       new_index_url = '--index-url https://aws:{}@your_domain-111122223333.d.codeartifact.us-west-2.amazonaws.com/pypi/your_repo/simple/'.format(token)
       s3_client.put_object(Body=new_index_url, Bucket=conn.schema, Key='dags/codeartifact.txt')
   
   
   dag = DAG(
       'exapmple_update_codeartifact_token',
       #Runs every 10 hours rather than 12 hours as a best practice
       schedule_interval='0 */10 * * *',
       dagrun_timeout=timedelta(minutes=5),
       start_date=datetime(2021, 4, 12),
       catchup=False,
       max_active_runs=1)
   
   get_authorization_token = PythonOperator(
       task_id='get_authorization_token',
       python_callable=fn_get_authorization_token,
       provide_context=True,
       dag=dag)
   
   update_index_url = PythonOperator(
       task_id='update_index_url',
       python_callable=fn_update_index_url,
       provide_context=True,
       dag=dag)
   
   get_authorization_token >> update_index_url
   ```

## What's next?<a name="samples-code-artifact-next-up"></a>
+ Learn how to upload the `requirements.txt` file in this example to your Amazon S3 bucket in [Installing Python dependencies](working-dags-dependencies.md)\.
+ Learn how to upload the DAG code in this example to the `dags` folder in your Amazon S3 bucket in [Adding or updating DAGs](configuring-dag-folder.md)\.
+ Learn more about how to upload the `plugins.zip` file in this example to your Amazon S3 bucket in [Installing custom plugins](configuring-dag-import-plugins.md)\.