# Custom plugin to change the DAG schedule timezone<a name="samples-plugins-timezone"></a>

 Apache Airflow schedules your DAGs in UTC\+0 by default\. However, Amazon MWAA supports overriding this default and modifying the timezone in which your DAGs are scheduled to run\. The following sample shows how you can set up a custom plugin for your Amazon MWAA environment to modify the timezone in which your DAGs are scheduled\. 

**Topics**
+ [Version](#samples-plugins-timezone-version)
+ [Prerequisites](#samples-plugins-timezone-prerequisites)
+ [Permissions](#samples-plugins-timezone-permissions)
+ [Custom plugin](#samples-plugins-timezone-custom-plugin)
+ [Plugins\.zip](#samples-plugins-timezone-plugins-zip)
+ [What's next?](#samples-plugins-timezone-plugins-next-up)

## Version<a name="samples-plugins-timezone-version"></a>
+ The sample code on this page can be used with **Apache Airflow v1\.10\.12** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.
+ The sample code on this page can be used with **Apache Airflow v2\.0\.2** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="samples-plugins-timezone-prerequisites"></a>

To use the sample code on this page, you'll need the following:
+ An [Amazon MWAA environment](get-started.md)\.

## Permissions<a name="samples-plugins-timezone-permissions"></a>
+ No additional permissions are required to use the sample code on this page\.

## Custom plugin<a name="samples-plugins-timezone-custom-plugin"></a>

Apache Airflow will run the Python files in the `plugins` directory at start\-up\. The following custom plugin will override and modify the default time zone for your scheduled DAGs\.

1. Create a directory names `plugins` for your custom plugin, and navigate to the directory\. For example:

   ```
   $ mkdir plugins
   $ cd plugins
   ```

1. Copy the contents of the following code sample and save locally as `dag-timezone-plugin.py` in the `plugins` folder\.

   ```
   import time
   import os
   
   os.environ['TZ'] = 'America/Los_Angeles'
   time.tzset()
   ```

1.  In the `plugins` directory, create an empty Python file named `__init__.py`\. Your `plugins` directory should be similar to the following: 

   ```
   plugins/
   |-- __init__.py
   |-- dag-timezone-plugin.py
   ```

## Plugins\.zip<a name="samples-plugins-timezone-plugins-zip"></a>

The following steps show how to create `plugins.zip`\. The content of this example can be combined with other plugins and binaries into a single `plugins.zip` file\.

1. In your command prompt, navigate to the `plugins` directory from the previous step\. For example:

   ```
   cd plugins
   ```

1. Zip the contents within your `plugins` directory\.

   ```
   zip -r ../plugins.zip ./
   ```

## What's next?<a name="samples-plugins-timezone-plugins-next-up"></a>
+ Learn more about how to upload the `plugins.zip` file in this example to your Amazon S3 bucket in [Installing custom plugins](configuring-dag-import-plugins.md)\.