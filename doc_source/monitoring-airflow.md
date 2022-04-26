# Viewing Airflow logs in Amazon CloudWatch<a name="monitoring-airflow"></a>

Amazon MWAA can send Apache Airflow logs to Amazon CloudWatch\. You can view logs for multiple environments from a single location to easily identify Apache Airflow task delays or workflow errors without the need for additional third\-party tools\. Apache Airflow logs need to be enabled on the Amazon Managed Workflows for Apache Airflow \(MWAA\) console to view Apache Airflow DAG processing, tasks, *Web server*, *Worker* logs in CloudWatch\.

**Contents**
+ [Pricing](#monitoring-airflow-pricing)
+ [Before you begin](#monitoring-airflow-before)
+ [Log types](#monitoring-airflow-log-groups)
+ [Enabling Apache Airflow logs](#monitoring-airflow-enable)
+ [Viewing Apache Airflow logs](#monitoring-airflow-view)
+ [Example scheduler logs](#monitoring-airflow-example)
+ [What's next?](#monitoring-airflow-next-up)

## Pricing<a name="monitoring-airflow-pricing"></a>
+ Standard CloudWatch Logs charges apply\. For more information, see [CloudWatch pricing](http://aws.amazon.com/cloudwatch/pricing/)\.

## Before you begin<a name="monitoring-airflow-before"></a>
+ You must have a role that can view logs in CloudWatch\. For more information, see [Accessing an Amazon MWAA environment](access-policies.md)\.

## Log types<a name="monitoring-airflow-log-groups"></a>

Amazon MWAA creates a log group for each Airflow logging option you enable, and pushes the logs to the CloudWatch Logs groups associated with an environment\. Log groups are named in the following format: `YourEnvironmentName-LogType`\. For example, if your environment's named `Airflow-v202-Public`, Apache Airflow task logs are sent to `Airflow-v202-Public-Task`\. 


| Log type | Description | 
| --- | --- | 
|  `YourEnvironmentName-DAGProcessing`  |  The logs of the DAG processor manager \(the part of the scheduler that processes DAG files\)\.  | 
|  `YourEnvironmentName-Scheduler`  |  The logs the Airflow scheduler generates\.  | 
|  `YourEnvironmentName-Task`  |  The task logs a DAG generates\.  | 
|  `YourEnvironmentName-WebServer`  |  The logs the Airflow web interface generates\.  | 
|  `YourEnvironmentName-Worker`  |  The logs generated as part of workflow and DAG execution\.  | 

## Enabling Apache Airflow logs<a name="monitoring-airflow-enable"></a>

You can enable Apache Airflow logs at the `INFO`, `WARNING`, `ERROR`, or `CRITICAL` level\. When you choose a log level, Amazon MWAA sends logs for that level and all higher levels of severity\. For example, if you enable logs at the `INFO` level, Amazon MWAA sends `INFO` logs and `WARNING`, `ERROR`, and `CRITICAL` log levels to CloudWatch Logs\.

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose **Edit**\.

1. Choose **Next**\.

1. Choose one or more of the following logging options:

   1. Choose the **Airflow scheduler log group** on the **Monitoring** pane\.

   1. Choose the **Airflow web server log group** on the **Monitoring** pane\.

   1. Choose the **Airflow worker log group** on the **Monitoring** pane\.

   1. Choose the **Airflow DAG processing log group** on the **Monitoring** pane\.

   1. Choose the **Airflow task log group** on the **Monitoring** pane\.

   1. Choose the logging level in **Log level**\.

1. Choose **Next**\.

1. Choose **Save**\.

## Viewing Apache Airflow logs<a name="monitoring-airflow-view"></a>

The following section describes how to view Apache Airflow logs in the CloudWatch console\.

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose a log group in the **Monitoring** pane\.

1. Choose a log in **Log stream**\.

## Example scheduler logs<a name="monitoring-airflow-example"></a>

You can view Apache Airflow logs for the *Scheduler* scheduling your workflows and parsing your `dags` folder\. The following steps describe how to open the log group for the *Scheduler* on the Amazon MWAA console, and view Apache Airflow logs on the CloudWatch Logs console\.

**To view logs for a `requirements.txt`**

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose the **Airflow scheduler log group** on the **Monitoring** pane\.

1. Choose the `requirements_install_ip` log in **Log streams**\.

1. You should see the list of packages that were installed on the environment at `/usr/local/airflow/.local/bin`\. For example:

   ```
   Collecting appdirs==1.4.4 (from -r /usr/local/airflow/.local/bin (line 1))
   Downloading https://files.pythonhosted.org/packages/3b/00/2344469e2084fb28kjdsfiuyweb47389789vxbmnbjhsdgf5463acd6cf5e3db69324/appdirs-1.4.4-py2.py3-none-any.whl  
   Collecting astroid==2.4.2 (from -r /usr/local/airflow/.local/bin (line 2))
   ```

1. Review the list of packages and whether any of these encountered an error during installation\. If something went wrong, you may see an error similar to the following:

   ```
   2021-03-05T14:34:42.731-07:00
   No matching distribution found for LibraryName==1.0.0 (from -r /usr/local/airflow/.local/bin (line 4))
   No matching distribution found for LibraryName==1.0.0 (from -r /usr/local/airflow/.local/bin (line 4))
   ```

## What's next?<a name="monitoring-airflow-next-up"></a>
+ Learn how to configure a CloudWatch alarm in [Using AWS CloudTrail alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)\.
+ Learn how to create a CloudWatch dashboard in [Using CloudWatch dashboards](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Dashboards.html)\.