# Using a DAG to import variables in the CLI<a name="samples-variables-import"></a>

The following sample code imports variables using the CLI on Amazon Managed Workflows for Apache Airflow \(MWAA\)\.

**Topics**
+ [Version](#samples-variables-import-version)
+ [Prerequisites](#samples-variables-import-prereqs)
+ [Permissions](#samples-variables-import-permissions)
+ [Dependencies](#samples-variables-import-dependencies)
+ [Code sample](#samples-variables-import-code)
+ [What's next?](#samples-variables-import-next-up)

## Version<a name="samples-variables-import-version"></a>
+ You can use the code example on this page with **Apache Airflow v2 and above** in [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)\.

## Prerequisites<a name="samples-variables-import-prereqs"></a>
+ No additional permissions are required to use the code example on this page\.

## Permissions<a name="samples-variables-import-permissions"></a>

Your AWS account needs access to the `AmazonMWAAAirflowCliAccess` policy\. To learn more, see [Apache Airflow CLI policy: AmazonMWAAAirflowCliAccess](access-policies.md)\.

## Dependencies<a name="samples-variables-import-dependencies"></a>
+ To use this code example with Apache Airflow v2, no additional dependencies are required\. The code uses the [Apache Airflow v2 base install](https://github.com/aws/aws-mwaa-local-runner/blob/main/docker/config/requirements.txt) on your environment\.

## Code sample<a name="samples-variables-import-code"></a>

The following sample code takes three inputs: your Amazon MWAA environment name \(in `mwaa_env`\), the AWS Region of your environment \(in `aws_region`\), and the local file that contains the variables you want to import \(in `var_file`\)\.

```
import boto3
import json
import requests 
import base64
import getopt
import sys

argv = sys.argv[1:]
mwaa_env=''
aws_region=''
var_file=''

try:
    opts, args = getopt.getopt(argv, 'e:v:r:', ['environment', 'variable-file','region'])
    #if len(opts) == 0 and len(opts) > 3:
    if len(opts) != 3:
        print ('Usage: -e MWAA environment -v variable file location and filename -r aws region')
    else:
        for opt, arg in opts:
            if opt in ("-e"):
                mwaa_env=arg
            elif opt in ("-r"):
                aws_region=arg
            elif opt in ("-v"):
                var_file=arg

        boto3.setup_default_session(region_name="{}".format(aws_region))
        mwaa_env_name = "{}".format(mwaa_env)

        client = boto3.client('mwaa')
        mwaa_cli_token = client.create_cli_token(
            Name=mwaa_env_name
        )
        
        with open ("{}".format(var_file), "r") as myfile:
            fileconf = myfile.read().replace('\n', '')

        json_dictionary = json.loads(fileconf)
        for key in json_dictionary:
            print(key, " ", json_dictionary[key])
            val = (key + " " + json_dictionary[key])
            mwaa_auth_token = 'Bearer ' + mwaa_cli_token['CliToken']
            mwaa_webserver_hostname = 'https://{0}/aws_mwaa/cli'.format(mwaa_cli_token['WebServerHostname'])
            raw_data = "variables set {0}".format(val)
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

except:
    print('Use this script with the following options: -e MWAA environment -v variable file location and filename -r aws region')
    print("Unexpected error:", sys.exc_info()[0])
    sys.exit(2)
```

## What's next?<a name="samples-variables-import-next-up"></a>
+ Learn how to upload the DAG code in this example to the `dags` folder in your Amazon S3 bucket in [Adding or updating DAGs](configuring-dag-folder.md)\.