# Apache Airflow versions on Amazon Managed Workflows for Apache Airflow \(MWAA\)<a name="airflow-versions"></a>

This page describes the Apache Airflow versions Amazon Managed Workflows for Apache Airflow \(MWAA\) supports and the strategies we recommend to upgrade to the latest version\.

**Contents**
+ [About Amazon MWAA versions](#airflow-versions-image)
+ [Latest version](#airflow-versions-latest)
+ [Apache Airflow versions](#airflow-versions-official)
+ [Apache Airflow components](#airflow-versions-components)
  + [Schedulers](#airflow-versions-components-schedulers)
  + [Workers](#airflow-versions-components-workers)
+ [Apache Airflow v2\.2\.2](#airflow-versions-v222)
  + [What's supported for Apache Airflow v2\.2\.2](#airflow-versions-what-supported-222)
  + [Installing Apache Airflow v2\.2\.2](#airflow-versions-installing-222)
+ [Apache Airflow v2\.0\.2](#airflow-versions-v202)
  + [What's supported for Apache Airflow v2\.0\.2](#airflow-versions-what-supported-202)
  + [Installing Apache Airflow v2\.0\.2](#airflow-versions-installing-202)
+ [Apache Airflow v1\.10\.12](#airflow-versions-v11012)
  + [Installing Apache Airflow v1\.10\.12](#airflow-versions-installing-11012)
+ [Upgrading Apache Airflow](#airflow-versions-upgrading)
  + [Upgrading from Apache Airflow v2\.0\.2 to Apache Airflow v2\.2\.2](#v202-to-v222)
  + [Upgrading from Apache Airflow v1 to Apache Airflow v2](#v1-to-v2)
+ [Changes between v1 and v2](#airflow-versions-what-changed-202)

## About Amazon MWAA versions<a name="airflow-versions-image"></a>

Amazon MWAA builds container images that bundle Apache Airflow releases with other common binaries and Python libraries\. The image uses the Apache Airflow base install for the version you specify\. When you create an environment, you specify an image version to use\. Once an environment is created, it keeps using the specified image version until you upgrade it to a later version\.

## Latest version<a name="airflow-versions-latest"></a>

Amazon MWAA supports more than one Apache Airflow version\. If you do not specify an image version when you create an environment, the latest version is used\.

## Apache Airflow versions<a name="airflow-versions-official"></a>

The following Apache Airflow versions are supported on Amazon Managed Workflows for Apache Airflow \(MWAA\)\.


| Apache Airflow version | Apache Airflow guide | Apache Airflow constraints | Python version | 
| --- | --- | --- | --- | 
|  v2\.2\.2  |  [Apache Airflow v2\.2\.2 reference guide](https://airflow.apache.org/docs/apache-airflow/2.2.2/index.html)  |  [Apache Airflow v2\.2\.2 constraints file](https://raw.githubusercontent.com/apache/airflow/constraints-2.2.2/constraints-3.7.txt)  |  [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)  | 
|  v2\.0\.2  |  [Apache Airflow v2\.0\.2 reference guide](http://airflow.apache.org/docs/apache-airflow/2.0.2/index.html)  |  [Apache Airflow v2\.0\.2 constraints file](https://raw.githubusercontent.com/apache/airflow/constraints-2.0.2/constraints-3.7.txt)  |  [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)  | 
|  v1\.10\.12  |  [Apache Airflow v1\.10\.12 reference guide](https://airflow.apache.org/docs/apache-airflow/1.10.12/)  |  [Apache Airflow v1\.10\.12 constraints file](https://raw.githubusercontent.com/apache/airflow/constraints-1.10.12/constraints-3.7.txt)  |  [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)  | 

## Apache Airflow components<a name="airflow-versions-components"></a>

This section describes the number of Apache Airflow *Scheduler\(s\)* and *Workers* available for each Apache Airflow version on Amazon MWAA\.

### Schedulers<a name="airflow-versions-components-schedulers"></a>


| Apache Airflow version | Scheduler \(default\) | Scheduler \(min\) | Scheduler \(max\) | 
| --- | --- | --- | --- | 
|  Apache Airflow v2 and above  |  2  |  2  |  5  | 
|  Apache Airflow v1  |  1  |  1  |  1  | 

### Workers<a name="airflow-versions-components-workers"></a>


| Airflow version | Workers \(min\) | Workers \(max\) | Workers \(default\) | 
| --- | --- | --- | --- | 
|  Apache Airflow v2  |  1  |  25  |  10  | 
|  Apache Airflow v1  |  1  |  25  |  10  | 

## Apache Airflow v2\.2\.2<a name="airflow-versions-v222"></a>

This section provides an overview of what's supported on Amazon MWAA, what's changed in Apache Airflow v2\.2\.2, and links to install Apache Airflow v2\.2\.2 in the *Apache Airflow reference guide*\.

### What's supported for Apache Airflow v2\.2\.2<a name="airflow-versions-what-supported-222"></a>

Amazon MWAA supports all the features outlined for [Apache Airflow v2\.0\.2](#airflow-versions-what-supported-202)\. In addition, beginning with Apache Airflow v2, Amazon MWAA will install Python requirements, provider packages, and custom plugins on the Apache Airflow web server\.

 For more information on new features in this version, see the [Apache Airflow Changelog](https://airflow.apache.org/docs/apache-airflow/2.2.0/changelog.html) on the Apache Airflow documentation website\. 

### Installing Apache Airflow v2\.2\.2<a name="airflow-versions-installing-222"></a>

The following section contains links to tutorials in the *Apache Airflow reference guide* to install and run Apache Airflow v2\.2\.2\. The steps assume you are starting from scratch and have the [Docker Engine](https://docs.docker.com/engine/installation/) and [Docker Compose](https://docs.docker.com/compose/install/) installed locally\.

To install Apache Airflow v2\.2\.2 in Docker, see [Running Airflow in Docker](https://airflow.apache.org/docs/apache-airflow/2.2.2/start/docker.html) in the *Apache Airflow reference guide*\.

## Apache Airflow v2\.0\.2<a name="airflow-versions-v202"></a>

This section provides an overview of what's supported on Amazon MWAA, what's changed between Apache Airflow v1\.10\.12 and Apache Airflow v2\.0\.2, and links to install or upgrade to Apache Airflow v2\.0\.2 in the *Apache Airflow reference guide*\.

### What's supported for Apache Airflow v2\.0\.2<a name="airflow-versions-what-supported-202"></a>


| Apache Airflow feature | Supported | Description | Apache Airflow guide | 
| --- | --- | --- | --- | 
|  High Availability  |  Yes  |  Allows you to run multiple *Schedulers* concurrently\.  |  [Running More Than One Scheduler](https://airflow.apache.org/docs/apache-airflow/stable/scheduler.html#running-more-than-one-scheduler)  | 
|  Full REST API  |  No  |  Allows you and third\-parties to acccess CRUD \(Create, Read, Update, Delete\) operations on all Airflow resources using a Swagger/OpenAPI specification\.  |  [Airflow's REST API](https://airflow.apache.org/docs/apache-airflow/stable/stable-rest-api-ref.html)  | 
|  Smart Sensors  |  Yes  |  Allows you to schedule a single and long running task, checks the status of a batch of *Sensor* tasks, and stores in your metadata database\.  |  [Smart Sensors](https://airflow.apache.org/docs/apache-airflow/stable/smart-sensor.html)  | 
|  TaskFlow API  |  Yes  |  Allows you to organize tasks into hierarchical groups and pass/share data between tasks\.  |  [TaskFlow API](https://airflow.apache.org/docs/apache-airflow/stable/concepts.html#taskflow-api)  | 
|  Task Groups  |  Yes  |  Allows you to view task groups in the Apache Airflow UI \(a UI grouping concept which fulfills the primary purpose of SubDAGs\)\.  |  [TaskGroup](https://airflow.apache.org/docs/apache-airflow/stable/concepts.html#taskgroup)  | 
|  Independent Providers  |  Yes  |  Allows you to use Airflow packages that have been separated from and independently versioned from the core Apache Airflow distribution\. \(Bash and Python Operators remain in the core distribution\.\)  |  [Provider Packages](https://airflow.apache.org/docs/apache-airflow-providers/index.html)  | 
|  Simplified Kubernetes Executor  |  No  |  Allows you to use a re\-architected Kubernetes Executor and KubernetesPodExecutor \(which allow users to dynamically launch tasks as individual Kubernetes Pods\)\.  |  [Kubernetes Executor](https://airflow.apache.org/docs/apache-airflow/stable/executor/kubernetes.html)  | 
|  UI/UX Improvements  |  Yes  |  Allows you to use a more intuitive front\-end experience to the Apache Airflow UI\.  |  [Airflow UI](https://airflow.apache.org/docs/apache-airflow/stable/ui.html)  | 
|  Airflow connections on Amazon MWAA  |  Yes  |  Allows you to use the Apache Airflow `connections add` and `connections delete` CLI commands\. To learn more, see [Apache Airflow CLI command reference](airflow-cli-command-reference.md)\.  |  [Command Line Interface](http://airflow.apache.org/docs/apache-airflow/2.0.2/cli-and-env-variables-ref.html#command-line-interface)  | 
|  DAG decorators  |  Yes  |  Allows you to decorate a function with @dag or @task to to turn it into a DAG generator function or into a task instance using a PythonOperator\.   |  [The DAG decorator](https://airflow.apache.org/docs/apache-airflow/stable/concepts/dags.html#the-dag-decorator)  | 

### Installing Apache Airflow v2\.0\.2<a name="airflow-versions-installing-202"></a>

The following section contains links to tutorials in the *Apache Airflow reference guide* to install and run Apache Airflow v2\.0\.2\. The steps assume you are starting from scratch and have the [Docker Engine](https://docs.docker.com/engine/installation/) and [Docker Compose](https://docs.docker.com/compose/install/) installed locally\. 

To install Apache Airflow v2\.0\.2 in Docker, see [Running Airflow in Docker](https://airflow.apache.org/docs/apache-airflow/2.0.2/start/docker.html) in the *Apache Airflow reference guide*\.

## Apache Airflow v1\.10\.12<a name="airflow-versions-v11012"></a>

This section provides an overview of how to get started with Apache Airflow v1\.10\.12, and links to install Apache Airflow v1\.10\.12 in the *Apache Airflow reference guide*\.

### Installing Apache Airflow v1\.10\.12<a name="airflow-versions-installing-11012"></a>

The following section contains links to tutorials in the *Apache Airflow reference guide* to install and run Apache Airflow v1\.10\.12\. To install Apache Airflow v1\.10\.12, see [Installation for Apache Airflow v1\.10\.12](https://airflow.apache.org/docs/apache-airflow/1.10.12/installation.html) in the *Apache Airflow reference guide*\.

## Upgrading Apache Airflow<a name="airflow-versions-upgrading"></a>

The following section contains links to tutorials in the *Apache Airflow reference guide* and the steps we recommend to upgrade your environment\.

**Important**  
 Currently, Amazon MWAA does not support in\-place upgrades of existing environments for any Apache Airflow versions\. To upgrade your Amazon MWAA environment to the latest supported release, you have to create a new environment and migrate your resources to the new environment\. 

**Topics**
+ [Upgrading from Apache Airflow v2\.0\.2 to Apache Airflow v2\.2\.2](#v202-to-v222)
+ [Upgrading from Apache Airflow v1 to Apache Airflow v2](#v1-to-v2)

### Upgrading from Apache Airflow v2\.0\.2 to Apache Airflow v2\.2\.2<a name="v202-to-v222"></a>

 The following section shows how you can upgrade your Apache Airflow v2\.0\.2 environment to Apache Airflow v2\.2\.2\. 

1.  Create a new Apache Airflow v2\.2\.2 environment\. 

1.  Copy your DAGs, custom plugins, and `requirements.txt` resources from your existing v2\.0\.2 Amazon S3 bucket to the new environment's Amazon S3 bucket\. 

   1.  If you use `requirements.txt` in your environment, you'll need to update the `--constraint` to [v2\.2\.2 constraints](https://raw.githubusercontent.com/apache/airflow/constraints-2.2.2/constraints-3.7.txt) and verify that current libraries and packages are compatible with Apache Airflow v2\.2\.2 

1.  Starting with Apache Airflow v2\.2\.2, the list of provider packages Amazon MWAA installs by default for your environment has changed, and you can now install dependencies and plugin on the Apache Airflow web server\. Compare the [list of provider packages installed by default](connections-packages.md) in Apache Airflow v2\.2\.2 and Apache Airflow v2\.0\.2, and configure any additional packages you might need for your new v2\.2\.2 environment\. 

1.  Test your DAGs using the new v2\.2\.2 environment\. 

1.  Once you have confirmed that your tasks complete successfully, delete the v2\.0\.2 environment\. 

### Upgrading from Apache Airflow v1 to Apache Airflow v2<a name="v1-to-v2"></a>

 The following section shows how you can upgrade your Apache Airflow v1 environment to Apache Airflow v2\. For more information about migrating your self\-managed Apache Airflow deployments, or migrating an existing Amazon MWAA environment, including instructions for backing up your metadata database, see the [Amazon MWAA Migration Guide](https://docs.aws.amazon.com/https://docs.aws.amazon.com/mwaa/latest/migrationguide/index.html)\. 

 The steps in this section assume you are starting with an existing codebase in Apache Airflow v1\.10\.12 and are upgrading it to any of the supported Apache Airflow v2 minor version releases\.To upgrade to Apache Airflow v2, create a new environment using the latest supported version with the required [Apache Airflow configuration options](configuring-env-variables.md) from your existing environment, and follow the steps in this section to modify your DAGs, Python dependencies in `requirements.txt`, and custom plugins in `plugins.zip` for the new version\. After you have successfully migrated your workflows,you can delete the older environment\.

1. Follow the steps to upgrade to the Apache Airflow v1 bridge release, run upgrade check scripts, and convert and test custom plugins and DAGs to Apache Airflow v2 locally, in [Upgrading from 1\.10 to 2](https://airflow.apache.org/docs/apache-airflow/2.2.2/upgrading-from-1-10/index.html) in the *Apache Airflow reference guide* and [Updating Airflow](https://github.com/apache/airflow/blob/master/UPDATING.md) on GitHub\.

   1. **Upgrade import statements**\. Get started by reviewing the Apache Airflow v2 sample code in [Using a secret key in AWS Secrets Manager for an Apache Airflow connection](samples-secrets-manager.md)— the import statements and [hooks](https://airflow.apache.org/docs/apache-airflow/2.2.2/_api/airflow/hooks/index.html) from Apache Airflow v1 to Apache Airflow v2 have changed\.

      1.   
**Example Apache Airflow v1**  

         ```
         from airflow.operators.python_operator import PythonOperator
         from airflow.contrib.hooks.aws_hook import AwsHook
             ...
         hook = AwsHook()
         ```

      1.   
**Example Apache Airflow v2**  

         ```
         from airflow.operators.python import PythonOperator
         from airflow.providers.amazon.aws.hooks.base_aws import AwsBaseHook
             ...
         hook = AwsBaseHook(client_type='secretsmanager')
         ```

   1. **Upgrade Apache Airflow CLI scripts**\. Get started by reviewing the Apache Airflow v2 CLI commands in [Using a bash script](call-mwaa-apis-cli.md#create-cli-token-bash)— Related commands are now grouped together as subcommands\. For example, `trigger_dag` in Apache Airflow v1 is now `dags trigger` Apache Airflow v2\.

      1.   
**Example Apache Airflow v1**  

         ```
         # brew install jq
         aws mwaa create-cli-token --name YOUR_ENVIRONMENT_NAME | export CLI_TOKEN=$(jq -r .CliToken) && curl --request POST "https://YOUR_HOST_NAME/aws_mwaa/cli" \
             --header "Authorization: Bearer $CLI_TOKEN" \
             --header "Content-Type: text/plain" \
             --data-raw "trigger_dag YOUR_DAG_NAME"
         ```

      1.   
**Example Apache Airflow v2**  

         ```
         # brew install jq
         aws mwaa create-cli-token --name YOUR_ENVIRONMENT_NAME | export CLI_TOKEN=$(jq -r .CliToken) && curl --request POST "https://YOUR_HOST_NAME/aws_mwaa/cli" \
             --header "Authorization: Bearer $CLI_TOKEN" \
             --header "Content-Type: text/plain" \
             --data-raw "dags trigger YOUR_DAG_NAME"
         ```

1. [Create an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment](mwaa-autoscaling.md) using the latest supported version and iteratively add DAGs, custom plugins in `plugins.zip`, and Python dependencies in `requirements.txt` to your new environment as you finish testing locally\. 

   1. Get started with the Apache Airflow v2 sample code in [Aurora PostgreSQL database cleanup on an Amazon MWAA environment](samples-database-cleanup.md)— The import statements in Apache Airflow v1 to Apache Airflow v2 and the Python dependencies in the `requirements.txt` have changed\.

     1.   
**Example Apache Airflow v1 dependencies**  

        ```
        apache-airflow[postgres]==1.10.12
        apache-airflow[mysql]==1.10.12
        ```

     1.   
**Example Apache Airflow v2 dependencies**  

        ```
        apache-airflow[postgres]==2.0.2
        apache-airflow[mysql]==2.0.2
        ```

     1.   
**Example Apache Airflow v1 imports**  

        ```
        from airflow.operators.python_operator import PythonOperator
        from airflow.jobs import BaseJob
        ```

     1.   
**Example Apache Airflow v2 imports**  

        ```
        from airflow.operators.python import PythonOperator
        from airflow.jobs.base_job import BaseJob
        ```

1. After you've migrated and tested all DAGs, custom plugins in `plugins.zip`, and Python dependencies in `requirements.txt` to your new new environment, you can delete your old environment\.

## Changes between v1 and v2<a name="airflow-versions-what-changed-202"></a>

The following is a summary of what's changed between Apache Airflow v1 and Apache Airflow v2 on Amazon MWAA\.
+ **New: High availability by default**\. By default, an Amazon MWAA environment in Apache Airflow v2 uses 2 Airflow *Schedulers*, and accepts a value up to 5 *Schedulers*\. To learn more about running more than one scheduler concurrently, see [Airflow Scheduler](https://airflow.apache.org/docs/apache-airflow/stable/scheduler.html?highlight=multiple%20schedulers#running-more-than-one-scheduler) in the *Apache Airflow reference guide*\.
+ **New: Airflow package extras**\. The Python dependencies that you specify in a `requirements.txt` on Amazon MWAA have changed in Apache Airflow v2\. For example, the [core extras](http://airflow.apache.org/docs/apache-airflow/2.2.2/extra-packages-ref.html#core-airflow-extras), [provider extras](http://airflow.apache.org/docs/apache-airflow/2.2.2/extra-packages-ref.html#providers-extras), [locally installed software extras](http://airflow.apache.org/docs/apache-airflow/2.2.2/extra-packages-ref.html#locally-installed-software-extras), [external service extras](http://airflow.apache.org/docs/apache-airflow/2.2.2/extra-packages-ref.html#external-services-extras), ["other" extras](http://airflow.apache.org/docs/apache-airflow/2.2.2/extra-packages-ref.html#other-extras), [bundle extras](http://airflow.apache.org/docs/apache-airflow/2.2.2/extra-packages-ref.html#bundle-extras), [doc extras](http://airflow.apache.org/docs/apache-airflow/2.2.2/extra-packages-ref.html#doc-extras), and [software extras](http://airflow.apache.org/docs/apache-airflow/2.2.2/extra-packages-ref.html#apache-software-extras) have changed\. To view a list of the packages installed for Apache Airflow v2 on Amazon MWAA, see [Amazon MWAA local runner `requirements.txt`](https://github.com/aws/aws-mwaa-local-runner/blob/main/docker/config/requirements.txt) on the GitHub website\.
+ **New: Operators, Hooks, and Executors**\. The import statements in your DAGs, and the custom plugins you specify in a `plugins.zip` on Amazon MWAA have changed between Apache Airflow v1 and Apache Airflow v2\. For example, `from airflow.contrib.hooks.aws_hook import AwsHook` in Apache Airflow v1 has changed to `from airflow.providers.amazon.aws.hooks.base_aws import AwsBaseHook` in Apache Airflow v2\. To learn more, see [Python API Reference](https://airflow.apache.org/docs/apache-airflow/2.2.2/python-api-ref.html) in the *Apache Airflow reference guide*\.
+ **New: Imports in plugins**\. Importing operators, sensors, hooks added in plugins using `airflow.{operators,sensors,hooks}.<plugin_name>` is no longer supported\. These extensions should be imported as regular Python modules\. In v2 and above, the recommended approach is to place them in the DAGs directory and create and use an *\.airflowignore* file to exclude them from being parsed as DAGs\. To learn more, see [Modules Management](https://airflow.apache.org/docs/apache-airflow/stable/modules_management.html) and [Creating a custom Operator](https://airflow.apache.org/docs/apache-airflow/stable/howto/custom-operator.html) in the *Apache Airflow reference guide*\.
+ **New: Airflow CLI command structure**\. The Apache Airflow v2 CLI is organized so that related commands are grouped together as subcommands, which means you need to update Apache Airflow v1 scripts if you want to upgrade to Apache Airflow v2\. For example, `unpause` in Apache Airflow v1 is now `dags unpause` in Apache Airflow v2\. To learn more, see [Airflow CLI changes in 2](http://airflow.apache.org/docs/apache-airflow/2.0.2/upgrading-to-2.html#airflow-cli-changes-in-2-0) in the *Apache Airflow reference guide*\.
+ **Changed: Airflow connection types**\. By default, the Airflow UI contains a subset of the connection types that were available in Apache Airflow v1\. To view a list of the connection types available for Apache Airflow v2 on Amazon MWAA by default, see [Apache Airflow v2 provider packages installed on Amazon MWAA environments](connections-packages.md)\. 
+ **Existing: Airflow configuration options**\. The Apache Airflow v2 configuration options are the **same** as Apache Airflow v1\. Although, in version 2 and above, Apache Airflow requires explicit specifications of configuration values in some cases, rather than defaulting to a generic value\. To learn more, see [ Airflow configuration options ](http://airflow.apache.org/docs/apache-airflow/2.0.2/upgrading-to-2.html#step-6-upgrade-configuration-settings) in the *Apache Airflow reference guide*\.