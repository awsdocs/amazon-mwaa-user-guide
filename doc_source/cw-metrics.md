# Amazon MWAA metrics<a name="cw-metrics"></a>

Amazon MWAA metrics provide data about the performance and usage of Amazon MWAA in your account\. Some metrics are about your Amazon MWAA environment, and some reflect metrics from Apache Airflow in your environment\. Metrics used in Amazon MWAA are included in the following categories\.
+ **Job\-based metrics** – includes data about Apache Airflow jobs, such as a count of start and end\. This group has the following dimension:
  + **Job** – specifies the job the metrics are about\.
+ **Functional metrics** – includes data about the Amazon MWAA environment, such as DAG processing, task scheduling, and workers\. It does not include metric data for a specific DAG, job, or pool\. This group has the following dimension:
  + **Function, Component** – specifies the component associated with the metric, such as Scheduler\.
+ **DAG\-file based metrics** – includes data specific DAGS, such as processing duration\. This group has the following dimension:
  + **DAG Filename** – specifies the DAG file associated with the metric\.
+ **DAG\-based metrics** – includes any metric specific to a DAG, but not specific to a certain task, such as LoadingDuration for a particular DAG\. This group has the following dimension:
  + **DAG** – specifies the DAG the metric is about\.
+ **Task\-based metrics** – includes data for specific DAG tasks, such as Duration or Successes\. This group has the following dimensions:
  + **DAG** – specifies the DAG the metric is about\.
  + **Task** – specifies the task the metric is about\. This can be “All” if this metric is about all tasks, such as Successes\.
+ **Operator\-based metrics** – includes data about metrics related to specific operators, for example the success rate for a particular operator\. This group has the following dimension:
  + **Operator** – specifies the operator the metric is about\.
+ **Pool\-based metrics** – includes data about a specific pool\. This group has the following dimension:
  + **Pool** – specifies the name of the pool the metric is about\.

## Metrics sent to Amazon CloudWatch<a name="mwaa-cw-metrics"></a>

The following metrics are sent to Amazon CloudWatch metrics:

### Job\-based metrics<a name="job-based-metrics"></a>


| CloudWatch metric | Apache Airflow metric | Unit | Dimension | 
| --- | --- | --- | --- | 
|   JobEnd  |   \{job\_name\}\_end  |   Count  |   Job, \{job\_name\}  | 
|   JobHeartbeatFailure  |   \{job\_name\}\_heartbeat\_failure   |   Count  |   Job, \{job\_name\}  | 
|   JobStart  |   \{job\_name\}\_start  |   Count  |   Job, \{job\_name\}  | 

### Functional metrics<a name="functional-metrics"></a>


| CloudWatch metric | Apache Airflow metric | Unit | Dimension | 
| --- | --- | --- | --- | 
|   CollectDBDags  |   collect\_db\_dags  |   Milliseconds  |   Function, DAG Processing  | 
|   DAGFileRefreshError  |   dag\_file\_refresh\_error  |   Count  |   Function, DAG Processing  | 
|   ImportErrors  |   dag\_processing\.import\_errors  |   Count  |   Function, DAG Processing  | 
|   Processes  |   dag\_processing\.processes  |   Count  |   Function, DAG Processing  | 
|   ProcessorTimeouts  |   dag\_processing\.processor\_timeouts  |   Count  |   Function, DAG Processing  | 
|   TotalParseTime  |   dag\_processing\.total\_parse\_time  |   Seconds  |   Function, DAG Processing  | 
|   DagBagSize  |   dagbag\_size  |   Count  |   Function, DAG Processing  | 
|   OpenSlots  |   executor\.open\_slots  |   Count  |   Function, Executor  | 
|   QueuedTasks  |   executor\.queued\_tasks  |   Count  |   Function, Executor  | 
|   RunningTasks  |   executor\.running\_tasks  |   Count  |   Function, Executor  | 
|   SchedulerHeartbeat  |   scheduler\_heartbeat  |   Count  |   Function, Scheduler  | 
|   TasksExecutable  |   scheduler\.tasks\.executable  |   Count  |   Function, Scheduler  | 
|   TasksKilledExternally  |   scheduler\.tasks\.killed\_externally  |   Count  |   Function, Scheduler  | 
|   TasksPending  |   scheduler\.tasks\.pending  |   Count  |   Function, Scheduler  | 
|   TasksRunning  |   scheduler\.tasks\.running  |   Count  |   Function, Scheduler  | 
|   TasksStarving  |   scheduler\.tasks\.starving  |   Count  |   Function, Scheduler  | 
|   TasksWithoutDagRun  |   scheduler\.tasks\.without\_dagrun  |   Count  |   Function, Scheduler  | 

### DAG File\-based metrics<a name="dag-file-based-metrics"></a>


| CloudWatch metric | Apache Airflow metric | Unit | Dimension | 
| --- | --- | --- | --- | 
|   DAGFileProcessingLastDuration  |   dag\_processing\.last\_duration\.\{dag\_filename\}  |   Seconds  |   DAG Filename, \{dag\_filename\}  | 
|   DAGFileProcessingLastRunSecondsAgo  |   dag\_processing\.last\_run\.seconds\_ago\.\{dag\_filename\}  |   Seconds  |   DAG Filename, \{dag\_filename\}  | 

### DAG\-based metrics<a name="dag-based-metrics"></a>


| CloudWatch metric | Apache Airflow metric | Unit | Dimension | 
| --- | --- | --- | --- | 
|   DAGDependencyCheck  |   dagrun\.dependency\-check\.\{dag\_id\}  |   Milliseconds  |   DAG, \{dag\_id\}  | 
|   DAGDurationFailed  |   dagrun\.duration\.failed\.\{dag\_id\}  |   Milliseconds  |   DAG, \{dag\_id\}  | 
|   DAGDurationSuccess  |   dagrun\.duration\.success\.\{dag\_id\}  |   Milliseconds  |   DAG, \{dag\_id\}  | 
|   DAGScheduleDelay  |   dagrun\.schedule\_delay\.\{dag\_id\}  |   Milliseconds  |   DAG, \{dag\_id\}  | 
|   TaskRemovedFromDAG  |   task\_removed\_from\_dag\.\{dag\_id\}  |   Count  |   DAG, \{dag\_id\}  | 
|   TaskRestoredToDAG  |   task\_restored\_to\_dag\.\{dag\_id\}  |   Count  |   DAG, \{dag\_id\}  | 

### Task\-based metrics<a name="task-based-metrics"></a>


| CloudWatch metric | Apache Airflow metric | Unit | Dimension | 
| --- | --- | --- | --- | 
|   TaskInstanceDuration  |   dag\.\{dag\_id\}\.\{task\_id\}\.duration  |   Milliseconds  |   DAG, \{dag\_id\} Task, \{task\_id\}  | 
|   TaskInstanceFailures  |   ti\_failures  |   Count  |   DAG, All Task, All  | 
|   TaskInstanceSuccesses  |   ti\_successes  |   Count  |   DAG, All Task, All  | 

### Operator\-based metrics<a name="operator-based-metrics"></a>


| CloudWatch metric | Apache Airflow metric | Unit | Dimension | 
| --- | --- | --- | --- | 
|   OperatorFailures  |   operator\_failures\_\{operator\_name\}  |   Count  |   Operator, \{operator\_name\}  | 
|   OperatorSuccesses  |   operator\_successes\_\{operator\_name\}  |   Count  |   Operator, \{operator\_name\}  | 
|   TaskInstanceCreatedUsingOperator  |   task\_instance\_created\-\{operator\_name\}  |   Count  |   Operator, \{operator\_name\}  | 

### Pool\-based metrics<a name="pool-based-metrics"></a>


| CloudWatch metric | Apache Airflow metric | Unit | Dimension | 
| --- | --- | --- | --- | 
|   PoolFailures  |   pool\.open\_slots\.\{pool\_name\}  |   Count  |   Pool, \{pool\_name\}  | 
|   PoolStarvingTasks  |   pool\.starving\_tasks\.\{pool\_name\}  |   Count  |   Pool, \{pool\_name\}  | 
|   PoolUsedSlots  |   pool\.used\_slots\.\{pool\_name\}  |   Count  |   Pool, \{pool\_name\}  | 