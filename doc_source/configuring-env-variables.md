# Apache Airflow configuration options<a name="configuring-env-variables"></a>

Apache Airflow configuration options can be attached to your Amazon Managed Workflows for Apache Airflow \(MWAA\) environment as environment variables\. You can choose from the suggested dropdown list, or specify custom configuration options for your Apache Airflow version on the Amazon MWAA console\. This page describes the Apache Airflow configuration options available, and how to use these options to override Apache Airflow configuration settings on your environment\.

**Contents**
+ [Prerequisites](#configuring-env-variables-prereqs)
+ [How it works](#configuring-env-variables-how)
+ [Using configuration options to load plugins in Apache Airflow v2](#configuring-2.0-airflow-override)
+ [Configuration options overview](#configuring-env-variables-customizing)
  + [Apache Airflow configuration options](#configuring-env-variables-airflow-ref)
  + [Apache Airflow reference](#configuring-env-variables-reference-options)
  + [Using the Amazon MWAA console](#configuring-env-variables-console-add)
+ [Configuration reference](#configuring-env-variables-reference)
  + [Email configurations](#configuring-env-variables-email)
  + [Task configurations](#configuring-env-variables-tasks)
  + [Scheduler configurations](#configuring-env-variables-scheduler)
  + [Worker configurations](#configuring-env-variables-workers)
  + [Web server configurations](#configuring-env-variables-webserver)
+ [Examples and sample code](#configuring-env-variables-code)
  + [Example DAG](#configuring-env-variables-dag)
  + [Example email notification settings](#configuring-env-variables-email)
+ [What's next?](#configuring-env-variables-next-up)

## Prerequisites<a name="configuring-env-variables-prereqs"></a>

You'll need the following before you can complete the steps on this page\.

1. **Access**\. Your AWS account must have been granted access by your administrator to the [AmazonMWAAFullConsoleAccess](access-policies.md#console-full-access) access control policy for your environment\.

1. **Amazon S3 configurations**\. The [Amazon S3 bucket](mwaa-s3-bucket.md) used to store your DAGs, custom plugins in `plugins.zip`, and Python dependencies in `requirements.txt` must be configured with *Public Access Blocked* and *Versioning Enabled*\.

1. **Permissions**\. Your Amazon MWAA environment must be permitted by your [execution role](mwaa-create-role.md) to access the AWS resources used by your environment\.

## How it works<a name="configuring-env-variables-how"></a>

When you create an environment, Amazon MWAA attaches the configuration settings you specify on the Amazon MWAA console in **Airflow configuration options** as environment variables to the AWS Fargate container for your environment\. If you're using a setting of the same name in `airflow.cfg`, the options you specify on the Amazon MWAA console override the values in `airflow.cfg`\.

While we don't expose the `airflow.cfg` in the Apache Airflow UI of an Amazon MWAA environment, you can change the Apache Airflow configuration options directly on the Amazon MWAA console and continue using all other settings in `airflow.cfg`\.

## Using configuration options to load plugins in Apache Airflow v2<a name="configuring-2.0-airflow-override"></a>

By default in Apache Airflow v2, plugins are configured to be "lazily" loaded using the `core.lazy_load_plugins : True` setting\. If you're using custom plugins in Apache Airflow v2, you must add `core.lazy_load_plugins : False` as an Apache Airflow configuration option to load plugins at the start of each Airflow process to override the default setting\.

## Configuration options overview<a name="configuring-env-variables-customizing"></a>

When you add a configuration on the Amazon MWAA console, Amazon MWAA writes the configuration as an environment variable\. 
+ **Listed options**\. You can choose from one of the configuration settings available for your Apache Airflow version in the dropdown list\. For example, `dag_concurrency` : `16`\. The configuration setting is translated to your environment's Fargate container as `AIRFLOW__CORE__DAG_CONCURRENCY : 16`
+ **Custom options**\. You can also specify Airflow configuration options that are not listed for your Apache Airflow version in the dropdown list\. For example, `foo.user` : `YOUR_USER_NAME`\. The configuration setting is translated to your environment's Fargate container as `AIRFLOW__FOO__USER : YOUR_USER_NAME`

### Apache Airflow configuration options<a name="configuring-env-variables-airflow-ref"></a>

The following image shows where you can customize the **Apache Airflow configuration options** on the Amazon MWAA console\.

![\[This image shows where you can customize the Apache Airflow configuration options on the Amazon MWAA console.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-airflow-config.png)

### Apache Airflow reference<a name="configuring-env-variables-reference-options"></a>

The following section contains links to the list of available Apache Airflow configuration options in the *Apache Airflow reference guide*\.
+ **v2\.2\.2**: [Apache Airflow v2\.2\.2 configuration options](https://airflow.apache.org/docs/apache-airflow/2.2.2/configurations-ref.html)
+ **v2\.0\.2**: [Apache Airflow v2\.0\.2 configuration options](https://airflow.apache.org/docs/apache-airflow/2.0.2/configurations-ref.html)
+ **v1\.10\.12**: [Apache Airflow v1\.10\.12 configuration options](https://airflow.apache.org/docs/apache-airflow/1.10.12/configurations-ref.html)

### Using the Amazon MWAA console<a name="configuring-env-variables-console-add"></a>

The following procedure walks you through the steps of adding an Airflow configuration option to your environment\.

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose **Edit**\.

1. Choose **Next**\.

1. Choose **Add custom configuration** in the **Airflow configuration options** pane\.

1. Choose a configuration from the dropdown list and enter a value, or type a custom configuration and enter a value\.

1. Choose **Add custom configuration** for each configuration you want to add\.

1. Choose **Save**\.

## Configuration reference<a name="configuring-env-variables-reference"></a>

The following section contains the list of available Apache Airflow configurations in the dropdown list on the Amazon MWAA console\.

### Email configurations<a name="configuring-env-variables-email"></a>

The following list shows the Airflow email notification configuration options available on Amazon MWAA\. 

We recommend using port 587 for SMTP traffic\. By default, AWS blocks outbound SMTP traffic on port 25 of all Amazon EC2 instances\. If you want to send outbound traffic on port 25, you can [request for this restriction to be removed](https://aws.amazon.com/premiumsupport/knowledge-center/ec2-port-25-throttle/)\.

------
#### [ Apache Airflow v2 ]


| Airflow version | Airflow configuration option | Description | Example value | 
| --- | --- | --- | --- | 
|  v2  |  email\.email\_backend  |  The Apache Airflow utility used for email notifications in [email\_backend](https://airflow.apache.org/docs/apache-airflow/2.0.2/configurations-ref.html#email-backend)\.  |  airflow\.utils\.email\.send\_email\_smtp  | 
|  v2  |  smtp\.smtp\_host  |  The name of the outbound server used for the email address in [smtp\_host](https://airflow.apache.org/docs/apache-airflow/2.0.2/configurations-ref.html#smtp-host)\.  |  localhost  | 
|  v2  |  smtp\.smtp\_starttls  |  Transport Layer Security \(TLS\) is used to encrypt the email over the Internet in [smtp\_starttls](https://airflow.apache.org/docs/apache-airflow/2.0.2/configurations-ref.html#smtp-starttls)\.  |  False  | 
|  v2  |  smtp\.smtp\_ssl  |  Secure Sockets Layer \(SSL\) is used to connect the server and email client in [smtp\_ssl](https://airflow.apache.org/docs/apache-airflow/2.0.2/configurations-ref.html#smtp-ssl)\.  |  True  | 
|  v2  |  smtp\.smtp\_port  |  The Transmission Control Protocol \(TCP\) port designated to the server in [smtp\_port](https://airflow.apache.org/docs/apache-airflow/2.0.2/configurations-ref.html#smtp-port)\.  |  587  | 
|  v2  |  smtp\.smtp\_mail\_from  |  The outbound email address in [smtp\_mail\_from](https://airflow.apache.org/docs/apache-airflow/2.0.2/configurations-ref.html#smtp-mail-from)\.  |  myemail@domain\.com  | 

------
#### [ Apache Airflow v1 ]


| Airflow version | Airflow configuration option | Description | Example value | 
| --- | --- | --- | --- | 
|  v1  |  email\.email\_backend  |  The Apache Airflow utility used for email notifications in [email\_backend](https://airflow.apache.org/docs/apache-airflow/1.10.12/configurations-ref.html#email-backend)\.  |  airflow\.utils\.email\.send\_email\_smtp  | 
|  v1  |  smtp\.smtp\_host  |  The name of the outbound server used for the email address in [smtp\_host](https://airflow.apache.org/docs/apache-airflow/1.10.12/configurations-ref.html#smtp-host)\.  |  localhost  | 
|  v1  |  smtp\.smtp\_starttls  |  Transport Layer Security \(TLS\) is used to encrypt the email over the Internet in [smtp\_starttls](https://airflow.apache.org/docs/apache-airflow/1.10.12/configurations-ref.html#smtp-starttls)\.  |  False  | 
|  v1  |  smtp\.smtp\_ssl  |  Secure Sockets Layer \(SSL\) is used to connect the server and email client in [smtp\_ssl](https://airflow.apache.org/docs/apache-airflow/1.10.12/configurations-ref.html#smtp-ssl)\.  |  True  | 
|  v1  |  smtp\.smtp\_port  |  The Transmission Control Protocol \(TCP\) port designated to the server in [smtp\_port](https://airflow.apache.org/docs/apache-airflow/1.10.12/configurations-ref.html#smtp-port)\.  |  587  | 
|  v1  |  smtp\.smtp\_mail\_from  |  The outbound email address in [smtp\_mail\_from](https://airflow.apache.org/docs/apache-airflow/1.10.12/configurations-ref.html#smtp-mail-from)\.  |  myemail@domain\.com  | 

------

### Task configurations<a name="configuring-env-variables-tasks"></a>

The following list shows the configurations available in the dropdown list for Airflow tasks on Amazon MWAA\. 

------
#### [ Apache Airflow v2 ]


| Airflow version | Airflow configuration option | Description | Example value | 
| --- | --- | --- | --- | 
|  v2  |  core\.default\_task\_retries  |  The number of times to retry an Apache Airflow task in [default\_task\_retries](https://airflow.apache.org/docs/apache-airflow/2.0.2/configurations-ref.html#default-task-retries)\.  |  3  | 
|  v2  |  core\.parallelism  |  The maximum number of task instances that can run simultaneously across the entire environment in parallel \([parallelism](https://airflow.apache.org/docs/apache-airflow/2.0.2/configurations-ref.html#parallelism)\)\.  |  40  | 

------
#### [ Apache Airflow v1 ]


| Airflow version | Airflow configuration option | Description | Example value | 
| --- | --- | --- | --- | 
|  v1  |  core\.default\_task\_retries  |  The number of times to retry an Apache Airflow task in [default\_task\_retries](https://airflow.apache.org/docs/apache-airflow/1.10.12/configurations-ref.html#default-task-retries)\.  |  3  | 
|  v1  |  core\.parallelism  |  The maximum number of task instances that can run simultaneously across the entire environment in parallel \([parallelism](https://airflow.apache.org/docs/apache-airflow/1.10.12/configurations-ref.html#parallelism)\)\.  |  40  | 

------

### Scheduler configurations<a name="configuring-env-variables-scheduler"></a>

The following list shows the Airflow scheduler configurations available in the dropdown list on Amazon MWAA\. 

------
#### [ Apache Airflow v2 ]


| Airflow version | Airflow configuration option | Description | Example value | 
| --- | --- | --- | --- | 
|  v2  |  scheduler\.catchup\_by\_default  |  Tells the scheduler to create a DAG run to "catch up" to the specific time interval in [catchup\_by\_default](https://airflow.apache.org/docs/apache-airflow/2.0.2/configurations-ref.html#catchup-by-default)\.  |  False  | 
|  v2  |  scheduler\.scheduler\_zombie\_task\_threshold  |  Tells the scheduler whether to mark the task instance as failed and reschedule the task in [scheduler\_zombie\_task\_threshold](https://airflow.apache.org/docs/apache-airflow/2.0.2/configurations-ref.html#scheduler-zombie-task-threshold)\.  |  300  | 

------
#### [ Apache Airflow v1 ]


| Airflow version | Airflow configuration option | Description | Example value | 
| --- | --- | --- | --- | 
|  v1  |  scheduler\.catchup\_by\_default  |  Tells the scheduler to create a DAG run to "catch up" to the specific time interval in [catchup\_by\_default](https://airflow.apache.org/docs/apache-airflow/1.10.12/configurations-ref.html#catchup-by-default)\.  |  False  | 
|  v1  |  scheduler\.scheduler\_zombie\_task\_threshold  |  Tells the scheduler whether to mark the task instance as failed and reschedule the task in [scheduler\_zombie\_task\_threshold](https://airflow.apache.org/docs/apache-airflow/1.10.12/configurations-ref.html#scheduler-zombie-task-threshold)\.  |  300  | 

------

### Worker configurations<a name="configuring-env-variables-workers"></a>

The following list shows the Airflow worker configurations available in the dropdown list on Amazon MWAA\. 

------
#### [ Apache Airflow v2 ]


| Airflow version | Airflow configuration option | Description | Example value | 
| --- | --- | --- | --- | 
|  v2  |  celery\.worker\_autoscale  |  The maximum and minimum number of tasks that can run concurrently on any worker using the [Celery Executor](https://airflow.apache.org/docs/apache-airflow/2.0.2/executor/celery.html) in [worker\_autoscale](https://airflow.apache.org/docs/apache-airflow/2.0.2/configurations-ref.html#worker-autoscale)\. Value must be comma\-separated in the following order: `max_concurrency,min_concurrency`\.  |  16,12  | 

------
#### [ Apache Airflow v1 ]


| Airflow version | Airflow configuration option | Description | Example value | 
| --- | --- | --- | --- | 
|  v1  |  celery\.worker\_autoscale  |  The maximum and minimum number of tasks that can run concurrently on any worker using the [Celery Executor](https://airflow.apache.org/docs/apache-airflow/1.10.12/executor/celery.html) in [worker\_autoscale](https://airflow.apache.org/docs/apache-airflow/1.10.12/configurations-ref.html#worker-autoscale)\. Value must be comma\-separated in the following order: `max_concurrency,min_concurrency`\.  |  16,12  | 

------

### Web server configurations<a name="configuring-env-variables-webserver"></a>

The following list shows the Airflow web server configurations available in the dropdown list on Amazon MWAA\. 

------
#### [ Apache Airflow v2 ]


| Airflow version | Airflow configuration option | Description | Example value | 
| --- | --- | --- | --- | 
|  v2  |  webserver\.default\_ui\_timezone  |  The default Apache Airflow UI datetime setting in [default\_ui\_timezone](https://airflow.apache.org/docs/apache-airflow/2.0.2/configurations-ref.html#default-ui-timezone)\.   Setting the `default_ui_timezone` option does not change the time zone in which your DAGs are scheduled to run\. To change the time zone for your DAGs, you can use a custom plugin\. For more information, see [Changing a DAG's timezone on Amazon MWAA](samples-plugins-timezone.md)\.    |  America/New\_York  | 

------
#### [ Apache Airflow v1 ]


| Airflow version | Airflow configuration option | Description | Example value | 
| --- | --- | --- | --- | 
|  v1  |  webserver\.default\_ui\_timezone  |  The default Apache Airflow UI datetime setting in [default\_ui\_timezone](https://airflow.apache.org/docs/apache-airflow/1.10.12/configurations-ref.html#default-ui-timezone)\.  |  America/New\_York  | 

------

## Examples and sample code<a name="configuring-env-variables-code"></a>

### Example DAG<a name="configuring-env-variables-dag"></a>

You can use the following DAG to print your `email_backend` Apache Airflow configuration options\. To run in response to Amazon MWAA events, copy the code to your environment's DAGs folder on your Amazon S3 storage bucket\.

```
def print_var(**kwargs):
    email_backend = kwargs['conf'].get(section='email', key='email_backend')
    print("email_backend")
    return email_backend
with DAG(dag_id="email_backend_dag", schedule_interval="@once", default_args=default_args, catchup=False) as dag:
    email_backend_test = PythonOperator(
        task_id="email_backend_test",
        python_callable=print_var,
        provide_context=True     
    )
```

### Example email notification settings<a name="configuring-env-variables-email"></a>

The following Apache Airflow configuration options can be used for a Gmail\.com email account using an app password\. For more information, see [Sign in using app passwords](https://support.google.com/mail/answer/185833?hl=en-GB) in the *Gmail Help reference guide*\.

![\[This image shows how to configure a gmail.com email account using Apache Airflow configuration options on the MWAA console.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-config-email-configuration.png)

## What's next?<a name="configuring-env-variables-next-up"></a>
+ Learn how to upload your DAG folder to your Amazon S3 bucket in [Adding or updating DAGs](configuring-dag-folder.md)\.