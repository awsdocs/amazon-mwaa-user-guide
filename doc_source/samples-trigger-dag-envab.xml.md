# Invoking DAGs in different Amazon MWAA environments<a name="samples-trigger-dag-envab.xml"></a>

The following sample code creates an Apache Airflow CLI token, then uses a DAG in one Amazon Managed Workflows for Apache Airflow \(MWAA\) environment to invoke a DAG in a different environment\.

**Topics**
+ [Version](#samples-trigger-dag-envab.xml-version)
+ [Prerequisites](#samples-lambda-prereqs)
+ [Permissions](#samples-lambda-permissions)
+ [Dependencies](#samples-sql-server-dependencies)
+ [Code sample](#samples-trigger-dag-envab-code)
+ [What's next?](#samples-trigger-dag-envab.xml-next-up)

## Version<a name="samples-trigger-dag-envab.xml-version"></a>
+ The sample code on this page can be used with **Apache Airflow v1\.10\.12**\.

## Prerequisites<a name="samples-lambda-prereqs"></a>

To use the sample code on this page, you'll need the following:
+ The **Public network** option for your [Amazon MWAA environment](get-started.md)\.

## Permissions<a name="samples-lambda-permissions"></a>

To use the sample code on this page, your AWS account needs access to the `AmazonMWAAAirflowCliAccess` policy\. To learn more, see [Apache Airflow CLI policy: AmazonMWAAAirflowCliAccess](access-policies.md)\.

## Dependencies<a name="samples-sql-server-dependencies"></a>

To use the sample code on this page, add the following dependency to the `requirements.txt` on your Amazon S3 bucket\. To learn more, see [Installing Python dependencies](working-dags-dependencies.md)\.

```
boto3 >= 1.17.4
```

## Code sample<a name="samples-trigger-dag-envab-code"></a>

The following sample code assumes you're using a DAG in your current environment \(such as "Environment A"\) to invoke a DAG your other environment \(such as "Environment B"\)\. Copy the sample code and substitute the placeholders with the following:
+ The name of the other environment where you want to invoke the DAG in `EnvB-Environment-Name`\.
+ The name of the other DAG you want to invoke in `EnvB-dag-id`\.

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
from airflow.operators.python_operator import PythonOperator
import boto3
from airflow.utils.dates import days_ago
from datetime import timedelta
import os, requests, json
DAG_ID = os.path.basename(__file__).replace(".py", "")
DEFAULT_ARGS = {
    'owner': 'airflow',
    'depends_on_past': False,
    'email': ['airflow@example.com']
}
def triggerDagFn(**kwargs):
    client = boto3.client('mwaa')
    token = client.create_cli_token(Name='EnvB-Environment-Name')
    url = "https://{0}/aws_mwaa/cli".format(token['WebServerHostname'])
    body = 'trigger_dag EnvB-dag-id'
    headers = {
        'Authorization' : 'Bearer '+token['CliToken'],
        'Content-Type': 'text/plain'
        }
    r = requests.post(url, data=body, headers=headers)
    print(r.content)
    return r

with DAG(
    dag_id=DAG_ID,
    default_args=DEFAULT_ARGS,
    dagrun_timeout=timedelta(hours=2),
    start_date=days_ago(1),
    schedule_interval='@once'
) as dag:
    triggerDag = PythonOperator(
        task_id="triggerDag",
        python_callable=triggerDagFn,
        provide_context=True 
    )
```

## What's next?<a name="samples-trigger-dag-envab.xml-next-up"></a>
+ Learn how to upload code to a DAGs folder in [Adding or updating DAGs](configuring-dag-folder.md)\.
+ Learn how to upload a `requirements.txt` file in [Installing Python dependencies](working-dags-dependencies.md)\.