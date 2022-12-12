# Using Amazon MWAA with Amazon RDS for Microsoft SQL Server<a name="samples-sql-server"></a>

You can use Amazon Managed Workflows for Apache Airflow \(MWAA\) to connect to an [RDS for SQL Server](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_SQLServer.html)\. The following sample code uses DAGs on an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment to connect to and execute queries on an Amazon RDS for Microsoft SQL Server\.

**Topics**
+ [Version](#samples-sql-server-version)
+ [Prerequisites](#samples-sql-server-prereqs)
+ [Dependencies](#samples-sql-server-dependencies)
+ [Apache Airflow v2 connection](#samples-sql-server-conn)
+ [Code sample](#samples-sql-server-code)
+ [What's next?](#samples-sql-server-next-up)

## Version<a name="samples-sql-server-version"></a>
+ The sample code on this page can be used with **Apache Airflow v1** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.
+ You can use the code example on this page with **Apache Airflow v2 and above** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="samples-sql-server-prereqs"></a>

To use the sample code on this page, you'll need the following:
+ An [Amazon MWAA environment](get-started.md)\.
+ Amazon MWAA and the RDS for SQL Server are running in the same Amazon VPC/
+ VPC security groups of Amazon MWAA and the server are configured with the following connections:
  + An inbound rule for the port `1433` open for Amazon RDS in Amazon MWAA's security group
  + Or an outbound rule for the port of `1433` open from Amazon MWAA to RDS
+ Apache Airflow Connection for RDS for SQL Server reflects the hostname, port, username and password from the Amazon RDS SQL server database created in previous process\.

## Dependencies<a name="samples-sql-server-dependencies"></a>

To use the sample code in this section, add the following dependency to your `requirements.txt`\. To learn more, see [Installing Python dependencies](working-dags-dependencies.md)

------
#### [ Apache Airflow v2 ]

```
apache-airflow-providers-microsoft-mssql==1.0.1
apache-airflow-providers-odbc==1.0.1
pymssql==2.2.1
```

------
#### [ Apache Airflow v1 ]

```
apache-airflow[mssql]==1.10.12
```

------

## Apache Airflow v2 connection<a name="samples-sql-server-conn"></a>

If you're using a connection in Apache Airflow v2, ensure the Airflow connection object includes the following key\-value pairs:

1. **Conn Id: ** mssql\_default

1. **Conn Type: ** Amazon Web Services

1. **Host: ** `YOUR_DB_HOST`

1. **Schema: **

1. **Login: ** admin

1. **Password: ** 

1. **Port: ** 1433

1. **Extra: **

## Code sample<a name="samples-sql-server-code"></a>

1. In your command prompt, navigate to the directory where your DAG code is stored\. For example:

   ```
   cd dags
   ```

1. Copy the contents of the following code sample and save locally as `sql-server.py`\. 

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
+ Learn how to upload the `requirements.txt` file in this example to your Amazon S3 bucket in [Installing Python dependencies](working-dags-dependencies.md)\.
+ Learn how to upload the DAG code in this example to the `dags` folder in your Amazon S3 bucket in [Adding or updating DAGs](configuring-dag-folder.md)\.
+ Explore example scripts and other [pymssql module examples](https://pymssql.readthedocs.io/en/stable/pymssql_examples.html)\.
+ Learn more about executing SQL code in a specific Microsoft SQL database using the [mssql\_operator](https://airflow.apache.org/docs/apache-airflow/1.10.12/_api/airflow/operators/mssql_operator/index.html?highlight=mssqloperator#airflow.operators.mssql_operator.MsSqlOperator) in the *Apache Airflow reference guide*\.