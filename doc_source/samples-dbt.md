# Using dbt with Amazon MWAA<a name="samples-dbt"></a>

 This topic demonstrates how you can use dbt and Postgres with Amazon MWAA\. In the following steps, you'll add the required dependencies to your `requirements.txt`, and upload a sample dbt project to your environment's Amazon S3 bucket\. Then, you'll use a sample DAG to verify that Amazon MWAA has installed the dependencies, and finally use the `BashOperator` to run the dbt project\. 

**Topics**
+ [Version](#samples-dbt-version)
+ [Prerequisites](#samples-dbt-prereqs)
+ [Dependencies](#samples-dbt-dependencies)
+ [Upload a dbt project to Amazon S3](#samples-dbt-upload-project)
+ [Use a DAG to verify dbt dependency installation](#samples-dbt-test-dependencies)
+ [Use a DAG to run a dbt project](#samples-dbt-run-project)

## Version<a name="samples-dbt-version"></a>
+ You can use the code example on this page with **Apache Airflow v2 and above** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="samples-dbt-prereqs"></a>

Before you can complete the following steps, you'll need the following:
+ An [Amazon MWAA environment](get-started.md) using Apache Airflow v2\.2\.2\. This sample was written, and tested with v2\.2\.2\. You might need to modify the sample to use with other Apache Airflow versions\.
+  A sample dbt project\. To get started using dbt with Amazon MWAA, you can create a fork and clone the [dbt starter project](https://github.com/dbt-labs/dbt-starter-project) from the dbt\-labs GitHub repository\. 

## Dependencies<a name="samples-dbt-dependencies"></a>

To use Amazon MWAA with dbt, add the following dependency to your `requirements.txt`\. To learn more, see [Installing Python dependencies](working-dags-dependencies.md)

 When your environment completes updating, Amazon MWAA will install the required dbt libraries and additional dependencies, such as `psycopg2`\. 

**Note**  
 The default constraints file provided with Apache Airflow v2\.2\.2 has a conflicting version of `jsonschema` that is not supported by the version of dbt used in this guide\. As such, when using Amazon MWAA with dbt, you can either download and modify the Apache Airflow constraints file into your Amazon S3 DAGs folder, then reference it in your `requirements.txt` file as `--constraint /usr/local/airflow/dags/my-updated-constraint.txt`, or omit `--constraint` from `requirements.txt` as shown in the following\. 

```
json-rpc==1.13.0
minimal-snowplow-tracker==0.0.2
packaging==20.9
networkx==2.6.3 
mashumaro==2.5
sqlparse==0.4.2

logbook==1.5.3
agate==1.6.1
dbt-extractor==0.4.0

pyparsing==2.4.7 
msgpack==1.0.2
parsedatetime==2.6
pytimeparse==1.1.8
leather==0.3.4
pyyaml==5.4.1

# Airflow constraints are jsonschema==3.2.0
jsonschema==3.1.1
hologram==0.0.14
dbt-core==0.21.1

psycopg2-binary==2.8.6
dbt-postgres==0.21.1
dbt-redshift==0.21.1
```

 In the following sections, you'll upload your dbt project directory to Amazon S3 and run a DAG that validates whether Amazon MWAA has successfully installed the required dbt dependencies\. 

## Upload a dbt project to Amazon S3<a name="samples-dbt-upload-project"></a>

 To be able to use a dbt project with your Amazon MWAA environment, you can upload the entire project directory to your environment's `dags` folder\. When the environment updates, Amazon MWAA downloads the dbt directory to the local `usr/local/airflow/dags/` folder\. 

**To upload a dbt project to Amazon S3**

1.  Navigate to the directory where you cloned the dbt starter project\. 

1.  Run the following Amazon S3 AWS CLI command to recursively copy the content of the project to your environment's `dags` folder using the `--recursive` parameter\. The command creates a sub\-directory called `dbt` that you can use for all of your dbt projects\. If the sub\-directory already exists, the project files are copied into the existing directory, and a new directory is not created\. The command also creates a sub\-directory within the `dbt` directory for this specific starter project\. 

   ```
   $ aws s3 cp dbt-starter-project s3://mwaa-bucket/dags/dbt/dbt-starter-project --recursive
   ```

    You can use different names for project sub\-directories to organize multiple dbt projects within the parent `dbt` directory\. 

## Use a DAG to verify dbt dependency installation<a name="samples-dbt-test-dependencies"></a>

 The following DAG uses a `BashOperator` and a bash command to verify whether Amazon MWAA has successfully installed the dbt dependencies specified in `requirements.txt`\. 

```
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.utils.dates import days_ago

with DAG(dag_id="dbt-installation-test", schedule_interval=None, catchup=False, start_date=days_ago(1)) as dag:
    cli_command = BashOperator(
        task_id="bash_command",
        bash_command="/usr/local/airflow/.local/bin/dbt --version"
    )
```

 Do the following to view task logs and verify that dbt and its dependencies have been installed\. 

1.  Navigate to the Amazon MWAA console, then choose **Open Airflow UI** from the list of available environments\. 

1.  On the Apache Airflow UI, find the `dbt-installation-test` DAG from the list, then choose the date under the `Last Run` column to open the last successful task\. 

1.  Using **Graph View**, choose the `bash_command` task to open the task instance details\. 

1.  Choose **Log** to open the task logs, then verify that the logs successfully list the dbt version we specified in `requirements.txt`\. 

## Use a DAG to run a dbt project<a name="samples-dbt-run-project"></a>

 The following DAG uses a `BashOperator` to copy the dbt projects you uploaded to Amazon S3 from the local `usr/local/airflow/dags/` directory to the write\-accessible `/tmp` directory, then runs the dbt project\. The bash commands assume a starter dbt project titled `dbt-starter-project`\. Modify the directory name according to the name of your project directory\. 

```
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.utils.dates import days_ago
import os
DAG_ID = os.path.basename(__file__).replace(".py", "")
with DAG(dag_id=DAG_ID, schedule_interval=None, catchup=False, start_date=days_ago(1)) as dag:
    cli_command = BashOperator(
        task_id="bash_command",
        bash_command="cp -R /usr/local/airflow/dags/dbt /tmp;\
cd /tmp/dbt/dbt-starter-project;\
/usr/local/airflow/.local/bin/dbt  run --project-dir /tmp/dbt/dbt-starter-project/ --profiles-dir ..;\
cat /tmp/dbt_logs/dbt.log"
    )
```