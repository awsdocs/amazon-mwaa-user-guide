# Refreshing a CodeArtifact token<a name="samples-code-artifact"></a>

 If you're using CodeArtifact to install Python dependencies, Amazon MWAA requires an active token\. To allow Amazon MWAA to access a CodeArtifact repository at runtime, your `requirements.txt` file must contain an [https://pip.pypa.io/en/stable/cli/pip_install/#cmdoption-extra-index-url](https://pip.pypa.io/en/stable/cli/pip_install/#cmdoption-extra-index-url) with the token\. To do so, you must set up a mechanism to update the token in `requirements.txt` on a recurring basis before it expires\. 

 The following topic describes how you can create a DAG that uses the [https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/codeartifact.html#CodeArtifact.Client.get_authorization_token](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/codeartifact.html#CodeArtifact.Client.get_authorization_token) CodeArtifact API operation to retrieve a fresh token every six hours\. Your DAG stores the new token in a `.txt` file in your environment's Amazon S3 bucket\. 

**Topics**
+ [Version](#samples-code-artifact-version)
+ [Prerequisites](#samples-code-artifact-prereqs)
+ [Permissions](#samples-code-artifact-permissions)
+ [Requirements](#samples-hive-dependencies)
+ [Code sample](#samples-code-artifact-code)
+ [What's next?](#samples-code-artifact-next-up)

## Version<a name="samples-code-artifact-version"></a>
+ You can use the code example on this page with **Apache Airflow v2 and above** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="samples-code-artifact-prereqs"></a>

To use the sample code on this page, you'll need the following:
+  An [Amazon MWAA environment](get-started.md)\. 
+  A [CodeArtifact repository](https://docs.aws.amazon.com/codeartifact/latest/ug/create-repo.html) where you store dependencies for your environment\. 

## Permissions<a name="samples-code-artifact-permissions"></a>

To refresh the CodeArtifact token and write the result to Amazon S3 Amazon MWAA must have the following permissions in the execution role\. 
+  The `codeartifact:GetAuthorizationToken` action allows Amazon MWAA to retrieve a new token from CodeArtifact\. The following policy grants permission for every CodeArtifact domain you create\. You can further restrict access to your domains by modifying the resource value in the statement, and specifying only the domains that you want your environment to access\. 

  ```
  {
    "Effect": "Allow",
    "Action": "codeartifact:GetAuthorizationToken",
    "Resource": "arn:aws:codeartifact:us-west-2:*:domain/*"
  }
  ```
+  The `sts:GetServiceBearerToken` action is required to call the CodeArtifact [https://docs.aws.amazon.com/codeartifact/latest/APIReference/API_GetAuthorizationToken.html](https://docs.aws.amazon.com/codeartifact/latest/APIReference/API_GetAuthorizationToken.html) API operation\. This operation returns a token that must be used when using a package manager such as `pip` with CodeArtifact\. To use a package manager with a CodeArtifact repository, your environment's execution role role must allow `sts:GetServiceBearerToken` as shown in the following policy statement\. 

  ```
  {
    "Sid": "AllowServiceBearerToken",
    "Effect": "Allow",
    "Action": "sts:GetServiceBearerToken",
    "Resource": "*"
  }
  ```
+  The `s3:PutObject` action allows Amazon MWAA to write to Amazon S3\. Amazon MWAA uses this permission to replace the `.txt` file containing the CodeArtifact URL in your bucket with a new file containing a fresh token\. 

  ```
  {
    "Effect": "Allow",
    "Action": [
        "s3:GetObject*",
        "s3:GetBucket*",
        "s3:PutObject*",
        "s3:List*"
    ],
    "Resource": [
        "arn:aws:s3:::your-mwaa-bucket",
        "arn:aws:s3:::your-mwaa-bucket/*"
    ]
  }
  ```

## Requirements<a name="samples-hive-dependencies"></a>

To use the sample code on this page, add the following dependencies\. To learn more, see [Installing Python dependencies](working-dags-dependencies.md)\.

You'll need to add the following files to your Amazon S3 bucket\.

1. Add a `codeartifact.txt` file to the `dags` folder in your Amazon S3 bucket\. For example:

   ```
   --extra-index-url https://aws:{auth_token}@your-codeartifact-domain-123456789012.d.codeartifact.us-west-2.amazonaws.com/pypi/your_repo/simple/
   ```

1. Add a `requirements.txt` file in your Amazon S3 bucket with the following line\. For example:

   ```
   -r /usr/local/airflow/dags/codeartifact.txt
   ```

## Code sample<a name="samples-code-artifact-code"></a>

The following steps describe how you can create the DAG code that updates the token\.

1. In your command prompt, navigate to the directory where your DAG code is stored\. For example:

   ```
   cd dags
   ```

1. Copy the contents of the following code sample and save locally as `code_artifact.py`\. You can change the `schedule_interval` cron expression to manage how frequently the token is refreshed\.

   ```
   from airflow.decorators import dag, task
   from datetime import datetime, timedelta
   import os
   import boto3
   from io import StringIO
   
   DAG_ID = os.path.basename(__file__).replace(".py", "")
   
   S3_BUCKET = 'your-mwaa-bucket'
   S3_KEY = 'dags/codeartifact.txt' 
   MAX_SECONDS = 43200 # 12 hours
   DOMAIN = "your-codeartifact-domain"
   OWNER = "your-account"
   
   @task()
   def update_token():
       s3 = boto3.client('s3')
       codeartifact = boto3.client('codeartifact')
   
       response = codeartifact.get_authorization_token(
           domain=DOMAIN,
           domainOwner=OWNER,
           durationSeconds=MAX_SECONDS
       )
   
       obj = s3.get_object(Bucket=S3_BUCKET, Key=S3_KEY)
       s_in = obj['Body'].read().decode('utf-8')
   
       req_lines = s_in.splitlines()
       for i in range(0,len(req_lines)):
           if req_lines[i].startswith("--extra-index-url"):
               s_new=req_lines[i][0:req_lines[i].find('aws:')+4] + response['authorizationToken'] + req_lines[i][req_lines[i].find('@'):]+"\r\n"
               print("old:",req_lines[i])
               print("new:",s_new)
               req_lines[i]=s_new
           else:
               req_lines[i]=req_lines[i]+"\r\n"
   
       s_out=""
       f = StringIO(s_out)
       f.writelines(req_lines)
   
       s3.put_object(Bucket=S3_BUCKET, Key=S3_KEY, Body=f.getvalue())
   
   @dag(
       dag_id=DAG_ID,
       schedule_interval="0 */6 * * *",  # Refreshes the token every 6 hours
       start_date=datetime(2022, 1, 1),
       dagrun_timeout=timedelta(minutes=60),
       catchup=False,
       )
   def update_codeartifact_dag():
       t = update_token()
   
   my_update_codeartifact_dag = update_codeartifact_dag()
   ```

1.  Run the following AWS CLI command to copy the DAG to your environment's bucket, then trigger the DAG using the Apache Airflow UI\. 

   ```
   $ aws s3 cp your-dag.py s3://your-environment-bucket/dags/
   ```

1. If successful, you'll output similar to the following in the task logs for the `update_token` task:

   ```
   [2022-01-01, 12:00:00 PDT] {{logging_mixin.py:109}} INFO - old: --extra-index-url https://aws:$AUTH_TOKEN@mwaa-test-695325144837.d.codeartifact.us-west-2.amazonaws.com/pypi/mwaa-test/simple/
   [2022-01-01, 12:00:00 PDT] {{logging_mixin.py:109}} INFO - new: --extra-index-url https://aws:your-new-token@mwaa-test-695325144837.d.codeartifact.us-west-2.amazonaws.com/pypi/mwaa-test/simple/
   [2022-01-01, 12:00:00 PDT] {{warnings.py:110}} WARNING - /usr/local/lib/python3.7/site-packages/watchtower/__init__.py:349: WatchtowerWarning: Received empty message. Empty messages cannot be sent to CloudWatch Logs
     warnings.warn("Received empty message. Empty messages cannot be sent to CloudWatch Logs", WatchtowerWarning)
   
   [2022-01-01, 12:00:00 PDT] {{python.py:152}} INFO - Done. Returned value was: None
   [2022-01-01, 12:00:00 PDT] {{taskinstance.py:1280}} INFO - Marking task as SUCCESS. dag_id=code-artifact-test, task_id=update_token, execution_date=20220101T120000, start_date=20220101T120000, end_date=20220101T120000
   [2022-01-01, 12:00:00 PDT] {{local_task_job.py:154}} INFO - Task exited with return code 0
   ```

## What's next?<a name="samples-code-artifact-next-up"></a>
+ Learn how to upload the `requirements.txt` file in this example to your Amazon S3 bucket in [Installing Python dependencies](working-dags-dependencies.md)\.
+ Learn how to upload the DAG code in this example to the `dags` folder in your Amazon S3 bucket in [Adding or updating DAGs](configuring-dag-folder.md)\.
+ Learn more about how to upload the `plugins.zip` file in this example to your Amazon S3 bucket in [Installing custom plugins](configuring-dag-import-plugins.md)\.