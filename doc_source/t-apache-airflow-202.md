# Troubleshooting: DAGs, Operators, Connections, and other issues in Apache Airflow v2\.0\.2<a name="t-apache-airflow-202"></a>

The topics on this page contains resolutions to Apache Airflow v2\.0\.2 Python dependencies, custom plugins, DAGs, Operators, Connections, tasks, and *Web server* issues you may encounter on an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment\.

**Contents**
+ [Connections](#troubleshooting-conn-202)
  + [I can't connect to Secrets Manager](#access-secrets-manager-202)
+ [Web server](#troubleshooting-webserver-202)
  + [I see a 5xx error accessing the web server](#5xx-webserver-202)
+ [Tasks](#troubleshooting-tasks-202)
  + [I see my tasks stuck or not completing](#stranded-tasks-202)

## Connections<a name="troubleshooting-conn-202"></a>

The following topic describes the errors you may receive when using an Apache Airflow connection, or using another AWS database\.

### I can't connect to Secrets Manager<a name="access-secrets-manager-202"></a>

We recommend the following steps:

1. Learn how to create secret keys for your Apache Airflow connection and variables in [Configuring an Apache Airflow connection using a Secrets Manager secret key](connections-secrets-manager.md)\.

1. Learn how to use the secret key for an Apache Airflow variable \(`test-variable`\) in [Using a secret key in AWS Secrets Manager for an Apache Airflow variable](samples-secrets-manager-var.md)\.

1. Learn how to use the secret key for an Apache Airflow connection \(`myconn`\) in [Using a secret key in AWS Secrets Manager for an Apache Airflow connection](samples-secrets-manager.md)\.

## Web server<a name="troubleshooting-webserver-202"></a>

The following topic describes the errors you may receive for your Apache Airflow *Web server* on Amazon MWAA\.

### I see a 5xx error accessing the web server<a name="5xx-webserver-202"></a>

We recommend the following steps:

1. Check Apache Airflow configuration options\. Verify that the key\-value pairs you specified as an Apache Airflow configuration option, such as AWS Secrets Manager, were configured correctly\. To learn more, see [I can't connect to Secrets Manager](t-apache-airflow-11012.md#access-secrets-manager)\.

1. Check the `requirements.txt`\. Verify the Airflow "extras" package and other libraries listed in your `requirements.txt` are compatible with your Apache Airflow version\.

1. Explore ways to specify Python dependencies in a `requirements.txt` file, see [Managing Python dependencies in requirements\.txt](best-practices-dependencies.md)\.

## Tasks<a name="troubleshooting-tasks-202"></a>

The following topic describes the errors you may receive for Apache Airflow tasks in an environment\.

### I see my tasks stuck or not completing<a name="stranded-tasks-202"></a>

If your Apache Airflow tasks are "stuck" or not completing, we recommend the following steps:

1. There may be a large number of DAGs defined\. Reduce the number of DAGs and perform an update of the environment \(such as changing a log level\) to force a reset\.

   1. Airflow parses DAGs whether they are enabled or not\. If you're using greater than 50% of your environment's capacity you may start overwhelming the Apache Airflow *Scheduler*\. This leads to large *Total Parse Time* in CloudWatch Metrics or long DAG processing times in CloudWatch Logs\.

     There are other ways to optimize Apache Airflow configurations which are outside the scope of this guide\.

1. There may be a large number of tasks in the queue\. This often appears as a large—and growing—number of tasks in the "None" state, or as a large number in *Queued Tasks* and/or *Tasks Pending* in CloudWatch Metrics\. This can occur for the following reasons:

   1. If there are more tasks to run than the environment has the capacity to run, and/or a large number of tasks that were queued before autoscaling has time to detect the tasks and deploy additional workers\. 

   1. If there are more tasks to run than an environment has the capacity to run, we recommend **reducing** the number of tasks that your DAGs run concurrently, and/or increasing the minimum Apache Airflow *Workers*\.

   1. If there are a large number of tasks that were queued before autoscaling has had time to detect and deploy additional workers, we recommend **staggering** task deployment and/or increasing the minimum Apache Airflow *Workers*\.

   1. The following AWS Command Line Interface \(AWS CLI\) command can be used to change the minimum or maximum number of workers in your environment\.

      ```
      aws mwaa update-environment --name MyEnvironmentName --min-workers 2 --max-workers 10
      ```

1. There may be tasks being deleted mid\-exectution that appear as task logs which stop with no further indication in Apache Airflow\. This can occur for the following reasons:

   1. If there is a brief moment where 1\) the current tasks exceed current environment capacity, followed by 2\) a few minutes of no tasks executing or being queued, then 3\) new tasks being queued\. 

   1. Amazon MWAA autoscaling reacts to the first scenario by adding additional workers\. In the second scenario, it removes the additional workers\. Some of the tasks being queued may result with the workers in the process of being removed, and will end when the container is deleted\.

   1. We recommend increasing the minimum number of workers on your environment\. You can also set the minimum workers equal to the maximum workers on your environment, effectively disabling autoscaling\. Another option is to adjust the timing of your DAGs and tasks to ensure that that these scenarios don't occur\.

   1. The following AWS Command Line Interface \(AWS CLI\) command can be used to change the minimum or maximum number of workers in your environment to be the same, disabling autoscaling\.

      ```
      aws mwaa update-environment --name MyEnvironmentName --min-workers 5 --max-workers 5
      ```

1. If your tasks are stuck in the "running" state, you can also clear the tasks or mark them as succeeded or failed\. This allows the autoscaling component for your environment to scale down the number of workers running in an environment\. The following image shows an example of a stranded task\.  
![\[This is an image with a stranded task.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-airflow-scaling.png)

   1. Choose the circle for the stranded task, and then select **Clear** \(as shown\)\. This allows Amazon MWAA to scale down workers; otherwise, Amazon MWAA can't determine which DAGs are enabled or disabled, and can't scale down, if there are still queued tasks\.  
![\[Apache Airflow Actions\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-airflow-scaling-menu.png)

1. Learn more about the Apache Airflow task lifecycle at [Concepts](https://airflow.apache.org/docs/apache-airflow/stable/concepts.html#task-lifecycle) in the *Apache Airflow reference guide*\.