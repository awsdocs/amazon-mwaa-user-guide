# Monitoring dashboards and alarms on Amazon MWAA<a name="monitoring-dashboard"></a>

You can create a custom dashboard in Amazon CloudWatch and add alarms for a particular metric to monitor the health status of an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment\. When an alarm is on a dashboard, it turns red when it is in the `ALARM` state, making it easier for you to monitor the health of an Amazon MWAA environment proactively\. 

Apache Airflow exposes metrics for a number of processes, including the number of DAG processes, DAG bag size, currently running tasks, task failures, and successes\. When you create an environment, Airflow is configured to automatically send metrics for an Amazon MWAA environment to CloudWatch\. This page describes how to create a health status dashboard for the Airflow metrics in CloudWatch for an Amazon MWAA environment\.

**Contents**
+ [Metrics](#monitoring-dashboard-metrics)
+ [Alarm states overview](#monitoring-dashboard-states)
+ [Example custom dashboards and alarms](#monitoring-dashboard-custom)
  + [About these metrics](#monitoring-dashboard-custom-about)
  + [About the dashboard](#monitoring-dashboard-custom-about-dash)
  + [Using AWS tutorials](#monitoring-dashboard-tutorials)
  + [Using AWS CloudFormation](#monitoring-dashboard-cfn)
+ [Deleting metrics and dashboards](#monitoring-dashboard-delete)
+ [What's next?](#monitoring-dashboard-next-up)

## Metrics<a name="monitoring-dashboard-metrics"></a>

You can create a custom dashboard and alarm for any of the metrics available for your Apache Airflow version\. Each metric corresponds to an Airflow key performance indicator \(KPI\)\. To view a list of metrics, see:
+ [Apache Airflow v2 environment metrics in CloudWatch](access-metrics-cw-202.md)
+ [Apache Airflow v1 environment metrics in CloudWatch](access-metrics-cw-110.md)

## Alarm states overview<a name="monitoring-dashboard-states"></a>

A metric alarm has the following possible states:
+ OK – The metric or expression is within the defined threshold\.
+ ALARM – The metric or expression is outside of the defined threshold\.
+ INSUFFICIENT\_DATA – The alarm has just started, the metric is not available, or not enough data is available for the metric to determine the alarm state\.

## Example custom dashboards and alarms<a name="monitoring-dashboard-custom"></a>

You can build a custom monitoring dashboard that displays charts of selected metrics for your Amazon MWAA environment\. 

### About these metrics<a name="monitoring-dashboard-custom-about"></a>

The following list describes each of the metrics created in the custom dashboard by the tutorial and template definitions in this section\.
+ *QueuedTasks* \- The number of tasks with queued state\. Corresponds to the `executor.queued_tasks` Airflow metric\.
+ *TasksPending* \- The number of tasks pending in executor\. Corresponds to the `scheduler.tasks.pending` Airflow metric\.
+ *RunningTasks* \- The number of tasks running in executor\. Corresponds to the `executor.running_tasks` Airflow metric\.
+ *SchedulerHeartbeat* \- The number of check\-ins Airflow performs on the scheduler job\. Corresponds to the `scheduler_heartbeat` Airflow metrics\.
+ *TotalParseTime* \- The number of seconds taken to scan and import all DAG files once\. Corresponds to the `dag_processing.total_parse_time` Airflow metric\.

### About the dashboard<a name="monitoring-dashboard-custom-about-dash"></a>

The following image shows the monitoring dashboard created by the tutorial and template definition in this section\. 

![\[This image shows where to find the Private network option on the Amazon MWAA console.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/cw-dashboard.png)

### Using AWS tutorials<a name="monitoring-dashboard-tutorials"></a>

You can use the following AWS tutorial to automatically create a health status dashboard for any Amazon MWAA environments that are currently deployed\. It also creates CloudWatch alarms for unhealthy workers and scheduler heartbeat failures across all Amazon MWAA environments\.
+ [CloudWatch Dashboard Automation for Amazon MWAA](https://github.com/aws-samples/mwaa-dashboard)

### Using AWS CloudFormation<a name="monitoring-dashboard-cfn"></a>

You can use the AWS CloudFormation template definition in this section to create a monitoring dashboard in CloudWatch, then add alarms on the CloudWatch console to receive notifications when a metric surpasses a particular threshold\. To create the stack using this template definition, see [Creating a stack on the AWS CloudFormation console](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html)\. To add an alarm to the dashboard, see [Using alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/AlarmThatSendsEmail.html)\. 

```
AWSTemplateFormatVersion: "2010-09-09"
Description: Creates MWAA Cloudwatch Dashboard
Parameters:
  DashboardName:
    Description: Enter the name of the CloudWatch Dashboard
    Type: String
  EnvironmentName:
    Description: Enter the name of the MWAA Environment
    Type: String    
Resources:
  BasicDashboard:
    Type: AWS::CloudWatch::Dashboard
    Properties:
      DashboardName: !Ref DashboardName
      DashboardBody:
        Fn::Sub: '{
              "widgets": [
                  {
                      "type": "metric",
                      "x": 0,
                      "y": 0,
                      "width": 12,
                      "height": 6,
                      "properties": {
                          "view": "timeSeries",
                          "stacked": true,
                          "metrics": [
                              [
                                  "AmazonMWAA",
                                  "QueuedTasks",
                                  "Function",
                                  "Executor",
                                  "Environment",
                                  "${EnvironmentName}"
                              ]
                          ],
                          "region": "${AWS::Region}",
                          "title": "QueuedTasks ${EnvironmentName}",
                          "period": 300
                      }
                  },
                  {
                      "type": "metric",
                      "x": 0,
                      "y": 6,
                      "width": 12,
                      "height": 6,
                      "properties": {
                          "view": "timeSeries",
                          "stacked": true,
                          "metrics": [
                              [
                                  "AmazonMWAA",
                                  "RunningTasks",
                                  "Function",
                                  "Executor",
                                  "Environment",
                                  "${EnvironmentName}"
                              ]
                          ],
                          "region": "${AWS::Region}",
                          "title": "RunningTasks ${EnvironmentName}",
                          "period": 300
                      }
                  },
                  {
                      "type": "metric",
                      "x": 12,
                      "y": 6,
                      "width": 12,
                      "height": 6,
                      "properties": {
                          "view": "timeSeries",
                          "stacked": true,
                          "metrics": [
                              [
                                  "AmazonMWAA",
                                  "SchedulerHeartbeat",
                                  "Function",
                                  "Scheduler",
                                  "Environment",
                                  "${EnvironmentName}"
                              ]
                          ],
                          "region": "${AWS::Region}",
                          "title": "SchedulerHeartbeat ${EnvironmentName}",
                          "period": 300
                      }
                  },
                  {
                      "type": "metric",
                      "x": 12,
                      "y": 0,
                      "width": 12,
                      "height": 6,
                      "properties": {
                          "view": "timeSeries",
                          "stacked": true,
                          "metrics": [
                              [
                                  "AmazonMWAA",
                                  "TasksPending",
                                  "Function",
                                  "Scheduler",
                                  "Environment",
                                  "${EnvironmentName}"
                              ]
                          ],
                          "region": "${AWS::Region}",
                          "title": "TasksPending ${EnvironmentName}",
                          "period": 300
                      }
                  },
                  {
                      "type": "metric",
                      "x": 0,
                      "y": 12,
                      "width": 24,
                      "height": 6,
                      "properties": {
                          "view": "timeSeries",
                          "stacked": true,
                          "region": "${AWS::Region}",
                          "metrics": [
                              [
                                  "AmazonMWAA",
                                  "TotalParseTime",
                                  "Function",
                                  "DAG Processing",
                                  "Environment",
                                  "${EnvironmentName}"
                              ]
                          ],
                          "title": "TotalParseTime  ${EnvironmentName}",
                          "period": 300
                      }
                  }
              ]
          }'
```

## Deleting metrics and dashboards<a name="monitoring-dashboard-delete"></a>

If you delete an Amazon MWAA environment, the corresponding dashboard is also deleted\. CloudWatch metrics are stored for fifteen \(15\) months and can not be deleted\. The CloudWatch console limits the search of metrics to two \(2\) weeks after a metric is last ingested to ensure that the most up to date instances are shown for your Amazon MWAA environment\. To learn more, see [Amazon CloudWatch FAQs](https://aws.amazon.com/cloudwatch/faqs/)\.

## What's next?<a name="monitoring-dashboard-next-up"></a>
+ Learn how to create a DAG that queries the Amazon Aurora PostgreSQL metadata database for your environment and publishes custom metrics to CloudWatch in [Using a DAG to write custom metrics in CloudWatch](samples-custom-metrics.md)\.