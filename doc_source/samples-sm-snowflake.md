# Using a secret key in AWS Secrets Manager for an Apache Airflow Snowflake connection<a name="samples-sm-snowflake"></a>

The following sample calls AWS Secrets Manager to get a secret key for an Apache Airflow Snowflake connection on Amazon Managed Workflows for Apache Airflow \(MWAA\)\. It assumes you've completed the steps in [Configuring an Apache Airflow connection using a Secrets Manager secret](connections-secrets-manager.md)\.

**Topics**
+ [Version](#samples-sm-snowflake-version)
+ [Prerequisites](#samples-sm-snowflake-prereqs)
+ [Permissions](#samples-sm-snowflake-permissions)
+ [Requirements](#samples-sm-snowflake-dependencies)
+ [Code sample](#samples-sm-snowflake-code)
+ [What's next?](#samples-sm-snowflake-next-up)

## Version<a name="samples-sm-snowflake-version"></a>
+ You can use the code example on this page with **Apache Airflow v2 and above** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="samples-sm-snowflake-prereqs"></a>

To use the sample code on this page, you'll need the following:
+ The Secrets Manager backend as an Apache Airflow configuration option as shown in [Configuring an Apache Airflow connection using a Secrets Manager secret](connections-secrets-manager.md)\.
+ An Apache Airflow connection string in Secrets Manager as shown in [Configuring an Apache Airflow connection using a Secrets Manager secret](connections-secrets-manager.md)\.

## Permissions<a name="samples-sm-snowflake-permissions"></a>
+ Secrets Manager permissions as shown in [Configuring an Apache Airflow connection using a Secrets Manager secret](connections-secrets-manager.md)\.

## Requirements<a name="samples-sm-snowflake-dependencies"></a>

To use the sample code on this page, add the following dependencies to your `requirements.txt`\. To learn more, see [Installing Python dependencies](working-dags-dependencies.md)\.

```
apache-airflow-providers-snowflake==1.3.0
```

## Code sample<a name="samples-sm-snowflake-code"></a>

The following steps describe how to create the DAG code that calls Secrets Manager to get the secret\.

1. In your command prompt, navigate to the directory where your DAG code is stored\. For example:

   ```
   cd dags
   ```

1. Copy the contents of the following code sample and save locally as `snowflake_connection.py`\.

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
   from airflow import DAG
   from airflow.providers.snowflake.operators.snowflake import SnowflakeOperator
   from airflow.utils.dates import days_ago
   
   snowflake_query = [
       """use warehouse "MY_WAREHOUSE";""",
       """select * from "SNOWFLAKE_SAMPLE_DATA"."WEATHER"."WEATHER_14_TOTAL" limit 100;""",
   ]
   
   with DAG(dag_id='snowflake_test', schedule_interval=None, catchup=False, start_date=days_ago(1)) as dag:
       snowflake_select = SnowflakeOperator(
           task_id="snowflake_select",
           sql=snowflake_query,
           snowflake_conn_id="snowflake_conn",
       )
   ```

## What's next?<a name="samples-sm-snowflake-next-up"></a>
+ Learn how to upload the DAG code in this example to the `dags` folder in your Amazon S3 bucket in [Adding or updating DAGs](configuring-dag-folder.md)\.