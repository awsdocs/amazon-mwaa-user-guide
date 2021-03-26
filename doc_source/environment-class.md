# Amazon MWAA environment class<a name="environment-class"></a>

The environment class you choose for your Amazon MWAA environment determines the size of the AWS\-managed AWS Fargate containers where the [Celery Executor](https://airflow.apache.org/docs/apache-airflow/stable/executor/celery.html) runs, and the AWS\-managed Amazon Aurora PostgreSQL metadata database where the Apache Airflow scheduler creates task instances\. This page describes each Amazon MWAA environment class, and steps to update the environment class on the Amazon MWAA console\.

**Topics**
+ [Prerequisites](#environment-class-prereqs)
+ [Environment class](#environment-class-onconsole)
+ [Environment capabilities](#environment-class-sizes)
+ [Using the Amazon MWAA console](#environment-class-config)

## Prerequisites<a name="environment-class-prereqs"></a>

**To use the steps on this page, you'll need:**

1. The required AWS resources configured for your environment as defined in [Get started with Amazon Managed Workflows for Apache Airflow \(MWAA\)](get-started.md)\.

1. An execution role with a permissions policy that grants Amazon MWAA access to the AWS resources used by your environment as defined in [Amazon MWAA Execution role](mwaa-create-role.md)\.

1. An AWS account with access in AWS Identity and Access Management \(IAM\) to the Amazon S3 console, or the AWS Command Line Interface \(AWS CLI\) as defined in [Accessing an Amazon MWAA environment](access-policies.md)\.

## Environment class<a name="environment-class-onconsole"></a>

The following image shows where you can update the **Environment class** on the Amazon MWAA console\.

![\[This image shows the Environment class on the Amazon MWAA console.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-environment-class.png)

## Environment capabilities<a name="environment-class-sizes"></a>

The environment class you choose for your Amazon MWAA environment determines the size of the AWS\-managed AWS Fargate containers where the [Celery Executor](https://airflow.apache.org/docs/apache-airflow/stable/executor/celery.html) runs, and the AWS\-managed Amazon Aurora PostgreSQL metadata database where the Apache Airflow scheduler creates task instances\.

------
#### [ mw1\.small ]
+ 5 concurrent tasks \(by default\)
+ 1 vCPUs
+ 2 GB RAM

------
#### [ mw1\.medium ]
+ 10 concurrent tasks \(by default\)
+ 2 vCPUs
+ 4 GB RAM

------
#### [ mw1\.large ]
+ 20 concurrent tasks \(by default\)
+ 4 vCPUs
+ 8 GB RAM

------

You can use `celery.worker_autoscale` to increase tasks per worker\. To learn more, see the [Example high performance use case](mwaa-autoscaling.md#mwaa-autoscaling-high-volume)\.

## Using the Amazon MWAA console<a name="environment-class-config"></a>

You can update your environment class on the Amazon MWAA console at any time\.

**To configure the environment size**

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose **Edit**\.

1. Choose **Next**\.

1. On the **Environment class** pane, choose an option\. 

1. Choose **Save**\.

**Note**  
It can take up to 30 minutes for changes to take effect\.