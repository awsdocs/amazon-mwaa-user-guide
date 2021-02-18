# Accessing the Apache Airflow UI<a name="access-airflow-ui"></a>

A link to your Apache Airflow UI is available on the **Environments** details page on the Amazon MWAA console after you create an environment\. This page describes the permissions needed to access the Apache Airflow UI, and how to generate a token to log into Apache Airflow UI\.

**Topics**
+ [Prerequisites](#access-airflow-ui-prereqs)
+ [Open Airflow UI](#access-airflow-ui-onconsole)
+ [Using a web login token](#createwebLogintoken)
+ [Using a CLI token](#CreateCliToken)
+ [Apache Airflow CLI command reference](#airflow-cli-commands-supported)

## Prerequisites<a name="access-airflow-ui-prereqs"></a>
+ You must have been granted access to the Amazon MWAA permissions policy in [Apache Airflow UI access policy: AmazonMWAAWebServerAccess](access-policies.md#web-ui-access)\.

## Open Airflow UI<a name="access-airflow-ui-onconsole"></a>

The following image shows the link to your Apache Airflow UI on the Amazon MWAA console\.

![\[This image shows the link to your Apache Airflow UI on the Amazon MWAA console.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-aa-ui.png)

## Using a web login token<a name="createwebLogintoken"></a>

The steps in this section use the `CreateWebLoginToken` Amazon MWAA API to create a short\-lived token that allows a user to log into Apache Airflow web UI\. The generated token is valid for 60 seconds\.

**Note**  
This login requires JavaScript on the aws\-console\-sso page to read the web token from the URL fragment, and make a POST call to login\.

The following is an example of a web server access URL that includes a `CreateWebLoginToken` token\.

```
https://<webServerHostName>/aws_mwaa/aws-console-sso?login=true#<webToken>
```

**Request**

```
{
    "name": "<Name of the environment>"
}
```

**Response**

```
{
    "webToken": "<Short-lived token generated for enabling access
                  to the Apache Airflow Webserver UI>",
    "webServerHostname": "<Hostname for the WebServer of the environment>"
}
```

## Using a CLI token<a name="CreateCliToken"></a>

You can use the `CreateCliToken` API action to create a short\-lived token that allows a user to invoke the Airflow CLI via an endpoint on the Apache Airflow Webserver\. The generated token is valid for only 60 seconds after creation\.

The following example POST request demonstrates how to access the Apache Airflow Webserver using curl:

```
CLI_JSON=$(aws mwaa create-cli-token --name MyMWAAEnvironment) \
  && CLI_TOKEN=$(echo $CLI_JSON | jq -r '.CliToken') \
  && WEB_SERVER_HOSTNAME=$(echo $CLI_JSON | jq -r '.WebServerHostname') \
  && CLI_RESULTS=$(curl --request POST "https://$WEB_SERVER_HOSTNAME/aws_mwaa/cli" \
  --header "Authorization: Bearer $CLI_TOKEN" \
  --header "Content-Type: text/plain" \
  --data-raw "trigger_dag my-dag-name") \
  && echo "Output:" \
  && echo $CLI_RESULTS | jq -r '.stdout' | base64 --decode \
  && echo "Errors:" \
  && echo $CLI_RESULTS | jq -r '.stderr' | base64 --decode
```

The response of a successful POST call contains the following JSON object:

**Response**

```
{
    "stderr":"<STDERR of the CLI execution (if any), base64 encoded>",
    "stdout":"<STDOUT of the CLI execution, base64 encoded>"
}
```

The following example demonstrates an additional method to create a CLI token:

```
# brew install jq
aws mwaa create-cli-token --name MWAAEnvironmentName | export CLI_TOKEN=$(jq -r .CliToken) && curl --request POST "https://$WEB_SERVER_HOSTNAME/aws_mwaa/cli" \
    --header "Authorization: Bearer $CLI_TOKEN" \
    --header "Content-Type: text/plain" \
    --data-raw "trigger_dag DAGName"
```

## Apache Airflow CLI command reference<a name="airflow-cli-commands-supported"></a>

The following section describes the supported and unsupported Apache Airflow CLI commands\.

### Supported commands<a name="airflow-cli-commands-supported"></a>

The following [Apache Airflow CLI commands](https://airflow.apache.org/docs/apache-airflow/stable/cli-and-env-variables-ref.html) are supported when using Apache Airflow in an Amazon MWAA environment:
+ clear
+ dag\_state
+ delete\_dag
+ list\_dag\_runs
+ list\_tasks
+ next\_execution
+ pause
+ pool
+ render
+ run
+ show\_dag
+ task\_failed\_deps
+ task\_state
+ test
+ trigger\_dag
+ unpause
+ variables
+ version

### Unsupported commands<a name="airflow-unsupported-cli-commands"></a>

The following Apache Airflow CLI commands are not supported when running Apache Airflow in an Amazon MWAA environment\.
+ backfill
+  checkdb
+ connections
+ create\_user
+ delete\_user
+ flower
+ initdb
+ kerberos
+ list\_dags
+ list\_users
+ resetdb
+ rotate\_fernet\_key
+ scheduler
+ serve\_logs
+ shell
+ sync\_perm
+ upgradedb
+ webserver
+ worker