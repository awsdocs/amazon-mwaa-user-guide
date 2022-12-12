# Creating a custom plugin for Apache Airflow PythonVirtualenvOperator<a name="samples-virtualenv"></a>

The following sample shows how to patch the Apache Airflow PythonVirtualenvOperator with a custom plugin on Amazon Managed Workflows for Apache Airflow \(MWAA\)\.

**Topics**
+ [Version](#samples-virtualenv-version)
+ [Prerequisites](#samples-virtualenv-prereqs)
+ [Permissions](#samples-virtualenv-permissions)
+ [Requirements](#samples-virtualenv-dependencies)
+ [Custom plugin sample code](#samples-virtualenv-plugins-code)
+ [Plugins\.zip](#samples-virtualenv-pluginszip)
+ [Code sample](#samples-virtualenv-code)
+ [Airflow configuration options](#samples-virtualenv-airflow-config)
+ [What's next?](#samples-virtualenv-next-up)

## Version<a name="samples-virtualenv-version"></a>
+ The sample code on this page can be used with **Apache Airflow v1** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.
+ You can use the code example on this page with **Apache Airflow v2 and above** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="samples-virtualenv-prereqs"></a>

To use the sample code on this page, you'll need the following:
+ An [Amazon MWAA environment](get-started.md)\.

## Permissions<a name="samples-virtualenv-permissions"></a>
+ No additional permissions are required to use the code example on this page\.

## Requirements<a name="samples-virtualenv-dependencies"></a>

To use the sample code on this page, add the following dependencies to your `requirements.txt`\. To learn more, see [Installing Python dependencies](working-dags-dependencies.md)\.

```
virtualenv
```

## Custom plugin sample code<a name="samples-virtualenv-plugins-code"></a>

Apache Airflow will execute the contents of Python files in the plugins folder at startup\. This plugin will patch the built\-in `PythonVirtualenvOperator` during that startup process to make it compatible with Amazon MWAA\. The following steps show the sample code for the custom plugin\.

------
#### [ Apache Airflow v2 ]

1. In your command prompt, navigate to the `plugins` directory above\. For example:

   ```
   cd plugins
   ```

1. Copy the contents of the following code sample and save locally as `virtual_python_plugin.py`\.

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
   from airflow.plugins_manager import AirflowPlugin
   import airflow.utils.python_virtualenv 
   from typing import List
   
   def _generate_virtualenv_cmd(tmp_dir: str, python_bin: str, system_site_packages: bool) -> List[str]:
       cmd = ['python3','/usr/local/airflow/.local/lib/python3.7/site-packages/virtualenv', tmp_dir]
       if system_site_packages:
           cmd.append('--system-site-packages')
       if python_bin is not None:
           cmd.append(f'--python={python_bin}')
       return cmd
   
   airflow.utils.python_virtualenv._generate_virtualenv_cmd=_generate_virtualenv_cmd
   
   class VirtualPythonPlugin(AirflowPlugin):                
       name = 'virtual_python_plugin'
   ```

------
#### [ Apache Airflow v1 ]

1. In your command prompt, navigate to the `plugins` directory above\. For example:

   ```
   cd plugins
   ```

1. Copy the contents of the following code sample and save locally as `virtual_python_plugin.py`\.

   ```
   from airflow.plugins_manager import AirflowPlugin
   from airflow.operators.python_operator import PythonVirtualenvOperator
   
   def _generate_virtualenv_cmd(self, tmp_dir):
       cmd = ['python3','/usr/local/airflow/.local/lib/python3.7/site-packages/virtualenv', tmp_dir]
       if self.system_site_packages:
           cmd.append('--system-site-packages')
       if self.python_version is not None:
           cmd.append('--python=python{}'.format(self.python_version))
       return cmd
   PythonVirtualenvOperator._generate_virtualenv_cmd=_generate_virtualenv_cmd
   
   class EnvVarPlugin(AirflowPlugin):                
       name = 'virtual_python_plugin'
   ```

------

## Plugins\.zip<a name="samples-virtualenv-pluginszip"></a>

The following steps show how to create the `plugins.zip`\.

1. In your command prompt, navigate to the directory containing `virtual_python_plugin.py` above\. For example:

   ```
   cd plugins
   ```

1. Zip the contents within your `plugins` folder\.

   ```
   zip plugins.zip virtual_python_plugin.py
   ```

## Code sample<a name="samples-virtualenv-code"></a>

The following steps describe how to create the DAG code for the custom plugin\.

------
#### [ Apache Airflow v2 ]

1. In your command prompt, navigate to the directory where your DAG code is stored\. For example:

   ```
   cd dags
   ```

1. Copy the contents of the following code sample and save locally as `virtualenv_test.py`\.

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
   from airflow.operators.python import PythonVirtualenvOperator
   from airflow.utils.dates import days_ago
   import os
   
   os.environ["PATH"] = os.getenv("PATH") + ":/usr/local/airflow/.local/bin"
   
   def virtualenv_fn():
       import boto3
       print("boto3 version ",boto3.__version__)
   
   with DAG(dag_id="virtualenv_test", schedule_interval=None, catchup=False, start_date=days_ago(1)) as dag:
       virtualenv_task = PythonVirtualenvOperator(
           task_id="virtualenv_task",
           python_callable=virtualenv_fn,
           requirements=["boto3>=1.17.43"],
           system_site_packages=False,
           dag=dag,
       )
   ```

------
#### [ Apache Airflow v1 ]

1. In your command prompt, navigate to the directory where your DAG code is stored\. For example:

   ```
   cd dags
   ```

1. Copy the contents of the following code sample and save locally as `virtualenv_test.py`\.

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
   from airflow.operators.python_operator import PythonVirtualenvOperator
   from airflow.utils.dates import days_ago
   import os
   
   os.environ["PATH"] = os.getenv("PATH") + ":/usr/local/airflow/.local/bin"
   
   def virtualenv_fn():
       import boto3
       print("boto3 version ",boto3.__version__)
   
   with DAG(dag_id="virtualenv_test", schedule_interval=None, catchup=False, start_date=days_ago(1)) as dag:
       virtualenv_task = PythonVirtualenvOperator(
           task_id="virtualenv_task",
           python_callable=virtualenv_fn,
           requirements=["boto3>=1.17.43"],
           system_site_packages=False,
           dag=dag,
       )
   ```

------

## Airflow configuration options<a name="samples-virtualenv-airflow-config"></a>

If you're using Apache Airflow v2, add `core.lazy_load_plugins : False` as an Apache Airflow configuration option\. To learn more, see [Using configuration options to load plugins in 2](configuring-env-variables.md#configuring-2.0-airflow-override)\.

## What's next?<a name="samples-virtualenv-next-up"></a>
+ Learn how to upload the `requirements.txt` file in this example to your Amazon S3 bucket in [Installing Python dependencies](working-dags-dependencies.md)\.
+ Learn how to upload the DAG code in this example to the `dags` folder in your Amazon S3 bucket in [Adding or updating DAGs](configuring-dag-folder.md)\.
+ Learn more about how to upload the `plugins.zip` file in this example to your Amazon S3 bucket in [Installing custom plugins](configuring-dag-import-plugins.md)\.