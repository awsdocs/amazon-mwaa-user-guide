# Invoking DAGs with an AWS Lambda function<a name="samples-lambda"></a>

The following sample code uses an AWS Lambda function to get an Apache Airflow CLI token and invoke a DAG in an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment\. 

**Topics**
+ [Version](#samples-lambda-version)
+ [Prerequisites](#samples-lambda-prereqs)
+ [Permissions](#samples-lambda-permissions)
+ [Dependencies](#samples-lambda-dependencies)
+ [Code sample](#samples-lambda-code)
+ [What's next?](#samples-lambda-next-up)

## Version<a name="samples-lambda-version"></a>
+ The sample code on this page can be used with **Apache Airflow v1\.10\.12** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="samples-lambda-prereqs"></a>

To use the sample code on this page, you'll need the following:
+ The **Public network** option for your [Amazon MWAA environment](get-started.md)\.

**Note**  
You can use this sample code on a private network if the Lambda function and your Amazon MWAA environment are in the same VPC\. The Lambda function’s execution role would need permission to call *CreateNetworkInterface* on EC2 \(using the [AWSLambdaVPCAccessExecutionRole](https://console.aws.amazon.com/https://console.aws.amazon.com/iam/home?#/policies/arn:aws:iam::aws:policy/service-role/AWSLambdaVPCAccessExecutionRole$jsonEditor) policy\)\.

## Permissions<a name="samples-lambda-permissions"></a>

To use the sample code on this page, your AWS account needs access to the `AmazonMWAAAirflowCliAccess` policy\. To learn more, see [Apache Airflow CLI policy: AmazonMWAAAirflowCliAccess](access-policies.md)\.

## Dependencies<a name="samples-lambda-dependencies"></a>

The sample code on this page doesn't require a `requirements.txt` or `plugins.zip` to run on your Amazon MWAA environment\.

## Code sample<a name="samples-lambda-code"></a>

The following sample code uses an AWS Lambda function to get an Apache Airflow CLI token and invoke a DAG in an Amazon MWAA environment\. Copy the sample code and substitute the placeholders with the following:
+ The name of the Amazon MWAA environment in `YOUR_ENVIRONMENT_NAME`\.
+ The name of the DAG you want to invoke in `YOUR_DAG_NAME`\.

```
import boto3
import http.client
import base64
import ast
mwaa_env_name = 'YOUR_ENVIRONMENT_NAME'
dag_name = 'YOUR_DAG_NAME'
mwaa_cli_command = 'trigger_dag'
​
client = boto3.client('mwaa')
​
def lambda_handler(event, context):
    # get web token
    mwaa_cli_token = client.create_cli_token(
        Name=mwaa_env_name
    )
    
    conn = http.client.HTTPSConnection(mwaa_cli_token['WebServerHostname'])
    payload = "trigger_dag " + dag_name
    headers = {
      'Authorization': 'Bearer ' + mwaa_cli_token['CliToken'],
      'Content-Type': 'text/plain'
    }
    conn.request("POST", "/aws_mwaa/cli", payload, headers)
    res = conn.getresponse()
    data = res.read()
    dict_str = data.decode("UTF-8")
    mydata = ast.literal_eval(dict_str)
    return base64.b64decode(mydata['stdout'])
```

## What's next?<a name="samples-lambda-next-up"></a>
+ Learn how to invoke a Lambda function using the AWS CLI in [AWS Lambda function logging in Python](https://docs.aws.amazon.com/lambda/latest/dg/python-logging.html)\.