# Writing DAG run information to a CSV file on Amazon S3<a name="samples-dag-run-info-to-csv"></a>

 You might want to export information from the Aurora PostgreSQL database in order to inspect the data locally, archive them in object storage, or combine them with tools like the [Amazon S3 to Amazon Redshift operator](https://airflow.apache.org/docs/apache-airflow-providers-amazon/stable/operators/s3_to_redshift.html) and the [database cleanup](samples-database-cleanup.md), in order to move Amazon MWAA metadata out of the environment, but preserve them for future analysis\. The following code sample shows how you can create a DAG that querries the database for a range of DAG run information, and writes the data to a `CSV` file stored on Amazon S3\. 

 You can query the database for any or all of the objects listed in [Apache Airflow models](https://github.com/apache/airflow/tree/v2-0-stable/airflow/models)\. This code sample uses three models, `DagRun`, `TaskFail`, and `TaskInstance`, which provide information relevant to DAG runs\. 

**Topics**
+ [Version](#samples-dag-run-info-to-csv-version)
+ [Prerequisites](#samples-dag-run-info-to-csv-prereqs)
+ [Permissions](#samples-dag-run-info-to-csv-permissions)
+ [Dependencies](#samples-dag-run-info-to-csv-dependencies)
+ [Code sample](#samples-dag-run-info-to-csv-code)

## Version<a name="samples-dag-run-info-to-csv-version"></a>
+ The sample code on this page can be used with **Apache Airflow v2\.0\.2** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="samples-dag-run-info-to-csv-prereqs"></a>

To use the sample code on this page, you'll need the following:
+ An [Amazon MWAA environment](get-started.md)\.

## Permissions<a name="samples-dag-run-info-to-csv-permissions"></a>
+ No additional permissions are required to use the sample code on this page\.

## Dependencies<a name="samples-dag-run-info-to-csv-dependencies"></a>
+ No additional dependencies are required to use the sample code on this page\. The sample code uses the [Apache Airflow v2\.0\.2 base install](https://github.com/aws/aws-mwaa-local-runner/blob/main/docker/config/requirements.txt) on your environment\.

## Code sample<a name="samples-dag-run-info-to-csv-code"></a>

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

from airflow import DAG, settings
 
from airflow.operators.python import PythonOperator
from airflow.utils.dates import days_ago
from airflow.models import DAG, DagRun, TaskFail, TaskInstance
from airflow.providers.amazon.aws.hooks.s3 import S3Hook

import csv, re
from io import StringIO

MAX_AGE_IN_DAYS = 30 
S3_BUCKET = 'my-export-bucket'
S3_KEY = 'files/export/{0}.csv' 

OBJECTS_TO_EXPORT = [
    [DagRun,DagRun.execution_date], 
    [TaskFail,TaskFail.execution_date], 
    [TaskInstance, TaskInstance.execution_date],
]
 
def export_db_fn(**kwargs):
    session = settings.Session()
    print("session: ",str(session))
 
    oldest_date = days_ago(MAX_AGE_IN_DAYS)
    print("oldest_date: ",oldest_date)
    s3_hook = S3Hook()
    s3_client = s3_hook.get_conn()
    for x in OBJECTS_TO_EXPORT:
        query = session.query(x[0]).filter(x[1] >= days_ago(MAX_AGE_IN_DAYS))
        print("type",type(query))
        allrows=query.all()
        name=re.sub("[<>']", "", str(x[0]))
        print(name,": ",str(allrows))
        if len(allrows) > 0:
            outfileStr=""
            f = StringIO(outfileStr)
            w = csv.DictWriter(f, vars(allrows[0]).keys())
            w.writeheader()
            for y in allrows:
                w.writerow(vars(y))
            outkey = S3_KEY.format(name[6:])
            s3_client.put_object(Bucket=S3_BUCKET, Key=outkey, Body=f.getvalue())
 
    return "OK"
 
with DAG(dag_id="db_export_dag", schedule_interval=None, catchup=False, start_date=days_ago(1)) as dag:
    export_db = PythonOperator(
        task_id="export_db",
        python_callable=export_db_fn,
        provide_context=True     
    )
```