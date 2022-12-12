# Creating a custom plugin that generates runtime environment variables<a name="samples-env-variables"></a>

The following sample walks you through the steps to create a custom plugin that generates environment variables at runtime on an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment\.

**Topics**
+ [Version](#samples-env-variables-version)
+ [Prerequisites](#samples-env-variables-prereqs)
+ [Permissions](#samples-env-variables-permissions)
+ [Requirements](#samples-env-variables-dependencies)
+ [Custom plugin](#samples-env-variables-plugins-code)
+ [Plugins\.zip](#samples-env-variables-pluginszip)
+ [Airflow configuration options](#samples-env-variables-airflow-config)
+ [What's next?](#samples-env-variables-next-up)

## Version<a name="samples-env-variables-version"></a>
+ The sample code on this page can be used with **Apache Airflow v1** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="samples-env-variables-prereqs"></a>

To use the sample code on this page, you'll need the following:
+ An [Amazon MWAA environment](get-started.md)\.

## Permissions<a name="samples-env-variables-permissions"></a>
+ No additional permissions are required to use the code example on this page\.

## Requirements<a name="samples-env-variables-dependencies"></a>
+ To use this code example with Apache Airflow v1, no additional dependencies are required\. The code uses the [Apache Airflow v1 base install](https://raw.githubusercontent.com/apache/airflow/constraints-1.10.12/constraints-3.7.txt) on your environment\.

## Custom plugin<a name="samples-env-variables-plugins-code"></a>

Apache Airflow will execute the contents of Python files in the plugins folder at startup\. This is used to set and modify environment variables\. The following steps describe the sample code for the custom plugin\.

1. In your command prompt, navigate to the directory where your plugins are stored\. For example:

   ```
   cd plugins
   ```

1. Copy the contents of the following code sample and save locally as `env_var_plugin.py` in the above folder\.

   ```
   from airflow.plugins_manager import AirflowPlugin
   import os
   
   os.environ["PATH"] = os.getenv("PATH") + ":/usr/local/airflow/.local/lib/python3.7/site-packages" 
   os.environ["JAVA_HOME"]="/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.272.b10-1.amzn2.0.1.x86_64"
   
   class EnvVarPlugin(AirflowPlugin):                
        name = 'env_var_plugin'
   ```

## Plugins\.zip<a name="samples-env-variables-pluginszip"></a>

The following steps show how to create `plugins.zip`\. The contents of this example can be combined with other plugins and binaries into a single `plugins.zip` file\.

1. In your command prompt, navigate to the `hive_plugin` directory from the previous step\. For example:

   ```
   cd plugins
   ```

1. Zip the contents within your `plugins` folder\.

   ```
   zip -r ../plugins.zip ./
   ```

## Airflow configuration options<a name="samples-env-variables-airflow-config"></a>

If you're using Apache Airflow v2, add `core.lazy_load_plugins : False` as an Apache Airflow configuration option\. To learn more, see [Using configuration options to load plugins in 2](configuring-env-variables.md#configuring-2.0-airflow-override)\.

## What's next?<a name="samples-env-variables-next-up"></a>
+ Learn how to upload the `requirements.txt` file in this example to your Amazon S3 bucket in [Installing Python dependencies](working-dags-dependencies.md)\.
+ Learn how to upload the DAG code in this example to the `dags` folder in your Amazon S3 bucket in [Adding or updating DAGs](configuring-dag-folder.md)\.
+ Learn more about how to upload the `plugins.zip` file in this example to your Amazon S3 bucket in [Installing custom plugins](configuring-dag-import-plugins.md)\.