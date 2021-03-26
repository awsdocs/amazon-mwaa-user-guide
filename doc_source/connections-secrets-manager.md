# Configuring an Apache Airflow connection using a Secrets Manager secret key<a name="connections-secrets-manager"></a>

AWS Secrets Manager is a supported alternative Apache Airflow backend on an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment\. This guide shows how to use Secrets Manager to securely store secrets for Apache Airflow variables and an AWS connection for an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment\. 

**Note**  
[AWS Systems Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html) Parameter Store is not a supported backend at this time on Amazon MWAA\.

**Contents**
+ [Prerequisites](#connections-sm-prereqs)
+ [Step one: Permit access of the secret keys to Amazon MWAA](#connections-sm-policy)
+ [Step two: Create the Secrets Manager backend as an Apache Airflow configuration option](#connections-sm-aa-configuration)
+ [Step three: Generate an Apache Airflow AWS connection URI string](#connections-sm-aa-uri)
+ [Step four: Add the variables in Secrets Manager](#connections-sm-createsecret-variables)
+ [Step five: Add the connection in Secrets Manager](#connections-sm-createsecret-connection)
+ [Using AWS blogs and tutorials](#connections-sm-blogs)
+ [What's next?](#connections-sm-next-up)

## Prerequisites<a name="connections-sm-prereqs"></a>

**To use the steps on this page, you'll need:**
+ The required AWS resources configured for your environment as defined in [Get started with Amazon Managed Workflows for Apache Airflow \(MWAA\)](get-started.md)\.

## Step one: Permit access of the secret keys to Amazon MWAA<a name="connections-sm-policy"></a>

The [execution role](mwaa-create-role.md) for your Amazon MWAA environment needs read access to the secret key in AWS Secrets Manager\. The following IAM policy allows read\-write access using the AWS managed [SecretsManagerReadWrite](https://console.aws.amazon.com/iam/home?#/policies/arn:aws:iam::aws:policy/SecretsManagerReadWrite$jsonEditor) policy\.

**To attach the policy to your execution role**

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose your execution role on the **Permissions** pane\.

1. Choose **Attach policies**\.

1. Type `SecretsManagerReadWrite` in the **Filter policies** text field\.

1. Choose **Attach policy**\.

## Step two: Create the Secrets Manager backend as an Apache Airflow configuration option<a name="connections-sm-aa-configuration"></a>

The following section describes how to create an Apache Airflow configuration option on the Amazon MWAA console for the AWS Secrets Manager backend\. If you're using a configuration setting of the same name in `airflow.cfg`, the configuration you create in the following steps will take precedence and override the configuration settings\.

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose **Edit**\.

1. Choose **Next**\.

1. Choose **Add custom configuration** in the **Airflow configuration options** pane\. Add the following key\-value pairs:

   1. `secrets.backend` : `airflow.contrib.secrets.aws_secrets_manager.SecretsManagerBackend`

   1. `secrets.backend_kwargs` : `{"connections_prefix" : "airflow/connections", "variables_prefix" : "airflow/variables"}`

      This tells Apache Airflow to look for the secret at the `airflow/connections/*` and `airflow/variables/*` path\.

1. Choose **Save**\.

## Step three: Generate an Apache Airflow AWS connection URI string<a name="connections-sm-aa-uri"></a>

The key to creating a connection URI string is to use the "tab" key on your keyboard to indent the key\-value pairs in the [Connection](https://airflow.apache.org/docs/stable/howto/connection/index.html) object\. We also recommend creating a variable for the `extra` object in your shell session\. The following section walks you through the steps to [generate an Apache Airflow connection URI](https://airflow.apache.org/docs/apache-airflow/stable/howto/connection.html#generating-a-connection-uri) string for an Amazon MWAA environment using Apache Airflow or a Python script\.

------
#### [ Airflow CLI ]

The following shell session uses your local Airflow CLI to generate a connection string\. If you don't have the CLI installed, we recommend using the Python script\.

1. Open a Python shell session:

   ```
   python3
   ```

1. Enter the following command:

   ```
   >>> import json
   ```

1. Enter the following command:

   ```
   >>> from airflow.models.connection import Connection
   ```

1. Create a variable in your shell session for the `extra` object\. Substitute the sample values in *YOUR\_EXECUTION\_ROLE\_ARN* with the execution role ARN, and the region in *YOUR\_REGION* \(such as `us-east-1`\)\.

   ```
   >>> extra=json.dumps({'role_arn': 'YOUR_EXECUTION_ROLE_ARN', 'region_name': 'YOUR_REGION'})
   ```

1. Create the connection object\. Substitute the sample value in `myconn` with the name of the Apache Airflow connection\.

   ```
   >>> myconn = Connection(
   ```

1. Use the "tab" key on your keyboard to indent each of the following key\-value pairs in your connection object\. Substitute the sample values in *red*\.

   1. Specify the AWS connection type:

      ```
      ... conn_id='aws'
      ```

   1. Specify the Apache Airflow database option:

      ```
      ... conn_type='mysql'
      ```

   1. Specify the Apache Airflow UI URL on Amazon MWAA:

      ```
      ... host='288888a0-50a0-888-9a88-1a111aaa0000.a1.us-east-1.airflow.amazonaws.com/home'
      ```

   1. Specify the AWS access key ID \(username\) to login to Amazon MWAA:

      ```
      ... login='YOUR_AWS_ACCESS_KEY_ID'
      ```

   1. Specify the AWS secret access key \(password\) to login to Amazon MWAA:

      ```
      ... password='YOUR_AWS_SECRET_ACCESS_KEY'
      ```

   1. Specify the `extra` shell session variable:

      ```
      ... extra=extra
      ```

   1. Close the connection object\.

      ```
      ... )
      ```

1. Print the connection URI string:

   ```
   >>> myconn.get_uri()
   ```

   You should see the connection URI string in the response:

   ```
   'mysql://288888a0-50a0-888-9a88-1a111aaa0000.a1.us-east-1.airflow.amazonaws.com%2Fhome?role_arn=arn%3Aaws%3Aiam%3A%3A001122332255%3Arole%2Fservice-role%2FAmazonMWAA-MyAirflowEnvironment-iAaaaA&region_name=us-east-1'
   ```

------
#### [ Python script ]

The following Python script does not require the Apache Airflow CLI\.

1. Copy the contents of the following code sample and save locally as `mwaa_connection.py`\.

   ```
   import urllib.parse
   
   conn_type = 'YOUR_DB_OPTION'
   host = 'YOUR_MWAA_AIRFLOW_UI_URL'
   login = 'YOUR_AWS_ACCESS_KEY_ID'
   password = 'YOUR_AWS_SECRET_ACCESS_KEY'
   role_arn = urllib.parse.quote_plus('YOUR_EXECUTION_ROLE_ARN')
   region_name = 'YOUR_REGION'
   
   conn_string = '{0}://{1}:{2}@{3}?role_arn={4}&region_name={5}'.format(conn_type, host, login, password, role_arn, region_name)
   print(conn_string)
   ```

1. Substitute the placeholders in *red*\.

1. Run the following script to generate a connection string\.

   ```
   python3 mwaa_connection.py
   ```

------

## Step four: Add the variables in Secrets Manager<a name="connections-sm-createsecret-variables"></a>

The following section describes how to create the secret for a variable in Secrets Manager\.

**To create the secret key**

1. Open the [AWS Secrets Manager console](https://console.aws.amazon.com/secretsmanager/home#/environments)\.

1. Choose **Store a new secret**\.

1. Choose **Other type of secrets**\.

1. Choose **Plaintext** on the **Specify the key/value pairs to be stored in this secret** pane\.

1. Add the variable value as **Plaintext** in the following format\.

   ```
   "YOUR_VARIABLE_VALUE"
   ```

   For example, to specify an integer:

   ```
   14
   ```

   For example, to specify a string:

   ```
   "mystring"
   ```

1. Choose an AWS KMS key option from the dropdown list\.

1. Enter a name in the text field for **Secret name** in the following format\.

   ```
   airflow/variables/YOUR_VARIABLE_NAME
   ```

   For example:

   ```
   airflow/variables/max_metadb_storage_days
   ```

1. Leave the remaining options blank, or set to their default values\.

1. Choose **Next**, **Next**, **Store**\.

1. Repeat these steps in Secrets Manager for any additional variables you want to add\.

## Step five: Add the connection in Secrets Manager<a name="connections-sm-createsecret-connection"></a>

The following section describes how to create the secret for your connection string URI in Secrets Manager\.

**To create the secret key**

1. Open the [AWS Secrets Manager console](https://console.aws.amazon.com/secretsmanager/home#/environments)\.

1. Choose **Store a new secret**\.

1. Choose **Other type of secrets**\.

1. Choose **Plaintext** on the **Specify the key/value pairs to be stored in this secret** pane\.

1. Add the connection URI string as **Plaintext** in the following format\.

   ```
   "YOUR_CONNECTION_URI_STRING"
   ```

   For example:

   ```
   "mysql://288888a0-50a0-888-9a88-1a111aaa0000.a1.us-east-1.airflow.amazonaws.com%2Fhome?role_arn=arn%3Aaws%3Aiam%3A%3A001122332255%3Arole%2Fservice-role%2FAmazonMWAA-MyAirflowEnvironment-iAaaaA&region_name=us-east-1"
   ```

1. Choose an AWS KMS key option from the dropdown list\.

1. Enter a name in the text field for **Secret name** in the following format\.

   ```
   airflow/connections/YOUR_CONNECTION_NAME
   ```

   For example:

   ```
   airflow/connections/myconn
   ```

1. Leave the remaining options blank, or set to their default values\.

1. Choose **Next**, **Next**, **Store**\.

**Note**  
If the name of your secret key for the connection is `airflow/connections/mysecret`, you'll use the connection in your DAG as:  

```
conn_id="mysecret", ...)
```

## Using AWS blogs and tutorials<a name="connections-sm-blogs"></a>
+ Use a Python script to migrate a large volume of Apache Airflow variables and connections to Secrets Manager in [Move your Apache Airflow connections and variables to AWS Secrets Manager](https://aws.amazon.com/blogs/opensource/move-apache-airflow-connections-variables-aws-secrets-manager/)\.

## What's next?<a name="connections-sm-next-up"></a>
+ Learn how to generate a token to access the Apache Airflow UI in [Accessing the Apache Airflow UI](access-airflow-ui.md)\.