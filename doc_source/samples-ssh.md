# Creating an SSH connection using the `SSHOperator`<a name="samples-ssh"></a>

The following example describes how you can use the `SSHOperator` in a directed acyclic graph \(DAG\) to connect to a remote Amazon EC2 instance from your Amazon Managed Workflows for Apache Airflow \(MWAA\) environment\. You can use a similar approach to connect to any remote instance with SSH access\. 

 In the following example, you upload a SSH secret key \(`.pem`\) to your environment's `dags` directory on Amazon S3\. Then, you install the necessary dependencies using `requirements.txt` and create a new Apache Airflow connection in the UI\. Finally, you write a DAG that creates an SSH connection to the remote instance\. 

**Topics**
+ [Version](#samples-ssh-version)
+ [Prerequisites](#samples-ssh-prereqs)
+ [Permissions](#samples-ssh-permissions)
+ [Requirements](#samples-ssh-dependencies)
+ [Copy your secret key to Amazon S3](#samples-ssh-secret)
+ [Create a new Apache Airflow connection](#samples-ssh-connection)
+ [Code sample](#samples-ssh-code)

## Version<a name="samples-ssh-version"></a>
+ You can use the code example on this page with **Apache Airflow v2 and above** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="samples-ssh-prereqs"></a>

To use the sample code on this page, you'll need the following:
+ An [Amazon MWAA environment](get-started.md)\.
+ An SSH secret key\. The code sample assumes you have an Amazon EC2 instance and a `.pem` in the same Region as your Amazon MWAA environment\. If you don't have a key, see [Create or import a key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#prepare-key-pair) in the *Amazon EC2 User Guide for Linux Instances*\.

## Permissions<a name="samples-ssh-permissions"></a>
+ No additional permissions are required to use the code example on this page\.

## Requirements<a name="samples-ssh-dependencies"></a>

Add the following parameter to `requirements.txt` to install the `apache-airflow-providers-ssh` package on your web server\. Once your environment updates and Amazon MWAA successfully installs the dependency, you'll be able to see a new **SSH** connection type in the Apache Airflow UI\. 

```
-c https://raw.githubusercontent.com/apache/airflow/constraints-2.2.2/constraints-3.7.txt
apache-airflow-providers-ssh
```

**Note**  
 `-c` defines the constraints URL in `requirements.txt`\. This ensures that Amazon MWAA installs the correct package version for your environemnt\. 

## Copy your secret key to Amazon S3<a name="samples-ssh-secret"></a>

Use the following AWS Command Line Interface command to copy your `.pem` key to your environment's `dags` directory in Amazon S3\.

```
$ aws s3 cp your-secret-key.pem s3://your-bucket/dags/
```

 Amazon MWAA copies the content in `dags`, including the `.pem` key, to the local `/usr/local/airflow/dags/` directory, By doing this, Apache Airflow can access the key\. 

## Create a new Apache Airflow connection<a name="samples-ssh-connection"></a>

**To create a new SSH connection using the Apache Airflow UI**

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. From the list of environments, choose **Open Airflow UI** for your environment\. 

1. On the Apache Airflow UI page, choose **Admin** from the top navigation bar to expand the dropdown list, then choose **Connections**\. 

1. On the **List Connections** page, choose **\+**, or **Add a new record** button to add a new connection\. 

1. On the **Add Connection** page, add the following information: 

   1. For **Connection Id**, enter **ssh\_new**\.

   1. For **Connection Type**, choose **SSH** from the dropdown list\.
**Note**  
 If the **SSH** connection type is not available in the list, Amazon MWAA hasn't installed the required `apache-airflow-providers-ssh` package\. Update your `requirements.txt` file to include this package, then try again\. 

   1. For **Host**, enter the IP address for the Amazon EC2 instance that you want to connect to\. For example, **12\.345\.67\.89**\.

   1. For **Username**, enter **ec2\-user** if you are connecting to an Amazon EC2 instance\. Your username might be different, depending on the type of remote instance you want Apache Airflow to connect to\.

   1. For **Extra**, enter the following key\-value pair in JSON format:

      ```
      { "key_file": "/usr/local/airflow/dags/your-secret-key.pem" }
      ```

      This key\-value pair instructs Apache Airflow to look for the secret key in the local `/dags` directory\.

## Code sample<a name="samples-ssh-code"></a>

The following DAG uses the `SSHOperator` to connect to your target Amazon EC2 instance, then runs the `hostname` Linux command to print the name of the instnace\. You can modify the DAG to run any command or script on the remote instance\. 

1. Open a terminal, and navigate to the directory where your DAG code is stored\. For example:

   ```
   cd dags
   ```

1. Copy the contents of the following code sample and save locally as `ssh.py`\.

   ```
   from airflow.decorators import dag
   from datetime import datetime
   from airflow.providers.ssh.operators.ssh import SSHOperator
   
   @dag(
       dag_id="ssh_operator_example",
       schedule_interval=None,     
       start_date=datetime(2022, 1, 1),
       catchup=False,
       )
   def ssh_dag():
       task_1=SSHOperator(
           task_id="ssh_task",
           ssh_conn_id='ssh_new',
           command='hostname',
       )
   
   my_ssh_dag = ssh_dag()
   ```

1.  Run the following AWS CLI command to copy the DAG to your environment's bucket, then trigger the DAG using the Apache Airflow UI\. 

   ```
   $ aws s3 cp your-dag.py s3://your-environment-bucket/dags/
   ```

1. If successful, you'll see output similar to the following in the task logs for `ssh_task` in the `ssh_operator_example` DAG: 

   ```
   [2022-01-01, 12:00:00 UTC] {{base.py:79}} INFO - Using connection to: id: ssh_new. Host: 12.345.67.89, Port: None,
   Schema: , Login: ec2-user, Password: None, extra: {'key_file': '/usr/local/airflow/dags/your-secret-key.pem'}
   [2022-01-01, 12:00:00 UTC] {{ssh.py:264}} WARNING - Remote Identification Change is not verified. This won't protect against Man-In-The-Middle attacks
   [2022-01-01, 12:00:00 UTC] {{ssh.py:270}} WARNING - No Host Key Verification. This won't protect against Man-In-The-Middle attacks
   [2022-01-01, 12:00:00 UTC] {{transport.py:1819}} INFO - Connected (version 2.0, client OpenSSH_7.4)
   [2022-01-01, 12:00:00 UTC] {{transport.py:1819}} INFO - Authentication (publickey) successful!
   [2022-01-01, 12:00:00 UTC] {{ssh.py:139}} INFO - Running command: hostname
   [2022-01-01, 12:00:00 UTC]{{ssh.py:171}} INFO - ip-123-45-67-89.us-west-2.compute.internal
   [2022-01-01, 12:00:00 UTC] {{taskinstance.py:1280}} INFO - Marking task as SUCCESS. dag_id=ssh_operator_example, task_id=ssh_task, execution_date=20220712T200914, start_date=20220712T200915, end_date=20220712T200916
   ```