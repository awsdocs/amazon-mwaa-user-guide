# Using a secret key in AWS Secrets Manager for an Apache Airflow variable<a name="samples-secrets-manager-var"></a>

The following sample calls AWS Secrets Manager to get a secret key for an Apache Airflow variable on Amazon Managed Workflows for Apache Airflow \(MWAA\)\. It assumes you've completed the steps in [Configuring an Apache Airflow connection using a Secrets Manager secret](connections-secrets-manager.md)\.

**Topics**
+ [Version](#samples-secrets-manager-var-version)
+ [Prerequisites](#samples-secrets-manager-var-prereqs)
+ [Permissions](#samples-secrets-manager-var-permissions)
+ [Requirements](#samples-hive-dependencies)
+ [Code sample](#samples-secrets-manager-var-code)
+ [What's next?](#samples-secrets-manager-var-next-up)

## Version<a name="samples-secrets-manager-var-version"></a>
+ The sample code on this page can be used with **Apache Airflow v1** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.
+ You can use the code example on this page with **Apache Airflow v2 and above** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="samples-secrets-manager-var-prereqs"></a>

To use the sample code on this page, you'll need the following:
+ The Secrets Manager backend as an Apache Airflow configuration option as shown in [Configuring an Apache Airflow connection using a Secrets Manager secret](connections-secrets-manager.md)\.
+ An Apache Airflow variable string in Secrets Manager as shown in [Configuring an Apache Airflow connection using a Secrets Manager secret](connections-secrets-manager.md)\.

## Permissions<a name="samples-secrets-manager-var-permissions"></a>
+ Secrets Manager permissions as shown in [Configuring an Apache Airflow connection using a Secrets Manager secret](connections-secrets-manager.md)\.

## Requirements<a name="samples-hive-dependencies"></a>
+ To use this code example with Apache Airflow v1, no additional dependencies are required\. The code uses the [Apache Airflow v1 base install](https://raw.githubusercontent.com/apache/airflow/constraints-1.10.12/constraints-3.7.txt) on your environment\.
+ To use this code example with Apache Airflow v2, no additional dependencies are required\. The code uses the [Apache Airflow v2 base install](https://github.com/aws/aws-mwaa-local-runner/blob/main/docker/config/requirements.txt) on your environment\.

## Code sample<a name="samples-secrets-manager-var-code"></a>

The following steps describe how to create the DAG code that calls Secrets Manager to get the secret\.

1. In your command prompt, navigate to the directory where your DAG code is stored\. For example:

   ```
   cd dags
   ```

1. Copy the contents of the following code sample and save locally as `secrets-manager-var.py`\.

   ```
   from airflow import DAG
   from airflow.operators.python_operator import PythonOperator
   from airflow.models import Variable
   from airflow.utils.dates import days_ago
   from datetime import timedelta
   import os
   DAG_ID = os.path.basename(__file__).replace(".py", "")
   DEFAULT_ARGS = {
       'owner': 'airflow',
       'depends_on_past': False,
       'email': ['airflow@example.com'],
       'email_on_failure': False,
       'email_on_retry': False,
   }
   def get_variable_fn(**kwargs):
       my_variable_name = Variable.get("test-variable", default_var="undefined")
       print("my_variable_name: ", my_variable_name)
       return my_variable_name
   with DAG(
       dag_id=DAG_ID,
       default_args=DEFAULT_ARGS,
       dagrun_timeout=timedelta(hours=2),
       start_date=days_ago(1),
       schedule_interval='@once',
       tags=['variable']
   ) as dag:
       get_variable = PythonOperator(
           task_id="get_variable",
           python_callable=get_variable_fn,
           provide_context=True
       )
   ```

## What's next?<a name="samples-secrets-manager-var-next-up"></a>
+ Learn how to upload the DAG code in this example to the `dags` folder in your Amazon S3 bucket in [Adding or updating DAGs](configuring-dag-folder.md)\.