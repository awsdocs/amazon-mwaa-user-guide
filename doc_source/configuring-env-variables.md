# Amazon MWAA Apache Airflow configuration options<a name="configuring-env-variables"></a>

Apache Airflow configuration options can be attached to your Amazon Managed Workflows for Apache Airflow \(MWAA\) environment as environment variables\. You can choose from the suggested dropdown list, or specify any [Apache Airflow configuration options](https://airflow.apache.org/docs/stable/configurations-ref.html) for your environment on the Amazon MWAA console\. This page describes the Apache Airflow configuration options available in the dropdown list on the Amazon MWAA console, and how to use these options to override Apache Airflow configuration settings\.

**Topics**
+ [Prerequisites](#configuring-env-variables-prereqs)
+ [How it works](#configuring-env-variables-how)
+ [Apache Airflow configuration options](#configuring-env-variables-onconsole)
+ [Using the Amazon MWAA console](#configuring-env-variables-console)
+ [Configuration reference](#configuring-env-variables-reference)
+ [Examples and sample code](#configuring-env-variables-code)
+ [What's next?](#configuring-env-variables-next-up)

## Prerequisites<a name="configuring-env-variables-prereqs"></a>

**To use the steps on this page, you'll need:**

1. The required AWS resources configured for your environment as defined in [Get started with Amazon Managed Workflows for Apache Airflow \(MWAA\)](get-started.md)\.

1. An execution role with a permissions policy that grants Amazon MWAA access to the AWS resources used by your environment as defined in [Amazon MWAA Execution role](mwaa-create-role.md)\.

1. An AWS account with access in AWS Identity and Access Management \(IAM\) to the Amazon S3 console, or the AWS Command Line Interface \(AWS CLI\) as defined in [Accessing an Amazon MWAA environment](access-policies.md)\.

## How it works<a name="configuring-env-variables-how"></a>

When you create an environment, Amazon MWAA attaches the configuration settings you specify on the Amazon MWAA console in **Airflow configuration options** as environment variables to the AWS Fargate container for your environment\. If you are using a setting of the same name in `airflow.cfg`, the options you specify on the Amazon MWAA console override the values in `airflow.cfg`\.
+ While we don't expose the `airflow.cfg` in the Apache Airflow UI of an Amazon MWAA environment, you can change the Apache Airflow configuration options directly on the Amazon MWAA console and continue using all other settings in `airflow.cfg`\.
+ You can choose from one of the suggested options in the dropdown list, or specify any Airflow configuration option\. For example, you could type a value of `core.dag_concurrency` and enter a value of `10`\.
+ When you add a configuration on the Amazon MWAA console, Amazon MWAA writes the configuration as an environment variable\. For example, if you add a configuration, such as `dag_concurrency`, Amazon MWAA writes the value to your environment's Fargate container as `AIRFLOW__CORE__DAG_CONCURRENCY`\.

For a complete reference, see [Configuration reference](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#) in the *Apache Airflow reference guide*\.

## Apache Airflow configuration options<a name="configuring-env-variables-onconsole"></a>

The following image shows where you can customize the **Apache Airflow configuration options** on the Amazon MWAA console\.

![\[This image shows where you can customize the Apache Airflow configuration options on the Amazon MWAA console.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-airflow-config.png)

## Using the Amazon MWAA console<a name="configuring-env-variables-console"></a>

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

### Email notifications<a name="configuring-env-variables-email"></a>

The following list shows the email notification configuration options available on the Amazon MWAA console\. 


| Amazon MWAA UI selection | Apache Airflow configuration option | Description | Example value | 
| --- | --- | --- | --- | 
|  email\.email\_backend  |  email\.email\_backend  |  The Apache Airflow utility used for email notifications in [email\_backend](https://airflow.apache.org/docs/stable/configurations-ref.html#email-backend)\.  |  airflow\.utils\.email\.send\_email\_smtp  | 
|  smtp\.smtp\_host  |  smtp\.smtp\_host  |  The name of the outbound server used for the email address in [smtp\_host](https://airflow.apache.org/docs/stable/configurations-ref.html#smtp-host)\.  |  localhost  | 
|  smtp\.smtp\_starttls  |  smtp\.smtp\_starttls  |  Transport Layer Security \(TLS\) is used to encrypt the email over the Internet in [smtp\_starttls](https://airflow.apache.org/docs/stable/configurations-ref.html#smtp-starttls)\.  |  False  | 
|  smtp\.smtp\_ssl  |  smtp\.smtp\_ssl  |  Secure Sockets Layer \(SSL\) is used to connect the server and email client in [smtp\_ssl](https://airflow.apache.org/docs/stable/configurations-ref.html#smtp-ssl)\.  |  True  | 
|  smtp\.smtp\_port  |  smtp\.smtp\_port  |  The Transmission Control Protocol \(TCP\) port designated to the server in [smtp\_port](https://airflow.apache.org/docs/stable/configurations-ref.html#smtp-port)\.  |  25  | 
|  smtp\.smtp\_mail\_from  |  smtp\.smtp\_mail\_from  |  The outbound email address in [smtp\_mail\_from](https://airflow.apache.org/docs/stable/configurations-ref.html#smtp-mail-from)\.  |  myemail@domain\.com  | 

### Task configurations<a name="configuring-env-variables-tasks"></a>

The following list shows the configurations available in the dropdown list for tasks on the Amazon MWAA console\. 


| Amazon MWAA UI selection | Apache Airflow configuration option | Description | Example value | 
| --- | --- | --- | --- | 
|  core\.default\_task\_retries  |  core\.default\_task\_retries  |  The number of times to retry an Apache Airflow task in [default\_task\_retries](https://airflow.apache.org/docs/stable/configurations-ref.html#default-task-retries)\.  |  3  | 
|  core\.parallelism  |  core\.parallelism  |  The maximum number of task instances that can run simultaneously across the entire environment in parallel \([parallelism](https://airflow.apache.org/docs/stable/configurations-ref.html#parallelism)\)\.  |  40  | 

### Scheduler configurations<a name="configuring-env-variables-scheduler"></a>

The following list shows the scheduler configurations available in the dropdown list on the Amazon MWAA console\. 


| Amazon MWAA UI selection | Apache Airflow configuration option | Description | Example value | 
| --- | --- | --- | --- | 
|  scheduler\.catchup\_by\_default  |  scheduler\.catchup\_by\_default  |  Tells the scheduler to create a DAG run to "catch up" to the specific time interval in [catchup\_by\_default](https://airflow.apache.org/docs/stable/configurations-ref.html#catchup-by-default)\.  |  False  | 
|  scheduler\.scheduler\_zombie\_task\_threshold  |  scheduler\.scheduler\_zombie\_task\_threshold  |  Tells the scheduler whether to mark the task instance as failed and reschedule the task in [scheduler\_zombie\_task\_threshold](https://airflow.apache.org/docs/stable/configurations-ref.html#scheduler-zombie-task-threshold)\.  |  300  | 

### Worker configurations<a name="configuring-env-variables-workers"></a>

The following list shows the configurations available in the dropdown list for workers on the Amazon MWAA console\. 


| Amazon MWAA UI selection | Apache Airflow configuration option | Description | Example value | 
| --- | --- | --- | --- | 
|  celery\.worker\_autoscale  |  celery\.worker\_autoscale  |  The maximum and minimum number of tasks that can run concurrently on any worker using the [Celery Executor](https://airflow.apache.org/docs/apache-airflow/stable/executor/celery.html) in [worker\_autoscale](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#worker-autoscale)\. Value must be comma\-separated in the following order: `max_concurrency,min_concurrency`\.  |  16,12  | 

### System settings<a name="configuring-env-variables-system"></a>

The following list shows the configurations available in the dropdown list for Apache Airflow system settings on the Amazon MWAA console\. 


| Amazon MWAA UI selection | Apache Airflow configuration option | Description | Example value | 
| --- | --- | --- | --- | 
|  core\.default\_ui\_timezone  |  core\.default\_ui\_timezone  |  The default Apache Airflow UI datetime setting in [default\_ui\_timezone](https://airflow.apache.org/docs/stable/configurations-ref.html#default-ui-timezone)\.  |  America/New\_York  | 

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

### Example email configuration<a name="configuring-env-variables-email"></a>

The following Apache Airflow configuration options can be used for a Gmail\.com email account using an app password\. For more information, see [Sign in using app passwords](https://support.google.com/mail/answer/185833?hl=en-GB) in the *Gmail Help reference guide*\.

![\[This image shows how to configure a gmail.com email account using Apache Airflow configuration options on the MWAA console.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-config-email-configuration.png)

## What's next?<a name="configuring-env-variables-next-up"></a>
+ Learn how to upload your DAG folder to your Amazon S3 bucket in [Adding or updating DAGs](configuring-dag-folder.md)\.