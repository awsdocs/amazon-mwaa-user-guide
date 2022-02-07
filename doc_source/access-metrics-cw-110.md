# Apache Airflow v1 environment metrics in CloudWatch<a name="access-metrics-cw-110"></a>

Apache Airflow v1\.10\.12 is already set\-up to collect and send [StatsD](https://github.com/etsy/statsd) metrics for an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment to Amazon CloudWatch\. The complete list of metrics sent is available on the [Apache Airflow v1\.10\.12 metrics](https://airflow.apache.org/docs/apache-airflow/1.10.12/metrics.html) in the *Apache Airflow reference guide*\. This page describes the Apache Airflow metrics available in CloudWatch, and how to access metrics in the CloudWatch console\.

**Contents**
+ [Terms](#access-metrics-cw-terms-v11012)
+ [Dimensions](#metrics-dimensions-v11012)
+ [Accessing metrics in the CloudWatch console](#access-metrics-cw-console-v11012)
+ [Apache Airflow metrics available in CloudWatch](#available-metrics-cw-v11012)
  + [Apache Airflow Counters](#counters-metrics-v11012)
  + [Apache Airflow Gauges](#gauges-metrics-v11012)
  + [Apache Airflow Timers](#timers-metrics-v11012)
+ [What's next?](#mwaa-metrics11012-next-up)

## Terms<a name="access-metrics-cw-terms-v11012"></a>

**Namespace**  
A namespace is a container for the CloudWatch metrics of an AWS service\. For Amazon MWAA, the namespace is *AmazonMWAA*\.

**CloudWatch metrics**  
A CloudWatch metric represents a time\-ordered set of data points that are specific to CloudWatch\. 

**Apache Airflow metrics**  
The [Apache Airflow v1\.10\.12 metrics](https://airflow.apache.org/docs/apache-airflow/1.10.12/metrics.html) specific to Apache Airflow\. 

**Dimension**  
A dimension is a name/value pair that is part of the identity of a metric\. 

**Unit**  
A statistic has a unit of measure\. For Amazon MWAA, units include *Count*, *Seconds*, and *Milliseconds*\. For Amazon MWAA, units are set based on the units in the original Airflow metrics\.

## Dimensions<a name="metrics-dimensions-v11012"></a>

This section describes the CloudWatch *Dimensions* grouping for Apache Airflow metrics in CloudWatch\.


| Dimension | Description | 
| --- | --- | 
|  DAG  |  Indicates a specific Apache Airflow DAG name\.  | 
|  DAG Filename  |  Indicates a specific Apache Airflow DAG file name\.  | 
|  Function  |  This dimension is used to improve the grouping of metrics in CloudWatch\.   | 
|  Job  |  Indicates an Apache Airflow *Job* run by the *Scheduler*\. Always has a value of *Job*\.  | 
|  Operator  |  Indicates a specific Apache Airflow operator\.  | 
|  Pool  |  Indicates a specific Apache Airflow *Worker Pool*\.  | 
|  Task  |  Indicates a specific Apache Airflow task\.  | 

## Accessing metrics in the CloudWatch console<a name="access-metrics-cw-console-v11012"></a>

This section describes how to access performance metrics in CloudWatch for a specific DAG\.

**To view performance metrics for a dimension**

1. Open the [Metrics page](https://console.aws.amazon.com/cloudwatch/home#metricsV2:graph=~()) on the CloudWatch console\.

1. Use the AWS Region selector to select your region\.

1. Choose the **AmazonMWAA** namespace\.

1. In the **All metrics** tab, select a dimension\. For example, *DAG, Environment*\.

1. Choose a CloudWatch metric for a dimension\. For example, *TaskInstanceSuccesses* or *TaskInstanceDuration*\. Choose **Graph all search results**\.

1. Choose the **Graphed metrics** tab to view performance statistics for Apache Airflow metrics, such as *DAG, Environment, Task*\.

## Apache Airflow metrics available in CloudWatch<a name="available-metrics-cw-v11012"></a>

This section describes the Apache Airflow metrics and dimensions sent to CloudWatch\. 

### Apache Airflow Counters<a name="counters-metrics-v11012"></a>

The Apache Airflow metrics in this section contain data about [Apache Airflow *Counters*](https://airflow.apache.org/docs/apache-airflow/1.10.12/metrics.html#counters)\. 


| CloudWatch metric | Apache Airflow metric | Unit | Dimension | 
| --- | --- | --- | --- | 
|  DagBagSize  |  dagbag\_size  |  Count  |  Function, DAG Processing  | 
|  JobEnd  |  \{job\_name\}\_end  |  Count  |  Job, \{job\_name\}  | 
|  JobHeartbeatFailure  |  \{job\_name\}\_heartbeat\_failure   |  Count  |  Job, \{job\_name\}  | 
|  JobStart  |  \{job\_name\}\_start  |  Count  |  Job, \{job\_name\}  | 
|  OperatorFailures  |  operator\_failures\_\{operator\_name\}  |  Count  |  Operator, \{operator\_name\}  | 
|  OperatorSuccesses  |  operator\_successes\_\{operator\_name\}  |  Count  |  Operator, \{operator\_name\}  | 
|  Processes  |  dag\_processing\.processes  |  Count  |  Function, DAG Processing  | 
|  SchedulerHeartbeat  |  scheduler\_heartbeat  |  Count  |  Function, Scheduler  | 
|  TasksKilledExternally  |  scheduler\.tasks\.killed\_externally  |  Count  |  Function, Scheduler  | 
|  TaskInstanceCreatedUsingOperator  |  task\_instance\_created\-\{operator\_name\}  |  Count  |  Operator, \{operator\_name\}  | 
|  TaskInstanceFailures  |  ti\_failures  |  Count  |  DAG, All Task, All  | 
|  TaskInstanceSuccesses  |  ti\_successes  |  Count  |  DAG, All Task, All  | 
|  TaskRemovedFromDAG  |  task\_removed\_from\_dag\.\{dag\_id\}  |  Count  |  DAG, \{dag\_id\}  | 
|  TaskRestoredToDAG  |  task\_restored\_to\_dag\.\{dag\_id\}  |  Count  |  DAG, \{dag\_id\}  | 
|  ZombiesKilled  |  zombies\_killed  |  Count  |  DAG, All Task, All  | 

### Apache Airflow Gauges<a name="gauges-metrics-v11012"></a>

The Apache Airflow metrics in this section contain data about [Apache Airflow *Gauges*](https://airflow.apache.org/docs/apache-airflow/1.10.12/metrics.html#gauges)\. 


| CloudWatch metric | Apache Airflow metric | Unit | Dimension | 
| --- | --- | --- | --- | 
|  DAGFileRefreshError  |  dag\_file\_refresh\_error  |  Count  |  Function, DAG Processing  | 
|  ImportErrors  |  dag\_processing\.import\_errors  |  Count  |  Function, DAG Processing  | 
|  TotalParseTime  |  dag\_processing\.total\_parse\_time  |  Seconds  |  Function, DAG Processing  | 
|  DAGFileProcessingLastRunSecondsAgo  |  dag\_processing\.last\_run\.seconds\_ago\.\{dag\_filename\}  |  Seconds  |  DAG Filename, \{dag\_filename\}  | 
|  OpenSlots  |  executor\.open\_slots  |  Count  |  Function, Executor  | 
|  PoolFailures  |  pool\.open\_slots\.\{pool\_name\}  |  Count  |  Pool, \{pool\_name\}  | 
|  PoolStarvingTasks  |  pool\.starving\_tasks\.\{pool\_name\}  |  Count  |  Pool, \{pool\_name\}  | 
|  PoolUsedSlots  |  pool\.used\_slots\.\{pool\_name\}  |  Count  |  Pool, \{pool\_name\}  | 
|  ProcessorTimeouts  |  dag\_processing\.processor\_timeouts  |  Count  |  Function, DAG Processing  | 
|  QueuedTasks  |  executor\.queued\_tasks  |  Count  |  Function, Executor  | 
|  RunningTasks  |  executor\.running\_tasks  |  Count  |  Function, Executor  | 
|  TasksExecutable  |  scheduler\.tasks\.executable  |  Count  |  Function, Scheduler  | 
|  TasksPending  |  scheduler\.tasks\.pending  |  Count  |  Function, Scheduler  | 
|  TasksRunning  |  scheduler\.tasks\.running  |  Count  |  Function, Scheduler  | 
|  TasksStarving  |  scheduler\.tasks\.starving  |  Count  |  Function, Scheduler  | 
|  TasksWithoutDagRun  |  scheduler\.tasks\.without\_dagrun  |  Count  |  Function, Scheduler  | 

### Apache Airflow Timers<a name="timers-metrics-v11012"></a>

The Apache Airflow metrics in this section contain data about [Apache Airflow *Timers*](https://airflow.apache.org/docs/apache-airflow/1.10.12/metrics.html#timers)\. 


| CloudWatch metric | Apache Airflow metric | Unit | Dimension | 
| --- | --- | --- | --- | 
|  CollectDBDags  |  collect\_db\_dags  |  Milliseconds  |  Function, DAG Processing  | 
|  DAGDependencyCheck  |  dagrun\.dependency\-check\.\{dag\_id\}  |  Milliseconds  |  DAG, \{dag\_id\}  | 
|  DAGDurationFailed  |  dagrun\.duration\.failed\.\{dag\_id\}  |  Milliseconds  |  DAG, \{dag\_id\}  | 
|  DAGDurationSuccess  |  dagrun\.duration\.success\.\{dag\_id\}  |  Milliseconds  |  DAG, \{dag\_id\}  | 
|  DAGFileProcessingLastDuration  |  dag\_processing\.last\_duration\.\{dag\_filename\}  |  Seconds  |  DAG Filename, \{dag\_filename\}  | 
|  DAGScheduleDelay  |  dagrun\.schedule\_delay\.\{dag\_id\}  |  Milliseconds  |  DAG, \{dag\_id\}  | 
|  TaskInstanceDuration  |  dag\.\{dag\_id\}\.\{task\_id\}\.duration  |  Milliseconds  |  DAG, \{dag\_id\} Task, \{task\_id\}  | 

## What's next?<a name="mwaa-metrics11012-next-up"></a>
+ Explore the Amazon MWAA API operation used to publish environment health metrics at [PublishMetrics](https://docs.aws.amazon.com/mwaa/latest/API/API_PublishMetrics.html)\.