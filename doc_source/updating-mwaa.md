# Updating an Amazon MWAA environment<a name="updating-mwaa"></a>

 The following section describes how you can update your environment preferences and settings, such as the environment class, the number of Apache Airflow schedulers, your environment's maintenance window, as well as the path for your DAGs, `requirements.txt`, and `plugins.zip` resources in Amazon S3\. 

**Topics**
+ [Updating your environment using the Amazon MWAA console](#environment-class-config)
+ [Troubleshooting environment updates](#environment-class-troubleshooting)

## Updating your environment using the Amazon MWAA console<a name="environment-class-config"></a>

You can update your environment on the Amazon MWAA console at any time\.

**To configure the environment**

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment from the list\.

1. Choose **Edit**\.

1. \(Optional\) On the **Specify details** page, you can do the following:

   1.  In the **Environment details** section, for **Weekly maintenance window start \(UTC\)** option, you can choose a day of the week and the time in 24\-hour format \(`HH:mm)` from the dropdown lists to specify when Amazon MWAA will perform scheduled maintenance operations for your environment\. 

   1.  In the **DAG code in Amazon S3** section, you can change your environment's Amazon S3 bucket, specify a different DAGs folder in your bucket, and change the location or version of your `plugins.zip` and `requirements.txt` files\. 

1. Choose **Next**\.

1. \(Optional\) On the **Configure advanced settings** page, in the **Networking** section, can do the following:

   1.  **Web server access** — Change your environment's web server access mode\. Choose **Private network** to restrict access to the Apache Airflow web server only to your Amazon VPC\. Choose **Public access** if you want to be able to access the web server using secure login over the internet\. For more information, see [Apache Airflow access modes](configuring-networking.md)\. 

   1.  **Security group\(s\)** — Create a new [security group](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html), and add or remove existing ones to configure the inbound and outbound rules for your environment\. 

1. \(Optional\) In the **Environment class** section, you can do the following:

   1.  Change the class type to configure your environment's scheduler, worker, and web server vCPU capacity\. For more information about environment classes, see [Amazon MWAA environment class](environment-class.md)\. 

   1.  For **Maximum worker count**, specify a number between 1 and 25 to set the maximum number of workers that Amazon MWAA can scale up to\. When a worker reaches its maximum task capacity, Amazon MWAA adds additional workers until the total number of workers reaches the number you set here\. 

   1.  For **Minimum worker count**, specify a number less than **Maximum worker count** to set the minimum number of workers that Amazon MWAA maintains to run tasks\. 

   1.  For **Scheduler count**, specify the number of schedulers you want to run for your environment\. 
      +  **Apache Airflow v2** — Accepts between values between **2** and **5**\. 
      +  **Apache Airflow v1** — Accepts **1**\. 

1.  \(Optional\) In the **Monitoring** section, under **Airflow logging configuration** you can activate or deactivate CloudWatch Logs for various Apache Airflow log types, and choose the **Log level** for each log type from the corresponding dropdown lists\. For more information on Apache Airflow logs, see [Viewing Airflow logs in Amazon CloudWatch](monitoring-airflow.md)\. 

1.  \(Optional\) In the **Airflow configuration options** section, you can add or remove custom configuration values to override the Apache Airflow defaults\. For more information on Apache Airflow configuration options, see [Apache Airflow configuration options](configuring-env-variables.md)\. 

1.  \(Optional\) In the **Permissions** section, under **Execution role** you can choose the IAM role used by your environment to perform various actions such as access your DAGs and resources in Amazon S3, and send logs to CloudWatch\. 

1.  Choose **Next**\. 

1.  On the **Review and save** page, review the changes you've made to your environment\. 

1. Choose **Save**\.

**Note**  
When you update your environment, it can take between 10 to 30 minutes for changes to take effect\.

## Troubleshooting environment updates<a name="environment-class-troubleshooting"></a>
+ If you update your environment to a different environment class \(such as changing an `mw1.medium` to an `mw1.small`\), and the request to update your environment fails, the status changes to the `UPDATE_FAILED` state, the environment is rolled back, and is billed according to, the previous stable version of the environment\. For more information, see [I tried changing the environment class but the update failed](t-create-update-environment.md#t-rollback-billing-failure)\.