# Creating an SSH connection using the SSHOperator<a name="samples-ssh"></a>

The following sample walks you through the steps on how to create a DAG that creates an SSH connection using a private key in AWS Secrets Manager on Amazon Managed Workflows for Apache Airflow \(MWAA\)\.

**Topics**
+ [Version](#samples-ssh-version)
+ [Prerequisites](#samples-ssh-prereqs)
+ [Permissions](#samples-ssh-permissions)
+ [Requirements](#samples-ssh-dependencies)
+ [Create a secret key](#samples-ssh-secret)
+ [Create the Airflow connection](#samples-ssh-connection)
+ [Code sample](#samples-ssh-code)
+ [What's next?](#samples-ssh-next-up)

## Version<a name="samples-ssh-version"></a>
+ The sample code on this page can be used with **Apache Airflow v1\.10\.12** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="samples-ssh-prereqs"></a>

To use the sample code on this page, you'll need the following:
+ An [Amazon MWAA environment](get-started.md)\.
+ Create an SSH key\. You need to create an Amazon EC2 SSH key \(**\.pem**\) in the same Region as your Amazon MWAA environment\. If you don't have an SSH key, see [Create or import a key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#prepare-key-pair) in the *Amazon EC2 User Guide for Linux Instances*\.

## Permissions<a name="samples-ssh-permissions"></a>
+ No additional permissions are required to use the sample code on this page\.

## Requirements<a name="samples-ssh-dependencies"></a>
+ Create a requirements\.txt with the *awscli* package, which invokes the command to fetch the secret value using a bash command\.

  ```
  awscli==1.19.5
  apache-airflow-backport-providers-ssh
  ```

## Create a secret key<a name="samples-ssh-secret"></a>
+ Create a secret in Secrets Manager to store the base64\-encoded private key\.

  ```
  aws secretsmanager create-secret --name your-key-name --secret-string 'base64 your-key-name.pem'
  ```

## Create the Airflow connection<a name="samples-ssh-connection"></a>
+ In the Apache Airflow UI for Apache Airflow v1\.10\.12, open *Admin* > *Connections*, and add the following connection\.

  1. Conn Id : ssh\_new

  1. Conn Type : SSH

  1. Host: *Your IP Address*

  1. Username: ec2\-user

  1. Extra: \{"key\_file":"/tmp/*your\-key\-name*\.pem"\}

## Code sample<a name="samples-ssh-code"></a>

The following DAG fetches the private key value from Secrets Manager, decodes and saves locally as */tmp/*your\-key\-name*\.pem*\. The SSHOperator creates an *\.sh* script in Amazon S3 and copies it to your local machine, then invokes it\.

1. In your command prompt, navigate to the directory where your DAG code is stored\. For example:

   ```
   cd dags
   ```

1. Copy the contents of the following code sample and save locally as `ssh.py`\.

   ```
   import airflow
   from airflow.models import DAG
   from airflow.operators.bash_operator import BashOperator
   from airflow.utils.dates import days_ago
   from datetime import datetime, timedelta
   from airflow.contrib.operators.ssh_operator import SSHOperator
   
   args = {
       "owner": "airflow",
       "provide_context": True,
   }
   with DAG(dag_id="remote_server_direct_key02", schedule_interval='@daily', start_date=days_ago(2), default_args=args, catchup=False) as dag:
       task1 = BashOperator(
           task_id="pem_script00",
           bash_command="ls "
           bash_command="/usr/local/airflow/.local/bin/aws secretsmanager get-secret-value --secret-id your-key-name --query 'SecretString' --output text |base64 -d > /tmp/your-key-name.pem"
      )
   
       task2 = SSHOperator(
           task_id="ssh_script00",
           ssh_conn_id='ssh_new',
           command='aws s3 cp s3://<bucket-name>/bashtest/scripts/ssh_connection.sh . && cat ssh_connection.sh | bash - ',
      )
   
   task1 >> task2
   ```

## What's next?<a name="samples-ssh-next-up"></a>
+ Learn how to upload the DAG code in this example to the `dags` folder in your Amazon S3 bucket in [Adding or updating DAGs](configuring-dag-folder.md)\.