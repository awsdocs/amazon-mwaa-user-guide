# Performance tuning for Apache Airflow on Amazon MWAA<a name="best-practices-tuning"></a>

This page describes the best practices we recommend to tune the performance of an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment using [Apache Airflow configuration options](configuring-env-variables.md)\. 

**Contents**
+ [Adding an Airflow configuration option](#best-practices-tuning-console-add)
+ [Airflow Scheduler](#best-practices-tuning-scheduler)
  + [Parameters](#best-practices-tuning-scheduler-params)
  + [Limits](#best-practices-tuning-scheduler-limits)
+ [DAG folders](#best-practices-tuning-dag-folders)
  + [Parameters](#best-practices-tuning-dag-folders-params)
+ [DAG files](#best-practices-tuning-dag-files)
  + [Parameters](#best-practices-tuning-dag-files-params)
+ [Tasks](#best-practices-tuning-tasks)
  + [Parameters](#best-practices-tuning-tasks-params)
+ [Extending Apache Airflow](#best-practices-tuning-extending)

## Adding an Airflow configuration option<a name="best-practices-tuning-console-add"></a>

The following procedure walks you through the steps of adding an Airflow configuration option to your environment\.

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose **Edit**\.

1. Choose **Next**\.

1. Choose **Add custom configuration** in the **Airflow configuration options** pane\.

1. Choose a configuration from the dropdown list and enter a value, or type a custom configuration and enter a value\.

1. Choose **Add custom configuration** for each configuration you want to add\.

1. Choose **Save**\.

To learn more, see [Apache Airflow configuration options](configuring-env-variables.md)\.

## Airflow Scheduler<a name="best-practices-tuning-scheduler"></a>

The Apache Airflow *Scheduler* is a core component of Apache Airflow\. An issue with the *Scheduler* can prevent DAGs from being parsed and tasks from being scheduled\. For more information about Apache Airflow *Scheduler* tuning, see [Fine\-tuning your Scheduler performance](https://airflow.apache.org/docs/apache-airflow/2.2.2/concepts/scheduler.html#fine-tuning-your-scheduler-performance) in the Apache Airflow documentation website\.

**Note**  
We recommend exercising caution when changing the default values for the parameters in this section\.

### Parameters<a name="best-practices-tuning-scheduler-params"></a>

This section describes the configuration options available for the Airflow *Scheduler* and their use cases\. 

------
#### [ Apache Airflow v2 ]


| Airflow version | Airflow configuration option | Default | Description | Use case | 
| --- | --- | --- | --- | --- | 
|  v2  |  [celery\.sync\_parallelism](https://airflow.apache.org/docs/apache-airflow/2.0.2/configurations-ref.html#sync-parallelism)   |  1  |  The number of processes the Celery Executor uses to sync task state\.  |  You can use this option to prevent queue conflicts by limiting the processes the Celery Executor uses\. By default, a value is set to `1` to prevent errors in delivering task logs to CloudWatch Logs\. Setting the value to `0` means using the maximum number of processes, but might cause errors when delivering task logs\.  | 
|  v2  |  [scheduler\.processor\_poll\_interval](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#processor-poll-interval)  |  1  |  The number of seconds to wait between consecutive DAG file processing in the *Scheduler* "loop\."  |  You can use this option to free up CPU usage on the *Scheduler* by **increasing** the time the *Scheduler* sleeps after it's finished retrieving DAG parsing results, finding and queuing tasks, and executing queued tasks in the *Executor*\. Increasing this value consumes the number of *Scheduler* threads run on an environment in `scheduler.parsing_processes` for Apache Airflow v2 and `scheduler.max_threads` for Apache Airflow v1\. This may reduce the capacity of the *Schedulers* to parse DAGs, and increase the time it takes for DAGs to appear in the *Web server*\.  | 
|  v2  |  [scheduler\.max\_dagruns\_to\_create\_per\_loop](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#max-dagruns-to-create-per-loop)  |  10  |  The maximum number of DAGs to create *DagRuns* for per *Scheduler* "loop\."  |  You can use this option to free up resources for scheduling tasks by **decreasing** the maximum number of *DagRuns* for the *Scheduler* "loop\."  | 
|  v2  |  [scheduler\.parsing\_processes](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#config-scheduler-parsing-processes)   |  4  |  The number of threads the *Scheduler* can run in parallel to schedule DAGs\.  |  You can use this option to free up resources by **decreasing** the number of processes the *Scheduler* runs in parallel to parse DAGs\. We recommend keeping this number low if DAG parsing is impacting task scheduling\. You **must** specify a value that's less than the vCPU count on your environment\. To learn more, see [Limits](#best-practices-tuning-scheduler-limits)\.  | 

------
#### [ Apache Airflow v1 ]


| Airflow version | Airflow configuration option | Default | Description | Use case | 
| --- | --- | --- | --- | --- | 
|  v1  |  [celery\.sync\_parallelism](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#sync-parallelism)   |  0  |  The number of processes the Celery Executor uses to sync task state\.  |  You can use this option to prevent queue conflicts by limiting the processes the Celery Executor uses\. By default, a value is set to `1` to prevent errors in delivering task logs to CloudWatch Logs\. Setting the value to `0` means using the maximum number of processes, but might cause errors when delivering task logs\.  | 
|  v1  |  [scheduler\.processor\_poll\_interval](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#processor-poll-interval)  |  1  |  The number of seconds to wait between consecutive DAG file processing in the *Scheduler* "loop\."  |  You can use this option to free up CPU usage on the *Scheduler* by **increasing** the time the *Scheduler* sleeps after it's finished retrieving DAG parsing results, finding and queuing tasks, and executing queued tasks in the *Executor*\. Increasing this value consumes the number of *Scheduler* threads run on an environment in `scheduler.parsing_processes` for Apache Airflow v2 and `scheduler.max_threads` for Apache Airflow v1\. This may reduce the capacity of the *Schedulers* to parse DAGs, and increase the time it takes for DAGs to appear in the *Web server*\.  | 
|  v1  |  [scheduler\.max\_threads](https://airflow.apache.org/docs/apache-airflow/1.10.12/configurations-ref.html#max-threads)  |  4  |  The number of threads the *Scheduler* can run in parallel to schedule DAGs\.  |  You can use this option to free up resources by **decreasing** the number of processes the *Scheduler* runs in parallel to parse DAGs\. We recommend keeping this number low if DAG parsing is impacting task scheduling\. You **must** specify a value that's less than the vCPU count on your environment\. To learn more, see [Limits](#best-practices-tuning-scheduler-limits)\.  | 

------

### Limits<a name="best-practices-tuning-scheduler-limits"></a>

This section describes the limits you should consider when adjusting the default parameters for the *Scheduler*\.<a name="scheduler-considerations"></a>

**scheduler\.parsing\_processes, scheduler\.max\_threads**  
Two threads are allowed per vCPU for an environment class\. At least one thread must be reserved for the *Scheduler* for an environment class\. If you notice a delay in tasks being scheduled, you may need to increase your [environment class](environment-class.md)\. For example, a large environment has a 4 vCPU Fargate container instance for its *Scheduler*\. This means that a maximum of `7` total threads are available to use for other processes\. That is, two threads "times" four vCPUs, minus one for the *Scheduler*\. The value you specify in `scheduler.max_threads` and `scheduler.parsing_processes` must not exceed the number of threads available for an environment class \(as shown, below\):  
+ **mw1\.small** – Must not exceed `1` thread for other processes\. The remaining thread is reserved for the *Scheduler*\.
+ **mw1\.medium** – Must not exceed `3` threads for other processes\. The remaining thread is reserved for the *Scheduler*\.
+ **mw1\.large** – Must not exceed `7` threads for other processes\. The remaining thread is reserved for the *Scheduler*\.

## DAG folders<a name="best-practices-tuning-dag-folders"></a>

The Apache Airflow *Scheduler* continuously scans the DAGs folder on your environment\. Any contained `plugins.zip` files, or Python \(`.py`\) files containing “airflow” import statements\. Any resulting Python DAG objects are then placed into a *DagBag* for that file to be processed by the *Scheduler* to determine what, if any, tasks need to be scheduled\. Dag file parsing occurs regardless of whether the files contain any viable DAG objects\. 

**Note**  
We recommend exercising caution when changing the default values for the parameters in this section\.

### Parameters<a name="best-practices-tuning-dag-folders-params"></a>

This section describes the configuration options available for the *DAGs folder* and their use cases\. 

------
#### [ Apache Airflow v2 ]


| Airflow version | Airflow configuration option | Default | Description | Use case | 
| --- | --- | --- | --- | --- | 
|  v2  |  [scheduler\.dag\_dir\_list\_interval](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#dag-dir-list-interval)  |  300 seconds  |  The number of seconds the DAGs folder should be scanned for new files\.  |  You can use this option to free up resources by **increasing** the number of seconds to parse the DAGs folder\. We recommend increasing this value if you're seeing long parsing times in `total_parse_time metrics`, which may be due to a large number of files in your DAGs folder\.  | 
|  v2  |  [scheduler\.min\_file\_process\_interval](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#min-file-process-interval)  |  30 seconds  |  The number of seconds after which the scheduler parses a DAG and updates to the DAG are reflected\.  |  You can use this option to free up resources by **increasing** the number of seconds that the scheduler waits before parsing a DAG\. For example, if you specify a value of `30`, the DAG file is parsed after every 30 seconds\. We recommend keeping this number high to decrease the CPU usage on your environment\.  | 

------
#### [ Apache Airflow v1 ]


| Airflow version | Airflow configuration option | Default | Description | Use case | 
| --- | --- | --- | --- | --- | 
|  v1  |  [scheduler\.dag\_dir\_list\_interval](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#dag-dir-list-interval)  |  300 seconds  |  The number of seconds the DAGs folder should be scanned for new files\.  |  You can use this option to free up resources by **increasing** the number of seconds to parse the DAGs folder\. We recommend increasing this value if you're seeing long parsing times in `total_parse_time metrics`, which may be due to a large number of files in your DAGs folder\.  | 
|  v1  |  [scheduler\.min\_file\_process\_interval](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#min-file-process-interval)  |  30 seconds  |  The number of seconds after which the scheduler parses a DAG and updates to the DAG are reflected\.  |  You can use this option to free up resources by **increasing** the number of seconds that the scheduler waits before parsing a DAG\. For example, if you specify a value of `30`, the DAG file is parsed after every 30 seconds\. We recommend keeping this number high to decrease the CPU usage on your environment\.  | 

------

## DAG files<a name="best-practices-tuning-dag-files"></a>

As part of the Apache Airflow *Scheduler* "loop," individual DAG files are parsed to extract DAG Python objects\. A maximum of *Scheduler* [parsing processes](https://airflow.apache.org/docs/apache-airflow/2.0.2/configurations-ref.html#parsing-processes) in Airflow 2\.0\+, files are processed at the same time and the number of seconds in `scheduler.min_file_process_interval` must pass before the same file is parsed again\.

**Note**  
We recommend exercising caution when changing the default values for the parameters in this section\.

### Parameters<a name="best-practices-tuning-dag-files-params"></a>

This section describes the configuration options available for Airflow *DAG files* and their use cases\. 

------
#### [ Apache Airflow v2 ]


| Airflow version | Airflow configuration option | Default | Description | Use case | 
| --- | --- | --- | --- | --- | 
|  v2  |  [core\.dag\_file\_processor\_timeout](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#dag-file-processor-timeout)  |  50 seconds  |  The number of seconds before the *DagFileProcessor* times out processing a DAG file\.  |  You can use this option to free up resources by **increasing** the time it takes before the *DagFileProcessor* times out\. We recommend increasing this value if you're seeing timeouts in your DAG processing logs that result in no viable DAGs being loaded\.   | 
|  v2  |  [core\.dagbag\_import\_timeout](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#dagbag-import-timeout)  |  30 seconds  |  The number of seconds before importing a Python file times out\.   |  You can use this option to free up resources by **increasing** the time it takes before the *Scheduler* times out while importing a Python file to extract the DAG objects\. This option is processed as part of the *Scheduler* "loop," and must contain a value lower than the value specified in `core.dag_file_processor_timeout`\.  | 
|  v2  |  [core\.min\_serialized\_dag\_update\_interval](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#min-serialized-update-dag-update-interval)  |  30  |  The minimum number of seconds after which serialized DAGs in the database are updated\.  |  You can use this option to free up resources by **increasing** the number of seconds after which serialized DAGs in the database are updated\. We recommend increasing this value if you have a large number of DAGs, or complex DAGs\. Increasing this value reduces the load on the *Scheduler* and the database as DAGs are serialized\.   | 
|  v2  |  [core\.min\_serialized\_dag\_fetch\_interval](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#min-serialized-dag-fetch-interval)  |  10  |  The number of seconds a serialized DAG is re\-fetched from the database when already loaded in the DagBag\.  |  You can use this option to free up resources by **increasing** the number of seconds a serialized DAG is re\-fetched\. The value must be higher than the value specified in `core.min_serialized_dag_update_interval` to reduce database "write" rates\. Increasing this value reduces the load on the *Web server* and the database as DAGs are serialized\.   | 

------
#### [ Apache Airflow v1 ]


| Airflow version | Airflow configuration option | Default | Description | Use case | 
| --- | --- | --- | --- | --- | 
|  v1  |  [core\.dag\_file\_processor\_timeout](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#dag-file-processor-timeout)   |  50 seconds  |  The number of seconds before the *DagFileProcessor* times out processing a DAG file\.  |  You can use this option to free up resources by **increasing** the time it takes before the *DagFileProcessor* times out\. We recommend increasing this value if you're seeing timeouts in your DAG processing logs that result in no viable DAGs being loaded\.   | 
|  v1  |  [core\.min\_serialized\_dag\_update\_interval](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#min-serialized-update-dag-update-interval)  |  30  |  The minimum number of seconds after which serialized DAGs in the database are updated\.  |  You can use this option to free up resources by **increasing** the number of seconds after which serialized DAGs in the database are updated\. We recommend increasing this value if you have a large number of DAGs, or complex DAGs\. Increasing this value reduces the load on the *Scheduler* and the database as DAGs are serialized\.   | 
|  v1  |  [core\.min\_serialized\_dag\_fetch\_interval](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#min-serialized-dag-fetch-interval)  |  10  |  The number of seconds a serialized DAG is re\-fetched from the database when already loaded in the DagBag\.  |  You can use this option to free up resources by **increasing** the number of seconds a serialized DAG is re\-fetched\. The value must be higher than the value specified in `core.min_serialized_dag_update_interval` to reduce database "write" rates\. Increasing this value reduces the load on the *Web server* and the database as DAGs are serialized\.   | 

------

## Tasks<a name="best-practices-tuning-tasks"></a>

The Apache Airflow *Scheduler* and *Workers* are both involved in queuing and de\-queuing tasks\. The *Scheduler* takes parsed tasks ready to schedule from a “None” status to a “Scheduled” status\. The *Executor*, also running on the *Scheduler* container in Fargate, queues those tasks and sets their status to “Queued”\. As the *Worker* containers have capacity, it takes the task from the queue and sets the status to “Running,” which subsequently changes its status to "Success" or "Failed" based upon the success or failure of the task\.

**Note**  
We recommend exercising caution when changing the default values for the parameters in this section\.

### Parameters<a name="best-practices-tuning-tasks-params"></a>

This section describes the configuration options available for Airflow *Tasks* and their use cases\. 

The default Airflow configuration options that Amazon MWAA overrides are marked in *red*\.

------
#### [ Apache Airflow v2 ]


| Airflow version | Airflow configuration option | Default | Description | Use case | 
| --- | --- | --- | --- | --- | 
|  v2  |  [core\.parallelism](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#parallelism)  |  *10000*  |  The maximum number of task instances that can have a status of "Running\."  |  You can use this option to free up resources by **increasing** the number of task instances that can run simultaneously\. The value specified should be the number of available *Workers* "times" the *Workers* task density\. We recommend changing this value only when you're seeing a large number of tasks stuck in the “Running” or “Queued” state\.  | 
|  v2  |  [core\.dag\_concurrency](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#dag-concurrency)  |  *10000*  |  The number of task instances allowed to run concurrently for each DAG\.   |  You can use this option to free up resources by **increasing** the number of task instances allowed to run concurrently\. For example, if you have one hundred DAGs with ten parallel tasks, and you want all DAGs to run concurrently, you can calculate the maximum parallelism as the number of available *Workers* "times" the *Workers* task density in `celery.worker_concurrency`, divided by the number of DAGs \(e\.g\. 100\)\.  | 
|  v2  |  [celery\.worker\_concurrency](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#worker-concurrency)  |  N/A  |  Amazon MWAA overrides the Airflow base install for this option to scale *Workers* as part of its autoscaling component\.  |  Any value specified for this option is ignored\.  | 
|  v2  |  [celery\.worker\_autoscale](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#worker-autoscale)  |  **mw1\.small** \- *5,5* **mw1\.medium** \- *10,10* **mw1\.large** \- *20,20*  |  The task concurrency for *Workers*\.  |  You can use this option to free up resources by **reducing** the `minimum`, `maximum` task concurrency of *Workers*\. The values specified in `minimum`, `maximum` must be the same\. *Workers* accept up to the `maximum` concurrent tasks configured, regardless of whether there are sufficient resources to do so\. If tasks are scheduled without sufficient resources, the tasks immediately fail\. We recommend changing this value for resource\-intensive tasks by reducing the values to be less than the defaults to allow more capacity per task\.  | 

------
#### [ Apache Airflow v1 ]


| Airflow version | Airflow configuration option | Default | Description | Use case | 
| --- | --- | --- | --- | --- | 
|  v1\.0\.12  |  [core\.parallelism](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#parallelism)  |  *10000*  |  The maximum number of task instances that can have a status of "Running\."  |  You can use this option to free up resources by **increasing** the number of task instances that can run simultaneously\. The value specified should be the number of available *Workers* "times" the *Workers* task density\. We recommend changing this value only when you're seeing a large number of tasks stuck in the “Running” or “Queued” state\.  | 
|  v1  |  [core\.dag\_concurrency](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#dag-concurrency)  |  *10000*  |  The number of task instances allowed to run concurrently for each DAG\.   |  You can use this option to free up resources by **increasing** the number of task instances allowed to run concurrently\. For example, if you have one hundred DAGs with ten parallel tasks, and you want all DAGs to run concurrently, you can calculate the maximum parallelism as the number of available *Workers* "times" the *Workers* task density in `celery.worker_concurrency`, divided by the number of DAGs \(e\.g\. 100\)\.  | 
|  v1  |  [celery\.worker\_concurrency](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#worker-concurrency)  |  *\-*  |  Amazon MWAA overrides the Airflow base install for this option to scale *Workers* as part of its autoscaling component\.  |  Any value specified for this option is ignored\.  | 
|  v1  |  [celery\.worker\_autoscale](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#worker-autoscale)  |  *5,5*  |  The task concurrency for *Workers*\.  |  You can use this option to free up resources by **reducing** the `minimum`, `maximum` task concurrency of *Workers*\. The values specified in `minimum`, `maximum` must be the same\. *Workers* accept up to the `maximum` concurrent tasks configured, regardless of whether there are sufficient resources to do so\. If tasks are scheduled without sufficient resources, the tasks immediately fail\. We recommend changing this value for resource\-intensive tasks by reducing the values to be less than the defaults to allow more capacity per task\.  | 

------

## Extending Apache Airflow<a name="best-practices-tuning-extending"></a>

You can use Smart Sensors in Apache Airflow v2 and above to free up task *slots* by consolidating multiple instances of small and light\-weight sensors into a single process\. To learn more, see [Smart Sensors](https://airflow.apache.org/docs/apache-airflow/2.3.4/concepts/smart-sensors.html)\.

Alternatively, sensor tasks can be broken\-up to free task slots for other activities\. For example, rather than configuring a sensor with a timeout of 600 seconds and a single retry, the same sensor could be configured with a timeout of 60 and 10 retries\. The sensor would function the same; however, when it's rescheduling, its task slot is freed for other activities\.
