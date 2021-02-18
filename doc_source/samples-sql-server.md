# Using Amazon MWAA with Amazon RDS Microsoft SQL Server<a name="samples-sql-server"></a>

You can use Amazon Managed Workflows for Apache Airflow \(MWAA\) to connect to an [Amazon RDS Microsoft SQL Server](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_SQLServer.html)\. The following sample code uses Amazon Managed Workflows for Apache Airflow \(MWAA\) DAGs to connect to and execute queries on an Amazon RDS Microsoft SQL Server\.

**Topics**
+ [Version](#samples-sql-server-version)
+ [Prerequisites](#samples-sql-server-prereqs)
+ [Dependencies](#samples-sql-server-dependencies)
+ [Code sample](#samples-sql-server-code)
+ [What's next?](#samples-sql-server-next-up)

## Version<a name="samples-sql-server-version"></a>
+ The sample code on this page can be used with **Apache Airflow v1\.10\.12**\.

## Prerequisites<a name="samples-sql-server-prereqs"></a>

To use the sample code on this page, you'll need the following:
+ An [Amazon MWAA environment](get-started.md)\.
+ Amazon MWAA and the RDS Microsoft SQL Server are running in the same VPC
+ VPC security groups of Amazon MWAA and the server are configured with the following connections:
  + An inbound port of `1433` is open for RDS from MWAA's security group
  + Or an outbound port of `1433` is open from MWAA to RDS
+ Airflow Connection for MSSQL reflects the hostname, port, username and password from the RDS SQL Server DB created in previous process

## Dependencies<a name="samples-sql-server-dependencies"></a>

To use the sample code in this section, add the following dependency to your `requirements.txt`\. To learn more, see [Installing Python dependencies](working-dags-dependencies.md)

```
apache-airflow[mssql]==1.10.12
```

## Code sample<a name="samples-sql-server-code"></a>

Copy the sample code to the `dags` folder in your Amazon S3 bucket\. To learn more, see [Adding or updating DAGs](configuring-dag-folder.md)\.

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
import pymssql
import logging
import sys
from airflow import DAG
from datetime import datetime
from airflow.operators.mssql_operator import MsSqlOperator
from airflow.operators.python_operator import PythonOperator

default_args = {
    'owner': 'aws',
    'depends_on_past': False,
    'start_date': datetime(2019, 2, 20),
    'provide_context': True
}

dag = DAG(
    'mssql_conn_example', default_args=default_args, schedule_interval=None)
    
drop_db = MsSqlOperator(
   task_id="drop_db",
   sql="DROP DATABASE IF EXISTS testdb;",
   mssql_conn_id="mssql_default",
   autocommit=True,
   dag=dag
)

create_db = MsSqlOperator(
   task_id="create_db",
   sql="create database testdb;",
   mssql_conn_id="mssql_default",
   autocommit=True,
   dag=dag
)

create_table = MsSqlOperator(
   task_id="create_table",
   sql="CREATE TABLE testdb.dbo.pet (name VARCHAR(20), owner VARCHAR(20));",
   mssql_conn_id="mssql_default",
   autocommit=True,
   dag=dag
)

insert_into_table = MsSqlOperator(
   task_id="insert_into_table",
   sql="INSERT INTO testdb.dbo.pet VALUES ('Olaf', 'Disney');",
   mssql_conn_id="mssql_default",
   autocommit=True,
   dag=dag
)

def select_pet(**kwargs):
   try:
        conn = pymssql.connect(
            server='sampledb.<xxxxxx>.<region>.rds.amazonaws.com',
            user='admin',
            password='<yoursupersecretpassword>',
            database='testdb'
        )
        
        # Create a cursor from the connection
        cursor = conn.cursor()
        cursor.execute("SELECT * from testdb.dbo.pet")
        row = cursor.fetchone()
        
        if row:
            print(row)
   except:
      logging.error("Error when creating pymssql database connection: %s", sys.exc_info()[0])

select_query = PythonOperator(
    task_id='select_query',
    python_callable=select_pet,
    dag=dag,
)

drop_db >> create_db >> create_table >> insert_into_table >> select_query
```

## What's next?<a name="samples-sql-server-next-up"></a>
+ Learn how to upload a `requirements.txt` file in [Installing Python dependencies](working-dags-dependencies.md)\.
+ Explore example scripts and other [pymssql examples](https://pymssql.readthedocs.io/en/stable/pymssql_examples.html)\.
+ Learn more about executing SQL code in a specific Microsoft SQL database using the [mssql\_operator](https://airflow.apache.org/docs/apache-airflow/1.10.12/_api/airflow/operators/mssql_operator/index.html?highlight=mssqloperator#airflow.operators.mssql_operator.MsSqlOperator) in the *Apache Airflow reference guide*\.