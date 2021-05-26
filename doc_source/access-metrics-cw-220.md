# Accessing Apache Airflow v2\.0\.2 metrics for an Amazon MWAA environment in CloudWatch<a name="access-metrics-cw-220"></a>

Apache Airflow v2\.0\.2 is already set\-up to collect and send [StatsD](https://github.com/etsy/statsd) metrics for an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment to Amazon CloudWatch\. The complete list of metrics sent is available on the [Apache Airflow v2\.0\.2 metrics](https://airflow.apache.org/docs/apache-airflow/2.0.2/logging-monitoring/metrics.html) in the *Apache Airflow reference guide*\. This page describes the Apache Airflow metrics available in CloudWatch, and how to access metrics in the CloudWatch console\.

**Contents**
+ [Terms](#access-metrics-cw-terms-v202)
+ [Dimensions](#metrics-dimensions-v202)
+ [Accessing metrics in the CloudWatch console](#access-metrics-cw-console-v202)
+ [Apache Airflow metrics available in CloudWatch](#available-metrics-cw-v202)
  + [Apache Airflow Counters](#counters-metrics-v202)
  + [Apache Airflow Gauges](#gauges-metrics-v202)
  + [Apache Airflow Timers](#timers-metrics-v202)

## Terms<a name="access-metrics-cw-terms-v202"></a>

**Namespace**  
A namespace is a container for the CloudWatch metrics of an AWS service\. For Amazon MWAA, the namespace is *AmazonMWAA*\.

**CloudWatch metrics**  
A CloudWatch metric represents a time\-ordered set of data points that are specific to CloudWatch\. 

**Apache Airflow metrics**  
The [Apache Airflow v2\.0\.2 metrics](https://airflow.apache.org/docs/apache-airflow/2.0.2/logging-monitoring/metrics.html ) specific to Apache Airflow\. 

**Dimension**  
A dimension is a name/value pair that is part of the identity of a metric\. 

**Unit**  
A statistic has a unit of measure\. For Amazon MWAA, units include *Count*, *Seconds*, and *Milliseconds*\. For Amazon MWAA, units are set based on the units in the original Airflow metrics\.

## Dimensions<a name="metrics-dimensions-v202"></a>

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

## Accessing metrics in the CloudWatch console<a name="access-metrics-cw-console-v202"></a>

This section describes how to access performance metrics in CloudWatch for a specific DAG\.

**To view performance metrics for a dimension**

1. Open the [Metrics page](https://console.aws.amazon.com/cloudwatch/home#metricsV2:graph=~()) on the CloudWatch console\.

1. Use the AWS Region selector to select your region\.

1. Choose the **AmazonMWAA** namespace\.

1. In the **All metrics** tab, select a dimension\. For example, *DAG, Environment*\.

1. Choose a CloudWatch metric for a dimension\. For example, *TaskInstanceSuccesses* or *TaskInstanceDuration*\. Choose **Graph all search results**\.

1. Choose the **Graphed metrics** tab to view performance statistics for Apache Airflow metrics, such as *DAG, Environment, Task*\.

## Apache Airflow metrics available in CloudWatch<a name="available-metrics-cw-v202"></a>

This section describes the Apache Airflow metrics and dimensions sent to CloudWatch\. 

### Apache Airflow Counters<a name="counters-metrics-v202"></a>

The Apache Airflow metrics in this section contain data about [Apache Airflow *Counters*](https://airflow.apache.org/docs/apache-airflow/2.0.2/logging-monitoring/metrics.html#counters)\. 


| CloudWatch metric | Apache Airflow metric | Unit | Dimension | 
| --- | --- | --- | --- | 
|  CriticalSectionBusy  |  scheduler\.critical\_section\_busy  |  Count  |  Function, Scheduler  | 
|  DagBagSize  |  dagbag\_size  |  Count  |  Function, DAG Processing  | 
|  DagCallbackExceptions  |  dag\.callback\_exceptions  |  Count  |  DAG, All  | 
|  FailedSLAEmailAttempts  |  sla\_email\_notification\_failure  |  Count  |  Function, Scheduler  | 
|  JobEnd  |  \{job\_name\}\_end  |  Count  |  Job, \{job\_name\}  | 
|  JobHeartbeatFailure  |  \{job\_name\}\_heartbeat\_failure   |  Count  |  Job, \{job\_name\}  | 
|  JobStart  |  \{job\_name\}\_start  |  Count  |  Job, \{job\_name\}  | 
|  ManagerStalls  |  dag\_processing\.manager\_stalls  |  Count  |  Function, DAG Processing  | 
|  OperatorFailures  |  operator\_failures\_\{operator\_name\}  |  Count  |  Operator, \{operator\_name\}  | 
|  OperatorSuccesses  |  operator\_successes\_\{operator\_name\}  |  Count  |  Operator, \{operator\_name\}  | 
|  Processes  |  dag\_processing\.processes  |  Count  |  Function, DAG Processing  | 
|  SchedulerHeartbeat  |  scheduler\_heartbeat  |  Count  |  Function, Scheduler  | 
|  TasksKilledExternally  |  scheduler\.tasks\.killed\_externally  |  Count  |  Function, Scheduler  | 
|  TaskTimeoutError  |  celery\.task\_timeout\_error  |  Count  |  Function, Celery  | 
|  TaskInstanceCreatedUsingOperator  |  task\_instance\_created\-\{operator\_name\}  |  Count  |  Operator, \{operator\_name\}  | 
|  TaskInstancePreviouslySucceeded  |  previously\_succeeded  |  Count  |  DAG, All Task, All  | 
|  TaskInstanceFailures  |  ti\_failures  |  Count  |  DAG, All Task, All  | 
|  TaskInstanceFinished  |  ti\.finish\.\{dag\_id\}\.\{task\_id\}\.\{state\}  |  Count  |  DAG, All Task, All State, \{state\}  | 
|  TaskInstanceStarted  |  ti\.start\.\{dag\_id\}\.\{task\_id\}  |  Count  |  DAG, All Task, All  | 
|  TaskInstanceSuccesses  |  ti\_successes  |  Count  |  DAG, All Task, All  | 
|  TaskRemovedFromDAG  |  task\_removed\_from\_dag\.\{dag\_id\}  |  Count  |  DAG, \{dag\_id\}  | 
|  TaskRestoredToDAG  |  task\_restored\_to\_dag\.\{dag\_id\}  |  Count  |  DAG, \{dag\_id\}  | 
|  ZombiesKilled  |  zombies\_killed  |  Count  |  DAG, All Task, All  | 

### Apache Airflow Gauges<a name="gauges-metrics-v202"></a>

The Apache Airflow metrics in this section contain data about [Apache Airflow *Gauges*](https://airflow.apache.org/docs/apache-airflow/2.0.2/logging-monitoring/metrics.html#gauges)\. 


| CloudWatch metric | Apache Airflow metric | Unit | Dimension | 
| --- | --- | --- | --- | 
|  DAGFileRefreshError  |  dag\_file\_refresh\_error  |  Count  |  Function, DAG Processing  | 
|  ImportErrors  |  dag\_processing\.import\_errors  |  Count  |  Function, DAG Processing  | 
|  ExceptionFailures  |  smart\_sensor\_operator\.exception\_failures  |  Count  |  Function, Smart Sensor Operator  | 
|  ExecutedTasks  |  smart\_sensor\_operator\.executed\_tasks  |  Count  |  Function, Smart Sensor Operator  | 
|  InfraFailures  |  smart\_sensor\_operator\.infra\_failures  |  Count  |  Function, Smart Sensor Operator  | 
|  LoadedTasks  |  smart\_sensor\_operator\.loaded\_tasks  |  Count  |  Function, Smart Sensor Operator  | 
|  TotalParseTime  |  dag\_processing\.total\_parse\_time  |  Seconds  |  Function, DAG Processing  | 
|  DAGFileProcessingLastRunSecondsAgo  |  dag\_processing\.last\_run\.seconds\_ago\.\{dag\_filename\}  |  Seconds  |  DAG Filename, \{dag\_filename\}  | 
|  OpenSlots  |  executor\.open\_slots  |  Count  |  Function, Executor  | 
|  OrphanedTasksAdopted  |  scheduler\.orphaned\_tasks\.adopted  |  Count  |  Function, Scheduler  | 
|  OrphanedTasksCleared  |  scheduler\.orphaned\_tasks\.cleared  |  Count  |  Function, Scheduler  | 
|  PokedExceptions  |  smart\_sensor\_operator\.poked\_exception  |  Count  |  Function, Smart Sensor Operator  | 
|  PokedSuccess  |  smart\_sensor\_operator\.poked\_success  |  Count  |  Function, Smart Sensor Operator  | 
|  PokedTasks  |  smart\_sensor\_operator\.poked\_tasks  |  Count  |  Function, Smart Sensor Operator  | 
|  PoolFailures  |  pool\.open\_slots\.\{pool\_name\}  |  Count  |  Pool, \{pool\_name\}  | 
|  PoolStarvingTasks  |  pool\.starving\_tasks\.\{pool\_name\}  |  Count  |  Pool, \{pool\_name\}  | 
|  PoolOpenSlots  |  pool\.open\_slots\.\{pool\_name\}  |  Count  |  Pool, \{pool\_name\}  | 
|  PoolQueuedSlots  |  pool\.queued\_slots\.\{pool\_name\}  |  Count  |  Pool, \{pool\_name\}  | 
|  PoolRunningSlots  |  pool\.running\_slots\.\{pool\_name\}  |  Count  |  Pool, \{pool\_name\}  | 
|  ProcessorTimeouts  |  dag\_processing\.processor\_timeouts  |  Count  |  Function, DAG Processing  | 
|  QueuedTasks  |  executor\.queued\_tasks  |  Count  |  Function, Executor  | 
|  RunningTasks  |  executor\.running\_tasks  |  Count  |  Function, Executor  | 
|  TasksExecutable  |  scheduler\.tasks\.executable  |  Count  |  Function, Scheduler  | 
|  TasksPending  |  scheduler\.tasks\.pending  |  Count  |  Function, Scheduler  | 
|  TasksRunning  |  scheduler\.tasks\.running  |  Count  |  Function, Scheduler  | 
|  TasksStarving  |  scheduler\.tasks\.starving  |  Count  |  Function, Scheduler  | 
|  TasksWithoutDagRun  |  scheduler\.tasks\.without\_dagrun  |  Count  |  Function, Scheduler  | 

### Apache Airflow Timers<a name="timers-metrics-v202"></a>

The Apache Airflow metrics in this section contain data about [Apache Airflow *Timers*](https://airflow.apache.org/docs/apache-airflow/2.0.2/logging-monitoring/metrics.html#timers)\. 


| CloudWatch metric | Apache Airflow metric | Unit | Dimension | 
| --- | --- | --- | --- | 
|  CollectDBDags  |  collect\_db\_dags  |  Milliseconds  |  Function, DAG Processing  | 
|  CriticalSectionDuration  |  scheduler\.critical\_section\_duration  |  Milliseconds  |  Function, Scheduler  | 
|  DAGDependencyCheck  |  dagrun\.dependency\-check\.\{dag\_id\}  |  Milliseconds  |  DAG, \{dag\_id\}  | 
|  DAGDurationFailed  |  dagrun\.duration\.failed\.\{dag\_id\}  |  Milliseconds  |  DAG, \{dag\_id\}  | 
|  DAGDurationSuccess  |  dagrun\.duration\.success\.\{dag\_id\}  |  Milliseconds  |  DAG, \{dag\_id\}  | 
|  DAGFileProcessingLastDuration  |  dag\_processing\.last\_duration\.\{dag\_filename\}  |  Seconds  |  DAG Filename, \{dag\_filename\}  | 
|  DAGScheduleDelay  |  dagrun\.schedule\_delay\.\{dag\_id\}  |  Milliseconds  |  DAG, \{dag\_id\}  | 
|  FirstTaskSchedulingDelay  |  dagrun\.\{dag\_id\}\.first\_task\_scheduling\_delay  |  Milliseconds  |  DAG, \{dag\_id\}  | 
|  TaskInstanceDuration  |  dag\.\{dag\_id\}\.\{task\_id\}\.duration  |  Milliseconds  |  DAG, \{dag\_id\} Task, \{task\_id\}  | 