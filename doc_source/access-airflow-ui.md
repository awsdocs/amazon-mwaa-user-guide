# Accessing the Apache Airflow UI<a name="access-airflow-ui"></a>

An Apache Airflow UI link is available on the Amazon Managed Workflows for Apache Airflow \(MWAA\) console after you create an environment\. You can use the Amazon MWAA console to view and invoke a DAG in your Apache Airflow UI, or use Amazon MWAA APIs to get a token and invoke a DAG\. This page describes the permissions needed to access the Apache Airflow UI, how to generate a token to make Amazon MWAA API calls directly in your command shell, and the supported commands in the Apache Airflow CLI\.

**Topics**
+ [Prerequisites](#access-airflow-ui-prereqs)
+ [Open Airflow UI](#access-airflow-ui-onconsole)
+ [Logging into the Apache Airflow UI](#access-airflow-ui-login)
+ [Examples to create an Apache Airflow web login token](#call-mwaa-apis-web)
+ [Examples to create an Apache Airflow CLI token](#call-mwaa-apis-cli)
+ [Apache Airflow CLI command reference](#airflow-cli-commands-supported)
+ [Sample code with Apache Airflow commands](#airflow-cli-commands-sample-code)
+ [Using AWS blogs and tutorials](#airflow-cli-commands-tutorials)

## Prerequisites<a name="access-airflow-ui-prereqs"></a>

The following section describes the preliminary steps required to use the commands and scripts on this page\.

### Access<a name="access-airflow-ui-prereqs-access"></a>
+ AWS account access in AWS Identity and Access Management \(IAM\) to the Amazon MWAA permissions policy in [Apache Airflow UI access policy: AmazonMWAAWebServerAccess](access-policies.md#web-ui-access)\.
+ AWS account access in AWS Identity and Access Management \(IAM\) to the Amazon MWAA permissions policy [Full API and console access policy: AmazonMWAAFullApiAccess](access-policies.md#full-access-policy)\.

### AWS CLI<a name="access-airflow-ui-prereqs-cli"></a>
+ The AWS Command Line Interface \(AWS CLI\) is an open source tool that enables you to interact with AWS services using commands in your command\-line shell\. To complete the steps in this section, you need the following:
  + [AWS CLI – Install version 2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
  + [AWS CLI – Quick configuration with `aws configure`](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)

## Open Airflow UI<a name="access-airflow-ui-onconsole"></a>

The following image shows the link to your Apache Airflow UI on the Amazon MWAA console\.

![\[This image shows the link to your Apache Airflow UI on the Amazon MWAA console.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-aa-ui.png)

## Logging into the Apache Airflow UI<a name="access-airflow-ui-login"></a>

The following steps describe how to log\-in to your Apache Airflow UI\.

**To access your Apache Airflow UI**

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose **Open Airflow UI**\.

**Note**  
You may need to ask your account administrator to add `AmazonMWAAWebServerAccess` permissions for your account to view your Apache Airflow UI\. For more information, see [Managing access](https://docs.aws.amazon.com/mwaa/latest/userguide/manage-access.html)\.

**To log\-in to your Apache Airflow UI**
+ Enter the AWS Identity and Access Management \(IAM\) user name and password for your account\.

![\[This image shows how to log-in to your Apache Airflow UI.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-aa-ui-login.png)

## Examples to create an Apache Airflow web login token<a name="call-mwaa-apis-web"></a>

You can use the following commands to generate a web login token, and then make Amazon MWAA API calls directly in your command shell\. For example, you can get a token, then deploy DAGs programmatically using Amazon MWAA APIs\. The following section includes the steps to create an Apache Airflow web login token using the AWS CLI, a bash script, a POST API request, or a Python script\. The token returned in the response is valid for 60 seconds\.

**Contents**
+ [Using the AWS CLI](#create-web-login-token-cli)
+ [Using a bash script](#create-web-login-token-bash)
+ [Using a POST API request](#create-web-login-token-post-api)
+ [Using a Python script](#create-web-login-token-python)

### Using the AWS CLI<a name="create-web-login-token-cli"></a>

The following example uses the [create\-web\-login\-token](https://docs.aws.amazon.com/cli/latest/reference/mwaa/create-web-login-token.html) command in the AWS CLI to create an Apache Airflow web login token\.

```
aws mwaa create-web-login-token --name YOUR_ENVIRONMENT_NAME
```

### Using a bash script<a name="create-web-login-token-bash"></a>

The following example uses a bash script to call the [create\-web\-login\-token](https://docs.aws.amazon.com/cli/latest/reference/mwaa/create-web-login-token.html) command in the AWS CLI to create an Apache Airflow web login token\.

1. Copy the contents of the following code sample and save locally as `get-web-token.sh`\.

   ```
   #!/bin/bash
   HOST=YOUR_HOST_NAME
   YOUR_URL=https://$HOST/aws_mwaa/aws-console-sso?login=true#
   WEB_TOKEN=$(aws mwaa create-web-login-token --name YOUR_ENVIRONMENT_NAME --query WebToken --output text)
   echo $YOUR_URL$WEB_TOKEN
   ```

1. Substitute the placeholders in *red* for `YOUR_HOST_NAME` and `YOUR_ENVIRONMENT_NAME`\. For example, a host name for a public network may look like this \(without the *https://\)*:

   ```
   123456a0-0101-2020-9e11-1b159eec9000.c2.us-east-1.airflow.amazonaws.com
   ```

1. \(optional\) macOS and Linux users may need to run the following command to ensure the script is executable\.

   ```
   chmod +x get-web-token.sh
   ```

1. Run the following script to get a web login token\.

   ```
   ./get-web-token.sh
   ```

1. You should see the following in your command prompt:

   ```
   https://123456a0-0101-2020-9e11-1b159eec9000.c2.us-east-1.airflow.amazonaws.com/aws_mwaa/aws-console-sso?login=true#{your-web-login-token}
   ```

### Using a POST API request<a name="create-web-login-token-post-api"></a>

The following example uses a POST API request to create an Apache Airflow web login token\.

1. Copy the following URL and paste in the URL field of your REST API client\.

   ```
   https://YOUR_HOST_NAME/aws_mwaa/aws-console-sso?login=true#WebToken
   ```

1. Substitute the placeholders in *red* for `YOUR_HOST_NAME`\. For example, a host name for a public network may look like this \(without the *https://\)*:

   ```
   123456a0-0101-2020-9e11-1b159eec9000.c2.us-east-1.airflow.amazonaws.com
   ```

1. Copy the following JSON and paste in the body field of your REST API client\.

   ```
   {
     "name": "YOUR_ENVIRONMENT_NAME"
   }
   ```

1. Substitute the placeholder in *red* for `YOUR_ENVIRONMENT_NAME`\.

1. Add key\-value pairs in the authorization field\. For example, if you're using Postman, choose **AWS Signature**, and then enter your:
   + `AWS_ACCESS_KEY_ID` in **AccessKey**
   + `AWS_SECRET_ACCESS_KEY` in **SecretKey**

1. You should see the following response:

   ```
   {
     "webToken": "<Short-lived token generated for enabling access to the Apache Airflow Webserver UI>",
     "webServerHostname": "<Hostname for the WebServer of the environment>"
   }
   ```

### Using a Python script<a name="create-web-login-token-python"></a>

The following example uses the [boto3 create\_web\_login\_token](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/mwaa.html#MWAA.Client.create_web_login_token) method in a Python script to create an Apache Airflow web login token\. You can run this script outside of Amazon MWAA\. The only thing you need to do is install the boto3 library\. You may want to create a virtual environment to install the library\. It assumes you have [configured AWS authentication credentials](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html#configuration) for your account\. 

1. Copy the contents of the following code sample and save locally as `create-web-login-token.py`\.

   ```
   import boto3
   mwaa = boto3.client('mwaa')
   response = mwaa.create_web_login_token(
       Name="YOUR_ENVIRONMENT_NAME"
   )
   webServerHostName = response["WebServerHostname"]
   webToken = response["WebToken"]
   airflowUIUrl = 'https://{0}/aws_mwaa/aws-console-sso?login=true#{1}'.format(webServerHostName, webToken)
   print("Here is your Airflow UI URL: ")
   airflowUIUrl
   ```

1. Substitute the placeholder in *red* for `YOUR_ENVIRONMENT_NAME`\.

1. Run the following script to get a web login token\.

   ```
   python3 create-web-login-token.py
   ```

## Examples to create an Apache Airflow CLI token<a name="call-mwaa-apis-cli"></a>

You can use the following commands to generate a CLI token, and then make Amazon MWAA API calls directly in your command shell\. For example, you can get a token, then deploy DAGs programmatically using Amazon MWAA APIs\. The following section includes the steps to create an Apache Airflow CLI token using the AWS CLI, a curl script, a Python script, or a bash script\. The token returned in the response is valid for 60 seconds\.

**Contents**
+ [Using the AWS CLI](#create-cli-token-cli)
+ [Using a curl script](#create-cli-token-curl)
+ [Using a bash script](#create-cli-token-bash)
+ [Using a Python script](#create-cli-token-python)

### Using the AWS CLI<a name="create-cli-token-cli"></a>

The following example uses the [create\-cli\-token](https://docs.aws.amazon.com/cli/latest/reference/mwaa/create-cli-token.html) command in the AWS CLI to create an Apache Airflow CLI token\.

```
aws mwaa create-cli-token --name YOUR_ENVIRONMENT_NAME
```

### Using a curl script<a name="create-cli-token-curl"></a>

The following example uses a curl script to call the [create\-web\-login\-token](https://docs.aws.amazon.com/cli/latest/reference/mwaa/create-cli-token.html) command in the AWS CLI to invoke the Apache Airflow CLI via an endpoint on the Apache Airflow web server\.

1. Copy the cURL statement from your text file and paste it in your command shell\.
**Note**  
After copying it to your clipboard, you may need to use **Edit > Paste** from your shell menu\.

   ```
   CLI_JSON=$(aws mwaa create-cli-token --name YOUR_HOST_NAME) \
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

1. Substitute the placeholders in *red* for `YOUR_HOST_NAME` and `YOUR_DAG_NAME`\. For example, a host name for a public network may look like this \(without the *https://\)*:

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

### Using a bash script<a name="create-cli-token-bash"></a>

The following example uses a bash script to call the [create\-cli\-token](https://docs.aws.amazon.com/cli/latest/reference/mwaa/create-cli-token.html) command in the AWS CLI to create an Apache Airflow CLI token\.

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

### Using a Python script<a name="create-cli-token-python"></a>

The following example uses the [boto3 create\_cli\_token](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/mwaa.html#MWAA.Client.create_cli_token) method in a Python script to create an Apache Airflow CLI token and trigger a DAG\. You can run this script outside of Amazon MWAA\. The only thing you need to do is install the boto3 library\. You may want to create a virtual environment to install the library\. It assumes you have [configured AWS authentication credentials](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html#configuration) for your account\. 

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
   raw_data = '{0} {1}'.format(mwaa_cli_command, YOUR_DAG_NAME)
   
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

1. Substitute the placeholders in *red* for `YOUR_ENVIRONMENT_NAME` and `YOUR_DAG_NAME`\.

1. Run the following script to create an Apache Airflow CLI token\.

   ```
   python3 create-cli-token.py
   ```

## Apache Airflow CLI command reference<a name="airflow-cli-commands-supported"></a>

The following section describes the supported and unsupported Apache Airflow CLI commands\.

### Supported commands<a name="airflow-cli-commands-supported"></a>

The following [Apache Airflow CLI commands](https://airflow.apache.org/docs/apache-airflow/stable/cli-and-env-variables-ref.html) are supported when using Apache Airflow in an Amazon MWAA environment:

**Note**  
Any command that parses a DAG \(such as `list_dags`, `backfill`\) will fail if the DAG uses plugins that depend on packages that are installed through `requirements.txt`\.
+ clear
+ `dag_state`
+ `delete_dag`
+ `list_dag_runs`
+ list\_tasks
+ `next_execution`
+ `pause`
+ `pool`
+ `render`
+ `run`
+ `show_dag`
+ `task_failed_deps`
+ `task_state`
+ `test`
+ `trigger_dag`
+ `unpause`
+ `variables`
+ `version`

### Unsupported commands<a name="airflow-unsupported-cli-commands"></a>

The following Apache Airflow CLI commands are not supported when running Apache Airflow in an Amazon MWAA environment\.
+ backfill
+  checkdb
+ `connections`
+ `create_user`
+ `delete_user`
+ `flower`
+ `initdb`
+ `kerberos`
+ list\_dags
+ list\_users
+ `resetdb`
+ `rotate_fernet_key`
+ `scheduler`
+ `serve_logs`
+ `shell`
+ `sync_perm`
+ `upgradedb`
+ `webserver`
+ `worker`

## Sample code with Apache Airflow commands<a name="airflow-cli-commands-sample-code"></a>

The following section contains sample code you can use with Apache Airflow commands\.

### Adding a configuration when triggering a DAG<a name="airflow-cli-commands-trigger"></a>

You can use the following sample code to add a configuration when triggering a DAG, such as `airflow trigger_dag 'dag_name' —conf '{"key":"value"}'`\.

```
import boto3
import json
import requests 
import base64

mwaa_env_name = 'YOUR_ENVIRONMENT_NAME'
dag_name = 'YOUR_DAG_NAME'
key = "YOUR_KEY"
value = "YOUR_VALUE"
conf = "{\"" + key + "\":\"" + value + "\"}"

client = boto3.client('mwaa')

mwaa_cli_token = client.create_cli_token(
    Name=mwaa_env_name
)

mwaa_auth_token = 'Bearer ' + mwaa_cli_token['CliToken']
mwaa_webserver_hostname = 'https://{0}/aws_mwaa/cli'.format(mwaa_cli_token['WebServerHostname'])
raw_data = "trigger_dag {0} -c '{1}'".format(dag_name, conf)

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

## Using AWS blogs and tutorials<a name="airflow-cli-commands-tutorials"></a>

The following section contains other AWS blogs and tutorials with Apache Airflow CLI tokens, web tokens, and commands\.
+ [Interacting with Amazon Managed Workflows for Apache Airflow \(MWAA\) via the command line](https://dev.to/aws/interacting-with-amazon-managed-workflows-for-apache-airflow-via-the-command-line-4e91)