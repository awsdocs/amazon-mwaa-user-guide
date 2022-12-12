# Creating a custom plugin with Oracle<a name="samples-oracle"></a>

The following sample walks you through the steps to create a custom plugin using Oracle for Amazon MWAA and can be combined with other custom plugins and binaries in your plugins\.zip file\.

**Contents**
+ [Version](#samples-oracle-version)
+ [Prerequisites](#samples-oracle-prereqs)
+ [Permissions](#samples-oracle-permissions)
+ [Requirements](#samples-oracle-dependencies)
+ [Code sample](#samples-oracle-code)
+ [Create the custom plugin](#samples-oracle-create-pluginszip-steps)
  + [Download dependencies](#samples-oracle-install)
  + [Custom plugin](#samples-oracle-plugins-code)
  + [Plugins\.zip](#samples-oracle-pluginszip)
+ [Airflow configuration options](#samples-oracle-airflow-config)
+ [What's next?](#samples-oracle-next-up)

## Version<a name="samples-oracle-version"></a>
+ The sample code on this page can be used with **Apache Airflow v1** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.
+ You can use the code example on this page with **Apache Airflow v2 and above** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="samples-oracle-prereqs"></a>

To use the sample code on this page, you'll need the following:
+ An [Amazon MWAA environment](get-started.md)\.
+  *Worker* logging enabled at any log level, `CRITICAL` or above, for your environment\. For more information about Amazon MWAA log types and how to manage your log groups, see [Viewing Airflow logs in Amazon CloudWatch](monitoring-airflow.md) 

## Permissions<a name="samples-oracle-permissions"></a>
+ No additional permissions are required to use the code example on this page\.

## Requirements<a name="samples-oracle-dependencies"></a>

To use the sample code on this page, add the following dependencies to your `requirements.txt`\. To learn more, see [Installing Python dependencies](working-dags-dependencies.md)\.

------
#### [ Apache Airflow v2 ]

```
-c https://raw.githubusercontent.com/apache/airflow/constraints-2.0.2/constraints-3.7.txt
cx_Oracle
apache-airflow-providers-oracle
```

------
#### [ Apache Airflow v1 ]

```
cx_Oracle==8.1.0
apache-airflow[oracle]==1.10.12
```

------

## Code sample<a name="samples-oracle-code"></a>

The following steps describe how to create the DAG code that will test the custom plugin\.

1. In your command prompt, navigate to the directory where your DAG code is stored\. For example:

   ```
   cd dags
   ```

1. Copy the contents of the following code sample and save locally as `oracle.py`\. 

   ```
   from airflow import DAG
   from airflow.operators.python_operator import PythonOperator
   from airflow.utils.dates import days_ago
   import os
   import cx_Oracle
   
   DAG_ID = os.path.basename(__file__).replace(".py", "")
   
   def testHook(**kwargs):
       cx_Oracle.init_oracle_client()
       version = cx_Oracle.clientversion()
       print("cx_Oracle.clientversion",version)
       return version
   
   with DAG(dag_id=DAG_ID, schedule_interval=None, catchup=False, start_date=days_ago(1)) as dag:
       hook_test = PythonOperator(
           task_id="hook_test",
           python_callable=testHook,
           provide_context=True 
       )
   ```

## Create the custom plugin<a name="samples-oracle-create-pluginszip-steps"></a>

This section describes how to download the dependencies, create the custom plugin and the plugins\.zip\.

### Download dependencies<a name="samples-oracle-install"></a>

Amazon MWAA will extract the contents of plugins\.zip into `/usr/local/airflow/plugins` on each Amazon MWAA scheduler and worker container\. This is used to add binaries to your environment\. The following steps describe how to assemble the files needed for the custom plugin\.

**Pull the Amazon Linux container image**

1. In your command prompt, pull the Amazon Linux container image, and run the container locally\. For example:

   ```
   docker pull amazonlinux
   docker run -it amazonlinux:latest /bin/bash
   ```

   Your command prompt should invoke a bash command line\. For example:

   ```
   bash-4.2#
   ```

1. Install the Linux\-native asynchronous I/O facility \(libaio\)\.

   ```
   yum -y install libaio
   ```

1. Keep this window open for subsequent steps\. We'll be copying the following files locally: `lib64/libaio.so.1`, `lib64/libaio.so.1.0.0`, `lib64/libaio.so.1.0.1`\.

**Download client folder**

1. Install the unzip package locally\. For example:

   ```
   sudo yum install unzip
   ```

1. Create an `oracle_plugin` directory\. For example:

   ```
   mkdir oracle_plugin
   cd oracle_plugin
   ```

1. Use the following curl command to download the [instantclient\-basic\-linux\.x64\-18\.5\.0\.0\.0dbru\.zip](https://download.oracle.com/otn_software/linux/instantclient/185000/instantclient-basic-linux.x64-18.5.0.0.0dbru.zip) from [Oracle Instant Client Downloads for Linux x86\-64 \(64\-bit\)](https://www.oracle.com/database/technologies/instant-client/linux-x86-64-downloads.html)\.

   ```
   curl https://download.oracle.com/otn_software/linux/instantclient/185000/instantclient-basic-linux.x64-18.5.0.0.0dbru.zip > client.zip
   ```

1. Unzip the `client.zip` file\. For example:

   ```
   unzip *.zip
   ```

**Extract files from Docker**

1. In a new command prompt, display and write down your Docker container ID\. For example:

   ```
   docker container ls
   ```

   Your command prompt should return all containers and their IDs\. For example:

   ```
   debc16fd6970
   ```

1. In your `oracle_plugin` directory, extract the `lib64/libaio.so.1`, `lib64/libaio.so.1.0.0`, `lib64/libaio.so.1.0.1` files to the local `instantclient_18_5` folder\. For example:

   ```
   docker cp debc16fd6970:/lib64/libaio.so.1 instantclient_18_5/
   docker cp debc16fd6970:/lib64/libaio.so.1.0.0 instantclient_18_5/
   docker cp debc16fd6970:/lib64/libaio.so.1.0.1 instantclient_18_5/
   ```

### Custom plugin<a name="samples-oracle-plugins-code"></a>

Apache Airflow will execute the contents of Python files in the plugins folder at startup\. This is used to set and modify environment variables\. The following steps describe the sample code for the custom plugin\.
+ Copy the contents of the following code sample and save locally as `env_var_plugin_oracle.py`\.

  ```
  from airflow.plugins_manager import AirflowPlugin
  import os
  
  os.environ["LD_LIBRARY_PATH"]='/usr/local/airflow/plugins/instantclient_18_5'
  os.environ["DPI_DEBUG_LEVEL"]="64"
  
  class EnvVarPlugin(AirflowPlugin):                
      name = 'env_var_plugin'
  ```

### Plugins\.zip<a name="samples-oracle-pluginszip"></a>

The following steps show how to create the `plugins.zip`\. The contents of this example can be combined with your other plugins and binaries into a single `plugins.zip` file\.

**Zip the contents of the plugin directory**

1. In your command prompt, navigate to the `oracle_plugin` directory\. For example:

   ```
   cd oracle_plugin
   ```

1. Zip the `instantclient_18_5` directory in plugins\.zip\. For example:

   ```
   zip -r ../plugins.zip ./
   ```

1. You should see the following in your command prompt:

   ```
   oracle_plugin$ ls
   client.zip		instantclient_18_5
   ```

1. Remove the `client.zip` file\. For example:

   ```
   rm client.zip
   ```

**Zip the env\_var\_plugin\_oracle\.py file**

1. Add the `env_var_plugin_oracle.py` file to the root of the plugins\.zip\. For example:

   ```
   zip plugins.zip env_var_plugin_oracle.py
   ```

1. Your plugins\.zip should now include the following:

   ```
   env_var_plugin_oracle.py
   instantclient_18_5/
   ```

## Airflow configuration options<a name="samples-oracle-airflow-config"></a>

If you're using Apache Airflow v2, add `core.lazy_load_plugins : False` as an Apache Airflow configuration option\. To learn more, see [Using configuration options to load plugins in 2](configuring-env-variables.md#configuring-2.0-airflow-override)\.

## What's next?<a name="samples-oracle-next-up"></a>
+ Learn how to upload the `requirements.txt` file in this example to your Amazon S3 bucket in [Installing Python dependencies](working-dags-dependencies.md)\.
+ Learn how to upload the DAG code in this example to the `dags` folder in your Amazon S3 bucket in [Adding or updating DAGs](configuring-dag-folder.md)\.
+ Learn more about how to upload the `plugins.zip` file in this example to your Amazon S3 bucket in [Installing custom plugins](configuring-dag-import-plugins.md)\.