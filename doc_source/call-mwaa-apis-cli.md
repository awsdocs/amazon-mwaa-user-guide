# Creating an Apache Airflow CLI token<a name="call-mwaa-apis-cli"></a>

You can use the commands on this page to generate a CLI token, and then make Amazon Managed Workflows for Apache Airflow \(MWAA\) API calls directly in your command shell\. For example, you can get a token, then deploy DAGs programmatically using Amazon MWAA APIs\. The following section includes the steps to create an Apache Airflow CLI token using the AWS CLI, a curl script, a Python script, or a bash script\. The token returned in the response is valid for 60 seconds\. 

**Note**  
 The AWS CLI token is intended as a replacement for synchronous shell actions, not asynchronous API commands\. As such, available concurrency is limited\. To ensure that the web server remains responsive for users, it is recommended not to open a new AWS CLI request until the previous one completes successfully\. 

**Contents**
+ [Prerequisites](#call-mwaa-apis-cli-prereqs)
  + [Access](#access-airflow-ui-prereqs-access)
  + [AWS CLI](#access-airflow-ui-prereqs-cli)
+ [Using the AWS CLI](#create-cli-token-cli)
+ [Using a curl script](#create-cli-token-curl)
+ [Using a bash script](#create-cli-token-bash)
+ [Using a Python script](#create-cli-token-python)
+ [What's next?](#mwaa-cli-next-up)

## Prerequisites<a name="call-mwaa-apis-cli-prereqs"></a>

The following section describes the preliminary steps required to use the commands and scripts on this page\.

### Access<a name="access-airflow-ui-prereqs-access"></a>
+ AWS account access in AWS Identity and Access Management \(IAM\) to the Amazon MWAA permissions policy in [Apache Airflow UI access policy: AmazonMWAAWebServerAccess](access-policies.md#web-ui-access)\.
+ AWS account access in AWS Identity and Access Management \(IAM\) to the Amazon MWAA permissions policy [Full API and console access policy: AmazonMWAAFullApiAccess](access-policies.md#full-access-policy)\.

### AWS CLI<a name="access-airflow-ui-prereqs-cli"></a>

The AWS Command Line Interface \(AWS CLI\) is an open source tool that enables you to interact with AWS services using commands in your command\-line shell\. To complete the steps on this page, you need the following:
+ [AWS CLI – Install version 2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)\.
+ [AWS CLI – Quick configuration with `aws configure`](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)\.

## Using the AWS CLI<a name="create-cli-token-cli"></a>

The following example uses the [create\-cli\-token](https://docs.aws.amazon.com/cli/latest/reference/mwaa/create-cli-token.html) command in the AWS CLI to create an Apache Airflow CLI token\.

```
aws mwaa create-cli-token --name YOUR_ENVIRONMENT_NAME
```

## Using a curl script<a name="create-cli-token-curl"></a>

The following example uses a curl script to call the [create\-web\-login\-token](https://docs.aws.amazon.com/cli/latest/reference/mwaa/create-cli-token.html) command in the AWS CLI to invoke the Apache Airflow CLI via an endpoint on the Apache Airflow web server\.

------
#### [ Airflow v2\.0\.2 ]

1. Copy the cURL statement from your text file and paste it in your command shell\.
**Note**  
After copying it to your clipboard, you may need to use **Edit > Paste** from your shell menu\.

   ```
   CLI_JSON=$(aws mwaa --region YOUR_REGION create-cli-token --name YOUR_HOST_NAME) \
     && CLI_TOKEN=$(echo $CLI_JSON | jq -r '.CliToken') \
     && WEB_SERVER_HOSTNAME=$(echo $CLI_JSON | jq -r '.WebServerHostname') \
     && CLI_RESULTS=$(curl --request POST "https://$WEB_SERVER_HOSTNAME/aws_mwaa/cli" \
     --header "Authorization: Bearer $CLI_TOKEN" \
     --header "Content-Type: text/plain" \
     --data-raw "dags trigger YOUR_DAG_NAME") \
     && echo "Output:" \
     && echo $CLI_RESULTS | jq -r '.stdout' | base64 --decode \
     && echo "Errors:" \
     && echo $CLI_RESULTS | jq -r '.stderr' | base64 --decode
   ```

1. Substitute the placeholders for `YOUR_REGION` with the AWS region for your environment, `YOUR_DAG_NAME`, and `YOUR_HOST_NAME`\. For example, a host name for a public network may look like this \(without the *https://\)*:

   ```
   123456a0-0101-2020-9e11-1b159eec9000.c2.us-east-1.airflow.amazonaws.com
   ```

1. You should see the following in your command prompt:

   ```
   {
     "stderr":"<STDERR of the CLI execution (if any), base64 encoded>",
     "stdout":"<STDOUT of the CLI execution, base64 encoded>"
   }
   ```

------
#### [ Airflow v1\.10\.12 ]

1. Copy the cURL statement from your text file and paste it in your command shell\.
**Note**  
After copying it to your clipboard, you may need to use **Edit > Paste** from your shell menu\.

   ```
   CLI_JSON=$(aws mwaa --region YOUR_REGION create-cli-token --name YOUR_HOST_NAME) \
     && CLI_TOKEN=$(echo $CLI_JSON | jq -r '.CliToken') \
     && WEB_SERVER_HOSTNAME=$(echo $CLI_JSON | jq -r '.WebServerHostname') \
     && CLI_RESULTS=$(curl --request POST "https://$WEB_SERVER_HOSTNAME/aws_mwaa/cli" \
     --header "Authorization: Bearer $CLI_TOKEN" \
     --header "Content-Type: text/plain" \
     --data-raw "trigger_dag YOUR_DAG_NAME") \
     && echo "Output:" \
     && echo $CLI_RESULTS | jq -r '.stdout' | base64 --decode \
     && echo "Errors:" \
     && echo $CLI_RESULTS | jq -r '.stderr' | base64 --decode
   ```

1. Substitute the placeholders for `YOUR_REGION` with the AWS region for your environment, `YOUR_DAG_NAME`, and `YOUR_HOST_NAME`\. For example, a host name for a public network may look like this \(without the *https://\)*:

   ```
   123456a0-0101-2020-9e11-1b159eec9000.c2.us-east-1.airflow.amazonaws.com
   ```

1. You should see the following in your command prompt:

   ```
   {
     "stderr":"<STDERR of the CLI execution (if any), base64 encoded>",
     "stdout":"<STDOUT of the CLI execution, base64 encoded>"
   }
   ```

1. Substitute the placeholders for `YOUR_ENVIRONMENT_NAME` and `YOUR_DAG_NAME`\.

------

## Using a bash script<a name="create-cli-token-bash"></a>

The following example uses a bash script to call the [create\-cli\-token](https://docs.aws.amazon.com/cli/latest/reference/mwaa/create-cli-token.html) command in the AWS CLI to create an Apache Airflow CLI token\.

------
#### [ Airflow v2\.0\.2 ]

1. Copy the contents of the following code sample and save locally as `get-cli-token.sh`\.

   ```
   # brew install jq
     aws mwaa create-cli-token --name YOUR_ENVIRONMENT_NAME | export CLI_TOKEN=$(jq -r .CliToken) && curl --request POST "https://YOUR_HOST_NAME/aws_mwaa/cli" \
         --header "Authorization: Bearer $CLI_TOKEN" \
         --header "Content-Type: text/plain" \
         --data-raw "dags trigger YOUR_DAG_NAME"
   ```

1. Substitute the placeholders in *red* for `YOUR_ENVIRONMENT_NAME`, `YOUR_HOST_NAME`, and `YOUR_DAG_NAME`\. For example, a host name for a public network may look like this \(without the *https://\)*:

   ```
   123456a0-0101-2020-9e11-1b159eec9000.c2.us-east-1.airflow.amazonaws.com
   ```

1. \(optional\) macOS and Linux users may need to run the following command to ensure the script is executable\.

   ```
   chmod +x get-cli-token.sh
   ```

1. Run the following script to create an Apache Airflow CLI token\.

   ```
   ./get-cli-token.sh
   ```

------
#### [ Airflow v1\.10\.12 ]

1. Copy the contents of the following code sample and save locally as `get-cli-token.sh`\.

   ```
   # brew install jq
     aws mwaa create-cli-token --name YOUR_ENVIRONMENT_NAME | export CLI_TOKEN=$(jq -r .CliToken) && curl --request POST "https://YOUR_HOST_NAME/aws_mwaa/cli" \
         --header "Authorization: Bearer $CLI_TOKEN" \
         --header "Content-Type: text/plain" \
         --data-raw "trigger_dag YOUR_DAG_NAME"
   ```

1. Substitute the placeholders in *red* for `YOUR_ENVIRONMENT_NAME`, `YOUR_HOST_NAME`, and `YOUR_DAG_NAME`\. For example, a host name for a public network may look like this \(without the *https://\)*:

   ```
   123456a0-0101-2020-9e11-1b159eec9000.c2.us-east-1.airflow.amazonaws.com
   ```

1. \(optional\) macOS and Linux users may need to run the following command to ensure the script is executable\.

   ```
   chmod +x get-cli-token.sh
   ```

1. Run the following script to create an Apache Airflow CLI token\.

   ```
   ./get-cli-token.sh
   ```

------

## Using a Python script<a name="create-cli-token-python"></a>

The following example uses the [boto3 create\_cli\_token](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/mwaa.html#MWAA.Client.create_cli_token) method in a Python script to create an Apache Airflow CLI token and trigger a DAG\. You can run this script outside of Amazon MWAA\. The only thing you need to do is install the boto3 library\. You may want to create a virtual environment to install the library\. It assumes you have [configured AWS authentication credentials](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html#configuration) for your account\. 

------
#### [ Airflow v2\.0\.2 ]

1. Copy the contents of the following code sample and save locally as `create-cli-token.py`\.

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
   import boto3
   import json
   import requests 
   import base64
   
   mwaa_env_name = 'YOUR_ENVIRONMENT_NAME'
   dag_name = 'YOUR_DAG_NAME'
   mwaa_cli_command = 'dags trigger'
   
   client = boto3.client('mwaa')
   
   mwaa_cli_token = client.create_cli_token(
       Name=mwaa_env_name
   )
   
   mwaa_auth_token = 'Bearer ' + mwaa_cli_token['CliToken']
   mwaa_webserver_hostname = 'https://{0}/aws_mwaa/cli'.format(mwaa_cli_token['WebServerHostname'])
   raw_data = '{0} {1}'.format(mwaa_cli_command, dag_name)
   
   mwaa_response = requests.post(
           mwaa_webserver_hostname,
           headers={
               'Authorization': mwaa_auth_token,
               'Content-Type': 'text/plain'
               },
           data=raw_data
           )
           
   mwaa_std_err_message = base64.b64decode(mwaa_response.json()['stderr']).decode('utf8')
   mwaa_std_out_message = base64.b64decode(mwaa_response.json()['stdout']).decode('utf8')
   
   print(mwaa_response.status_code)
   print(mwaa_std_err_message)
   print(mwaa_std_out_message)
   ```

1. Substitute the placeholders for `YOUR_ENVIRONMENT_NAME` and `YOUR_DAG_NAME`\.

1. Run the following script to create an Apache Airflow CLI token\.

   ```
   python3 create-cli-token.py
   ```

------
#### [ Airflow v1\.10\.12 ]

1. Copy the contents of the following code sample and save locally as `create-cli-token.py`\.

   ```
   import boto3
   import json
   import requests 
   import base64
   
   mwaa_env_name = 'YOUR_ENVIRONMENT_NAME'
   dag_name = 'YOUR_DAG_NAME'
   mwaa_cli_command = 'trigger_dag'
   
   client = boto3.client('mwaa')
   
   mwaa_cli_token = client.create_cli_token(
       Name=mwaa_env_name
   )
   
   mwaa_auth_token = 'Bearer ' + mwaa_cli_token['CliToken']
   mwaa_webserver_hostname = 'https://{0}/aws_mwaa/cli'.format(mwaa_cli_token['WebServerHostname'])
   raw_data = '{0} {1}'.format(mwaa_cli_command, dag_name)
   
   mwaa_response = requests.post(
           mwaa_webserver_hostname,
           headers={
               'Authorization': mwaa_auth_token,
               'Content-Type': 'text/plain'
               },
           data=raw_data
           )
           
   mwaa_std_err_message = base64.b64decode(mwaa_response.json()['stderr']).decode('utf8')
   mwaa_std_out_message = base64.b64decode(mwaa_response.json()['stdout']).decode('utf8')
   
   print(mwaa_response.status_code)
   print(mwaa_std_err_message)
   print(mwaa_std_out_message)
   ```

1. Substitute the placeholders for `YOUR_ENVIRONMENT_NAME` and `YOUR_DAG_NAME`\.

1. Run the following script to create an Apache Airflow CLI token\.

   ```
   python3 create-cli-token.py
   ```

------

## What's next?<a name="mwaa-cli-next-up"></a>
+ Explore the Amazon MWAA API operation used to create a CLI token at [CreateCliToken](https://docs.aws.amazon.com/mwaa/latest/API/API_CreateCliToken.html)\.