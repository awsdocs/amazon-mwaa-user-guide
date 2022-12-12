# Creating a custom plugin with Apache Hive and Hadoop<a name="samples-hive"></a>

Amazon MWAA extracts the contents of a `plugins.zip` to `/usr/local/airflow/plugins`\. This can be used to add binaries to your containers\. In addition, Apache Airflow executes the contents of Python files in the `plugins` folder at *startup*—enabling you to set and modify environment variables\. The following sample walks you through the steps to create a custom plugin using Apache Hive and Hadoop on an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment and can be combined with other custom plugins and binaries\.

**Topics**
+ [Version](#samples-hive-version)
+ [Prerequisites](#samples-hive-prereqs)
+ [Permissions](#samples-hive-permissions)
+ [Requirements](#samples-hive-dependencies)
+ [Download dependencies](#samples-hive-install)
+ [Custom plugin](#samples-hive-plugins-code)
+ [Plugins\.zip](#samples-hive-pluginszip)
+ [Code sample](#samples-hive-code)
+ [Airflow configuration options](#samples-hive-airflow-config)
+ [What's next?](#samples-hive-next-up)

## Version<a name="samples-hive-version"></a>
+ The sample code on this page can be used with **Apache Airflow v1** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.
+ You can use the code example on this page with **Apache Airflow v2 and above** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="samples-hive-prereqs"></a>

To use the sample code on this page, you'll need the following:
+ An [Amazon MWAA environment](get-started.md)\.

## Permissions<a name="samples-hive-permissions"></a>
+ No additional permissions are required to use the code example on this page\.

## Requirements<a name="samples-hive-dependencies"></a>

To use the sample code on this page, add the following dependencies to your `requirements.txt`\. To learn more, see [Installing Python dependencies](working-dags-dependencies.md)\.

------
#### [ Apache Airflow v2 ]

```
-c https://raw.githubusercontent.com/apache/airflow/constraints-2.0.2/constraints-3.7.txt
apache-airflow-providers-amazon[apache.hive]
```

------
#### [ Apache Airflow v1 ]

```
apache-airflow[hive]==1.10.12
```

------

## Download dependencies<a name="samples-hive-install"></a>

Amazon MWAA will extract the contents of plugins\.zip into `/usr/local/airflow/plugins` on each Amazon MWAA scheduler and worker container\. This is used to add binaries to your environment\. The following steps describe how to assemble the files needed for the custom plugin\.

1. In your command prompt, navigate to the directory where you would like to create your plugin\. For example:

   ```
   cd plugins
   ```

1. Download [Hadoop](https://hadoop.apache.org/) from a [mirror](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.3.0/hadoop-3.3.0.tar.gz), for example:

   ```
   wget https://downloads.apache.org/hadoop/common/hadoop-3.3.0/hadoop-3.3.0.tar.gz
   ```

1. Download [Hive](https://hive.apache.org/) from a [mirror](https://www.apache.org/dyn/closer.cgi/hive/), for example:

   ```
   wget https://downloads.apache.org/hive/hive-3.1.2/apache-hive-3.1.2-bin.tar.gz
   ```

1. Create a directory\. For example:

   ```
   mkdir hive_plugin
   ```

1. Extract Hadoop\.

   ```
   tar -xvzf hadoop-3.3.0.tar.gz -C hive_plugin
   ```

1. Extract Hive\.

   ```
   tar -xvzf apache-hive-3.1.2-bin.tar.gz -C hive_plugin
   ```

## Custom plugin<a name="samples-hive-plugins-code"></a>

Apache Airflow will execute the contents of Python files in the plugins folder at startup\. This is used to set and modify environment variables\. The following steps describe the sample code for the custom plugin\.

1. In your command prompt, navigate to the `hive_plugin` directory\. For example:

   ```
   cd hive_plugin
   ```

1. Copy the contents of the following code sample and save locally as `hive_plugin.py` in the `hive_plugin` directory\.

   ```
   from airflow.plugins_manager import AirflowPlugin
   import os
   os.environ["JAVA_HOME"]="/usr/lib/jvm/jre"
   os.environ["HADOOP_HOME"]='/usr/local/airflow/plugins/hadoop-3.3.0'
   os.environ["HADOOP_CONF_DIR"]='/usr/local/airflow/plugins/hadoop-3.3.0/etc/hadoop'
   os.environ["HIVE_HOME"]='/usr/local/airflow/plugins/apache-hive-3.1.2-bin'
   os.environ["PATH"] = os.getenv("PATH") + ":/usr/local/airflow/plugins/hadoop-3.3.0:/usr/local/airflow/plugins/apache-hive-3.1.2-bin/bin:/usr/local/airflow/plugins/apache-hive-3.1.2-bin/lib" 
   os.environ["CLASSPATH"] = os.getenv("CLASSPATH") + ":/usr/local/airflow/plugins/apache-hive-3.1.2-bin/lib" 
   class EnvVarPlugin(AirflowPlugin):                
       name = 'hive_plugin'
   ```

1.  Cope the content of the following text and save locally as `.airflowignore` in the `hive_plugin` directory\. 

   ```
   hadoop-3.3.0
   apache-hive-3.1.2-bin
   ```

## Plugins\.zip<a name="samples-hive-pluginszip"></a>

The following steps show how to create `plugins.zip`\. The contents of this example can be combined with other plugins and binaries into a single `plugins.zip` file\.

1. In your command prompt, navigate to the `hive_plugin` directory from the previous step\. For example:

   ```
   cd hive_plugin
   ```

1. Zip the contents within your `plugins` folder\.

   ```
   zip -r ../hive_plugin.zip ./
   ```

## Code sample<a name="samples-hive-code"></a>

The following steps describe how to create the DAG code that will test the custom plugin\.

1. In your command prompt, navigate to the directory where your DAG code is stored\. For example:

   ```
   cd dags
   ```

1. Copy the contents of the following code sample and save locally as `hive.py`\.

   ```
   from airflow import DAG
   from airflow.operators.bash_operator import BashOperator
   from airflow.utils.dates import days_ago
   
   with DAG(dag_id="hive_test_dag", schedule_interval=None, catchup=False, start_date=days_ago(1)) as dag:
       hive_test = BashOperator(
           task_id="hive_test",
           bash_command='hive --help'
       )
   ```

## Airflow configuration options<a name="samples-hive-airflow-config"></a>

If you're using Apache Airflow v2, add `core.lazy_load_plugins : False` as an Apache Airflow configuration option\. To learn more, see [Using configuration options to load plugins in 2](configuring-env-variables.md#configuring-2.0-airflow-override)\.

## What's next?<a name="samples-hive-next-up"></a>
+ Learn how to upload the `requirements.txt` file in this example to your Amazon S3 bucket in [Installing Python dependencies](working-dags-dependencies.md)\.
+ Learn how to upload the DAG code in this example to the `dags` folder in your Amazon S3 bucket in [Adding or updating DAGs](configuring-dag-folder.md)\.
+ Learn more about how to upload the `plugins.zip` file in this example to your Amazon S3 bucket in [Installing custom plugins](configuring-dag-import-plugins.md)\.