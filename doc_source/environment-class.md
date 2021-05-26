# Amazon MWAA environment class<a name="environment-class"></a>

The environment class you choose for your Amazon MWAA environment determines the size of the AWS\-managed AWS Fargate containers where the [Celery Executor](https://airflow.apache.org/docs/apache-airflow/stable/executor/celery.html) runs, and the AWS\-managed Amazon Aurora PostgreSQL metadata database where the Apache Airflow scheduler\(s\) creates task instances\. This page describes each Amazon MWAA environment class, and steps to update the environment class on the Amazon MWAA console\.

**Topics**
+ [Environment class](#environment-class-onconsole)
+ [Environment capabilities](#environment-class-sizes)
+ [Airflow Schedulers](#environment-class-schedulers)
+ [Updating the environment class on the Amazon MWAA console](#environment-class-config)
+ [Updating Airflow Schedulers on the Amazon MWAA console](#environment-class-schedulers-mwaa-console)
+ [Troubleshooting environment updates](#environment-class-troubleshooting)

## Environment class<a name="environment-class-onconsole"></a>

The following image shows where you can update the **Environment class** on the Amazon MWAA console\.

------
#### [ Airflow v2\.0\.2 ]

![\[This image shows the Environment class on the Amazon MWAA console.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-environment-class-v202.png)

------
#### [ Airflow v1\.10\.12 ]

![\[This image shows the Environment class on the Amazon MWAA console.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-environment-class-v11012.png)

------

## Environment capabilities<a name="environment-class-sizes"></a>

The following section contains the default concurrent Apache Airflow tasks, Random Access Memory \(RAM\), and the virtual centralized processing units \(vCPUs\) for each environment class\. The concurrent tasks listed assume that task concurrency does not exceed the Apache Airflow *Worker* capacity in the environment\.

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

## Airflow Schedulers<a name="environment-class-schedulers"></a>

The following section contains the Apache Airflow *Scheduler* options available on the Amazon MWAA console\.

------
#### [ Airflow v2\.0\.2 ]
+ **v2\.0\.2** \- Accepts between `2` to `5`\. Defaults to `2`\.

------
#### [ Airflow v1\.10\.12 ]
+ **v1\.10\.12** \- Accepts `1`\.

------

## Updating the environment class on the Amazon MWAA console<a name="environment-class-config"></a>

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

## Updating Airflow Schedulers on the Amazon MWAA console<a name="environment-class-schedulers-mwaa-console"></a>

If you're using Apache Airflow v2\.0\.2, you can update the number of Apache Airflow *Schedulers* on the Amazon MWAA console at any time\.

**To configure the environment size**

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose **Edit**\.

1. Choose **Next**\.

1. On the **Environment class** pane, enter a value up to `5` *Schedulers*\. 

1. Choose **Save**\.

**Note**  
It can take up to 30 minutes for changes to take effect\.

## Troubleshooting environment updates<a name="environment-class-troubleshooting"></a>
+ If you update your environment to a different environment class \(such as changing an `mw1.medium` to an `mw1.small`\), and the request to update your environment failed, the environment status goes into an `UPDATE_FAILED` state and the environment is rolled back to, and is billed according to, the previous stable version of an environment\. To learn more, see [I tried changing the environment class but the update failed](t-create-update-environment.md#t-rollback-billing-failure)\.