# Creating an Apache Airflow web login token<a name="call-mwaa-apis-web"></a>

You can use the commands on this page to generate a web login token, and then make Amazon Managed Workflows for Apache Airflow \(MWAA\) API calls directly in your command shell\. For example, you can get a token, then deploy DAGs programmatically using Amazon MWAA APIs\. The following section includes the steps to create an Apache Airflow web login token using the AWS CLI, a bash script, a POST API request, or a Python script\. The token returned in the response is valid for 60 seconds\.

**Contents**
+ [Prerequisites](#call-mwaa-apis-web-prereqs)
  + [Access](#access-airflow-ui-prereqs-access)
  + [AWS CLI](#access-airflow-ui-prereqs-cli)
+ [Using the AWS CLI](#create-web-login-token-cli)
+ [Using a bash script](#create-web-login-token-bash)
+ [Using a POST API request](#create-web-login-token-post-api)
+ [Using a Python script](#create-web-login-token-python)
+ [What's next?](#mwaa-webcli-next-up)

## Prerequisites<a name="call-mwaa-apis-web-prereqs"></a>

The following section describes the preliminary steps required to use the commands and scripts on this page\.

### Access<a name="access-airflow-ui-prereqs-access"></a>
+ AWS account access in AWS Identity and Access Management \(IAM\) to the Amazon MWAA permissions policy in [Apache Airflow UI access policy: AmazonMWAAWebServerAccess](access-policies.md#web-ui-access)\.
+ AWS account access in AWS Identity and Access Management \(IAM\) to the Amazon MWAA permissions policy [Full API and console access policy: AmazonMWAAFullApiAccess](access-policies.md#full-access-policy)\.

### AWS CLI<a name="access-airflow-ui-prereqs-cli"></a>

The AWS Command Line Interface \(AWS CLI\) is an open source tool that enables you to interact with AWS services using commands in your command\-line shell\. To complete the steps on this page, you need the following:
+ [AWS CLI – Install version 2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
+ [AWS CLI – Quick configuration with `aws configure`](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)

## Using the AWS CLI<a name="create-web-login-token-cli"></a>

The following example uses the [create\-web\-login\-token](https://docs.aws.amazon.com/cli/latest/reference/mwaa/create-web-login-token.html) command in the AWS CLI to create an Apache Airflow web login token\.

```
aws mwaa create-web-login-token --name YOUR_ENVIRONMENT_NAME
```

## Using a bash script<a name="create-web-login-token-bash"></a>

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

## Using a POST API request<a name="create-web-login-token-post-api"></a>

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

## Using a Python script<a name="create-web-login-token-python"></a>

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

## What's next?<a name="mwaa-webcli-next-up"></a>
+ Explore the Amazon MWAA API operation used to create a web login token at [CreateWebLoginToken](https://docs.aws.amazon.com/mwaa/latest/API/API_CreateWebLoginToken.html)\.