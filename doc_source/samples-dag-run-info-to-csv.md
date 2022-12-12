# Exporting environment metadata to CSV files on Amazon S3<a name="samples-dag-run-info-to-csv"></a>

 The following code example shows how you can create a directed acyclic graph \(DAG\) that querries the database for a range of DAG run information, and writes the data to `.csv` files stored on Amazon S3\. 

 You might want to export information from the your environment's Aurora PostgreSQL database in order to inspect the data locally, archive them in object storage, or combine them with tools like the [Amazon S3 to Amazon Redshift operator](https://airflow.apache.org/docs/apache-airflow-providers-amazon/stable/operators/s3_to_redshift.html) and the [database cleanup](samples-database-cleanup.md), in order to move Amazon MWAA metadata out of the environment, but preserve them for future analysis\. 

 You can query the database for anyl of the objects listed in [Apache Airflow models](https://github.com/apache/airflow/tree/v2-0-stable/airflow/models)\. This code sample uses three models, `DagRun`, `TaskFail`, and `TaskInstance`, which provide information relevant to DAG runs\. 

**Topics**
+ [Version](#samples-dag-run-info-to-csv-version)
+ [Prerequisites](#samples-dag-run-info-to-csv-prereqs)
+ [Permissions](#samples-dag-run-info-to-csv-permissions)
+ [Requirements](#samples-dag-run-info-to-csv-dependencies)
+ [Code sample](#samples-dag-run-info-to-csv-code)

## Version<a name="samples-dag-run-info-to-csv-version"></a>
+ You can use the code example on this page with **Apache Airflow v2 and above** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="samples-dag-run-info-to-csv-prereqs"></a>

To use the sample code on this page, you'll need the following:
+ An [Amazon MWAA environment](get-started.md)\.
+  A [new Amazon S3 bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/create-bucket-overview.html) where you want to export your metadata information\. 

## Permissions<a name="samples-dag-run-info-to-csv-permissions"></a>

 Amazon MWAA needs permission for the Amazon S3 action `s3:PutObject` to write the queried metadata information to Amazon S3\. Add the following policy statement to your environment's execution role\. 

```
{
  "Effect": "Allow",
  "Action": "s3:PutObject*",
  "Resource": "arn:aws:s3:::your-new-export-bucket"
}
```

 This policy limits write access to only *your\-new\-export\-bucket*\. 

## Requirements<a name="samples-dag-run-info-to-csv-dependencies"></a>
+ To use this code example with Apache Airflow v2, no additional dependencies are required\. The code uses the [Apache Airflow v2 base install](https://github.com/aws/aws-mwaa-local-runner/blob/main/docker/config/requirements.txt) on your environment\.

## Code sample<a name="samples-dag-run-info-to-csv-code"></a>

 The following steps describe how you can create a DAG that queries the Aurora PostgreSQL and writes the result to your new Amazon S3 bucket\. 

1. In your terminal, navigate to the directory where your DAG code is stored\. For example:

   ```
   cd dags
   ```

1.  Copy the contents of the following code example and save it locally as `metadata_to_csv.py`\. You can change the value assigned to `MAX_AGE_IN_DAYS` to control the age of the oldest records your DAG queries from the metadata database\. 

   ```
   from airflow.decorators import dag, task
   from airflow import settings
   import os
   import boto3
   from airflow.utils.dates import days_ago
   from airflow.models import DagRun, TaskFail, TaskInstance
   import csv, re
   from io import StringIO
   
   DAG_ID = os.path.basename(__file__).replace(".py", "")
   
   MAX_AGE_IN_DAYS = 30 
   S3_BUCKET = '<your-export-bucket>'
   S3_KEY = 'files/export/{0}.csv' 
   
   # You can add other objects to export from the metadatabase,
   OBJECTS_TO_EXPORT = [
       [DagRun,DagRun.execution_date], 
       [TaskFail,TaskFail.execution_date], 
       [TaskInstance, TaskInstance.execution_date],
   ]
    
   @task()
   def export_db_task(**kwargs):
       session = settings.Session()
       print("session: ",str(session))
    
       oldest_date = days_ago(MAX_AGE_IN_DAYS)
       print("oldest_date: ",oldest_date)
   
       s3 = boto3.client('s3')
   
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
               s3.put_object(Bucket=S3_BUCKET, Key=outkey, Body=f.getvalue())
    
   @dag(
       dag_id=DAG_ID,
       schedule_interval=None,
       start_date=days_ago(1),
       )
   def export_db():
       t = export_db_task()
   
   metadb_to_s3_test = export_db()
   ```

1.  Run the following AWS CLI command to copy the DAG to your environment's bucket, then trigger the DAG using the Apache Airflow UI\. 

   ```
   $ aws s3 cp your-dag.py s3://your-environment-bucket/dags/
   ```

1. If successful, you'll output similar to the following in the task logs for the `export_db` task:

   ```
   [2022-01-01, 12:00:00 PDT] {{logging_mixin.py:109}} INFO - type <class 'sqlalchemy.orm.query.Query'>
   [2022-01-01, 12:00:00 PDT] {{logging_mixin.py:109}} INFO - class airflow.models.dagrun.DagRun : [your-tasks]
   [2022-01-01, 12:00:00 PDT] {{logging_mixin.py:109}} INFO - type <class 'sqlalchemy.orm.query.Query'>
   [2022-01-01, 12:00:00 PDT] {{logging_mixin.py:109}} INFO - class airflow.models.taskfail.TaskFail :  [your-tasks]
   [2022-01-01, 12:00:00 PDT] {{logging_mixin.py:109}} INFO - type <class 'sqlalchemy.orm.query.Query'>
   [2022-01-01, 12:00:00 PDT] {{logging_mixin.py:109}} INFO - class airflow.models.taskinstance.TaskInstance :  [your-tasks]
   [2022-01-01, 12:00:00 PDT] {{python.py:152}} INFO - Done. Returned value was: OK
   [2022-01-01, 12:00:00 PDT] {{taskinstance.py:1280}} INFO - Marking task as SUCCESS. dag_id=metadb_to_s3, task_id=export_db, execution_date=20220101T000000, start_date=20220101T000000, end_date=20220101T000000
   [2022-01-01, 12:00:00 PDT] {{local_task_job.py:154}} INFO - Task exited with return code 0
   [2022-01-01, 12:00:00 PDT] {{local_task_job.py:264}} INFO - 0 downstream tasks scheduled from follow-on schedule check
   ```

    You can now access and download the exported `.csv` files in your new Amazon S3 bucket in `/files/export/`\. 