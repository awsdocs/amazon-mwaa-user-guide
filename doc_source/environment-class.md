# Amazon MWAA environment class<a name="environment-class"></a>

The environment class you choose for your Amazon MWAA environment determines the size of the AWS\-managed AWS Fargate containers where the [Celery Executor](https://airflow.apache.org/docs/apache-airflow/stable/executor/celery.html) runs, and the AWS\-managed Amazon Aurora PostgreSQL metadata database where the Apache Airflow schedulers creates task instances\. This page describes each Amazon MWAA environment class, and steps to update the environment class on the Amazon MWAA console\.

**Topics**
+ [Environment class](#environment-class-onconsole)
+ [Environment capabilities](#environment-class-sizes)
+ [Airflow Schedulers](#environment-class-schedulers)

## Environment class<a name="environment-class-onconsole"></a>

The following image shows where you can update the **Environment class** on the Amazon MWAA console\.

------
#### [ Apache Airflow v2 ]

![\[This image shows the Environment class on the Amazon MWAA console.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-environment-class-v202.png)

------
#### [ Apache Airflow v1 ]

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
#### [ Apache Airflow v2 ]
+ **v2** \- Accepts between `2` to `5`\. Defaults to `2`\.

------
#### [ Apache Airflow v1 ]
+ **v1** \- Accepts `1`\.

------