# Apache Airflow CLI command reference<a name="airflow-cli-command-reference"></a>

This page describes the supported and unsupported Apache Airflow CLI commands on Amazon Managed Workflows for Apache Airflow \(MWAA\)\.

**Contents**
+ [Supported commands](#airflow-cli-commands-supported)
+ [Unsupported commands](#airflow-unsupported-cli-commands)
+ [Using commands that parse DAGs](#parsing-support)
+ [Example code to add a configuration when triggering a DAG](#example-airflow-cli-commands-trigger)
+ [Samples in GitHub and AWS tutorials](#airflow-cli-commands-tutorials)

## Supported commands<a name="airflow-cli-commands-supported"></a>

The following list shows the Apache Airflow CLI commands available on Amazon MWAA\. 

------
#### [ Airflow v2\.0\.2 ]


| Airflow version | Supported | Command | 
| --- | --- | --- | 
|  v2\.0\.2  |  Yes  |  [cheat\-sheet](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#cheat-sheet)  | 
|  v2\.0\.2  |  Yes  |  [dags delete](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#delete_repeat1)  | 
|  v2\.0\.2  |  Yes  |  [dags list\-jobs](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#list-jobs)  | 
|  v2\.0\.2  |  Yes  |  [dags list\-runs](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#list-runs)  | 
|  v2\.0\.2  |  Yes  |  [dags next\-execution](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#next-execution)  | 
|  v2\.0\.2  |  Yes  |  [dags pause](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#pause)  | 
|  v2\.0\.2  |  Yes  |  [dags report](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#report)  | 
|  v2\.0\.2  |  Yes  |  [dags show](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#show)  | 
|  v2\.0\.2  |  Yes  |  [dags state](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#state)  | 
|  v2\.0\.2  |  Yes  |  [dags test](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#test)  | 
|  v2\.0\.2  |  Yes  |  [dags trigger](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#trigger)  | 
|  v2\.0\.2  |  Yes  |  [dags unpause](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#unpause)  | 
|  v2\.0\.2  |  Yes  |  [providers behaviours](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#behaviours)  | 
|  v2\.0\.2  |  Yes  |  [providers get](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#get_repeat2)  | 
|  v2\.0\.2  |  Yes  |  [providers hooks](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#hooks)  | 
|  v2\.0\.2  |  Yes  |  [providers links](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#links)  | 
|  v2\.0\.2  |  Yes  |  [providers list](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#list_repeat4)  | 
|  v2\.0\.2  |  Yes  |  [providers widgets](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#widgets)  | 
|  v2\.0\.2  |  Yes  |  [roles create](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#create)  | 
|  v2\.0\.2  |  Yes  |  [roles list](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#list_repeat5)  | 
|  v2\.0\.2  |  Yes  |  [tasks clear](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#clear)  | 
|  v2\.0\.2  |  Yes  |  [tasks failed\-deps](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#failed-deps)  | 
|  v2\.0\.2  |  Yes  |  [tasks list](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#list_repeat6)  | 
|  v2\.0\.2  |  Yes  |  [tasks render](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#render)  | 
|  v2\.0\.2  |  Yes  |  [tasks state](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#state_repeat1)  | 
|  v2\.0\.2  |  Yes  |  [tasks states\-for\-dag\-run](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#states-for-dag-run)  | 
|  v2\.0\.2  |  Yes  |  [tasks test](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#test_repeat1)  | 
|  v2\.0\.2  |  Yes  |  [variables delete](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#delete_repeat4)  | 
|  v2\.0\.2  |  Yes  |  [variables get](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#get_repeat3)  | 
|  v2\.0\.2  |  Yes  |  [variables list](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#list_repeat8)  | 
|  v2\.0\.2  |  Yes  |  [variables set](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#set_repeat1)  | 
|  v2\.0\.2  |  Yes  |  [version](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#version)  | 

------
#### [ Airflow v1\.10\.12 ]


| Airflow version | Supported | Command | 
| --- | --- | --- | 
|  v1\.10\.12  |  Yes  |  [clear](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#clear)  | 
|  v1\.10\.12  |  Yes  |  [delete\_dag](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#delete_dag)  | 
|  v1\.10\.12  |  Yes  |  [next\_execution](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#next_execution)  | 
|  v1\.10\.12  |  Yes  |  [pause](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#pause)  | 
|  v1\.10\.12  |  Yes  |  [pool](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#pool)  | 
|  v1\.10\.12  |  Yes  |  [render](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#render)  | 
|  v1\.10\.12  |  Yes  |  [run](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#run)  | 
|  v1\.10\.12  |  Yes  |  [task\_failed\_deps](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#task_failed_deps)  | 
|  v1\.10\.12  |  Yes  |  [trigger\_dag](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#trigger_dag)  | 
|  v1\.10\.12  |  Yes  |  [unpause](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#unpause)  | 
|  v1\.10\.12  |  Yes  |  [variables](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#variables)  | 
|  v1\.10\.12  |  Yes  |  [version](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#version)  | 

------

## Unsupported commands<a name="airflow-unsupported-cli-commands"></a>

The following list shows the Apache Airflow CLI commands **not** available on Amazon MWAA\. 

------
#### [ Airflow v2\.0\.2 ]


| Airflow version | Supported | Command | 
| --- | --- | --- | 
|  v2\.0\.2  |  No  |  [connections](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#connections)  | 
|  v2\.0\.2  |  \*No \([note](#parsing-support)\)  |  [dags backfill](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#backfill)  | 
|  v2\.0\.2  |  \*No \([note](#parsing-support)\)  |  [dags list](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#list_repeat2)  | 
|  v2\.0\.2  |  No  |  [dags show **\-\-save**](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#show)  | 
|  v2\.0\.2  |  No  |  [dags test **\-\-save\-dagrun**](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#test)  | 
|  v2\.0\.2  |  No  |  [dags test **\-\-show\-dagrun**](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#test)  | 
|  v2\.0\.2  |  No  |  [db check](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#check)  | 
|  v2\.0\.2  |  No  |  [db check\-migrations](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#check-migrations)  | 
|  v2\.0\.2  |  No  |  [db init](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#init)  | 
|  v2\.0\.2  |  No  |  [db reset](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#reset)  | 
|  v2\.0\.2  |  No  |  [db upgrade](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#upgrade)  | 
|  v2\.0\.2  |  No  |  [flower](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#flower)  | 
|  v2\.0\.2  |  No  |  [kerberos](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#kerberos)  | 
|  v2\.0\.2  |  No  |  [pools delete](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#delete_repeat2)  | 
|  v2\.0\.2  |  No  |  [pools list](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#list_repeat3)  | 
|  v2\.0\.2  |  No  |  [pools export](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#export_repeat1)  | 
|  v2\.0\.2  |  No  |  [pools set](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#set)  | 
|  v2\.0\.2  |  No  |  [rotate\-fernet\-key](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#rotate-fernet-key)  | 
|  v2\.0\.2  |  No  |  [scheduler](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#scheduler)  | 
|  v2\.0\.2  |  No  |  [shell](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#shell)  | 
|  v2\.0\.2  |  No  |  [sync\-perm](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#sync-perm)  | 
|  v2\.0\.2  |  No  |  [tasks run](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#run)  | 
|  v2\.0\.2  |  No  |  [users create](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#create_repeat1)  | 
|  v2\.0\.2  |  No  |  [users delete](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#delete_repeat3)  | 
|  v2\.0\.2  |  No  |  [users list](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#list_repeat7)  | 
|  v2\.0\.2  |  No  |  [webserver](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#webserver)  | 
|  v2\.0\.2  |  No  |  [worker](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#worker)  | 

------
#### [ Airflow v1\.10\.12 ]


| Airflow version | Supported | Command | 
| --- | --- | --- | 
|  v1\.10\.12  |  \*No \([note](#parsing-support)\)  |  [backfill](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#backfill)  | 
|  v1\.10\.12  |  No  |  [checkdb](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#checkdb)  | 
|  v1\.10\.12  |  No  |  [connections](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#connections)  | 
|  v1\.10\.12  |  No  |  [create\_user](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#create_user)  | 
|  v1\.10\.12  |  \*No \([note](#parsing-support)\)  |  [dag\_state](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#dag_state)  | 
|  v1\.10\.12  |  No  |  [delete\_user](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#delete_user)  | 
|  v1\.10\.12  |  No  |  [flower](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#flower)  | 
|  v1\.10\.12  |  No  |  [initdb](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#initdb)  | 
|  v1\.10\.12  |  No  |  [kerberos](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#kerberos)  | 
|  v1\.10\.12  |  \*No \([note](#parsing-support)\)  |  [list\_dag\_runs](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#list_dag_runs)  | 
|  v1\.10\.12  |  \*No \([note](#parsing-support)\)  |  [list\_dags](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#list_dags)  | 
|  v1\.10\.12  |  No  |  [list\_users](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#list_users)  | 
|  v1\.10\.12  |  \*No \([note](#parsing-support)\)  |  [list\_tasks](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#list_tasks)  | 
|  v1\.10\.12  |  No  |  [resetdb](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#resetdb)  | 
|  v1\.10\.12  |  No  |  [rotate\_fernet\_key](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#rotate_fernet_key)  | 
|  v1\.10\.12  |  No  |  [scheduler](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#scheduler)  | 
|  v1\.10\.12  |  No  |  [serve\_logs](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#serve_logs)  | 
|  v1\.10\.12  |  No  |  [shell](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#shell)  | 
|  v1\.10\.12  |  \*No \([note](#parsing-support)\)  |  [show\_dag](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#show_dag)  | 
|  v1\.10\.12  |  No  |  [sync\_perm](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#sync_perm)  | 
|  v1\.10\.12  |  \*No \([note](#parsing-support)\)  |  [task\_state](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#task_state)  | 
|  v1\.10\.12  |  \*No \([note](#parsing-support)\)  |  [test](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#test)  | 
|  v1\.10\.12  |  No  |  [upgradedb](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#upgradedb)  | 
|  v1\.10\.12  |  No  |  [webserver](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#webserver)  | 
|  v1\.10\.12  |  No  |  [worker](http://airflow.apache.org/docs/apache-airflow/1.10.12/cli-ref.html#worker)  | 

------

## Using commands that parse DAGs<a name="parsing-support"></a>

Apache Airflow CLI commands that parse DAGs will fail if the DAG uses plugins that depend on packages installed through a `requirements.txt`:
+ `backfill`
+ `dag_state`
+ `list_dag_runs`
+ `list_dags`
+ `list_tasks`
+ `show_dag`
+ `task_state`
+ `test`

You can use these CLI commands if your DAGS don't use plugins that depend on packages installed through a `requirements.txt`\.

## Example code to add a configuration when triggering a DAG<a name="example-airflow-cli-commands-trigger"></a>

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

## Samples in GitHub and AWS tutorials<a name="airflow-cli-commands-tutorials"></a>
+ [Interacting with Apache Airflow v1\.10\.12 on Amazon MWAA via the command line](https://dev.to/aws/interacting-with-amazon-managed-workflows-for-apache-airflow-via-the-command-line-4e91)
+ [Interactive Commands with Apache Airflow v1\.10\.12 on Amazon MWAA and Bash Operator](https://github.com/aws-samples/amazon-mwaa-examples/tree/main/dags/bash_operator_script) *on GitHub*