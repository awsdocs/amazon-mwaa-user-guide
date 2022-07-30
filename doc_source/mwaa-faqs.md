# Amazon MWAA frequently asked questions<a name="mwaa-faqs"></a>

This page describes common questions you may encounter when using Amazon Managed Workflows for Apache Airflow \(MWAA\)\.

**Contents**
+ [Supported versions](#q-supported-versions)
  + [What does Amazon MWAA support for Apache Airflow v2?](#airflow-support)
  + [Why are older versions of Apache Airflow not supported?](#airflow-version)
  + [What Python version should I use?](#python-version)
  + [Can I specify more than 25 Apache Airflow Workers?](#scaling-quota)
+ [Use cases](#t-common-questions)
  + [When should I use AWS Step Functions vs\. Amazon MWAA?](#t-step-functions)
+ [Environment specifications](#q-supported-features)
  + [How much task storage is available to each environment?](#worker-storage)
  + [What is the default operating system used for Amazon MWAA environments?](#default-os)
  + [Can I use a custom image for my Amazon MWAA environment?](#custom-image)
  + [Is MWAA HIPAA compliant?](#hipaa-compliance)
  + [Does Amazon MWAA support Spot Instances?](#spot-instances)
  + [Does Amazon MWAA support a custom domain?](#custom-dns)
  + [Can I SSH into my environment?](#ssh-dag)
  + [Why is a self\-referencing rule required on the VPC security group?](#sg-rule)
  + [Can I hide environments from different groups in IAM?](#hide-environments)
  + [Can I store temporary data on the Apache Airflow Worker?](#store-data)
  + [Does Amazon MWAA support shared Amazon VPCs or shared subnets?](#shared-vpc)
+ [Metrics](#q-metrics)
  + [What metrics are used to determine whether to scale Workers?](#metrics-workers)
  + [Can I create custom metrics in CloudWatch?](#metrics-custom)
+ [DAGs, Operators, Connections, and other questions](#q-dags)
  + [Can I use the `PythonVirtualenvOperator`?](#virtual-env-dags)
  + [How long does it take Amazon MWAA to recognize a new DAG file?](#recog-dag)
  + [Why is my DAG file not picked up by Apache Airflow?](#dag-file-error)
  + [Can I remove a `plugins.zip` or `requirements.txt` from an environment?](#remove-plugins-reqs)
  + [Why don't I see my plugins in the Apache Airflow v2\.0\.2 Admin Plugins menu?](#view-plugins-ui)
  + [Can I use AWS Database Migration Service \(DMS\) Operators?](#ops-dms)
+ [Migrating](#q-migrating)
  + [How do I migrate to Amazon MWAA from an on\-premises or a self\-managed Apache Airflow deployment?](#migrate-from-onprem)

## Supported versions<a name="q-supported-versions"></a>

### What does Amazon MWAA support for Apache Airflow v2?<a name="airflow-support"></a>

To learn what Amazon MWAA supports, see [Apache Airflow versions on Amazon Managed Workflows for Apache Airflow \(MWAA\)](airflow-versions.md)\.

### Why are older versions of Apache Airflow not supported?<a name="airflow-version"></a>

We are only supporting the latest \(as of launch\) Apache Airflow version Apache Airflow v1\.10\.12 due to security concerns with older versions\.

### What Python version should I use?<a name="python-version"></a>

The following Apache Airflow versions are supported on Amazon Managed Workflows for Apache Airflow \(MWAA\)\.


| Apache Airflow version | Apache Airflow guide | Apache Airflow constraints | Python version | 
| --- | --- | --- | --- | 
|  v2\.2\.2  |  [Apache Airflow v2\.2\.2 reference guide](https://airflow.apache.org/docs/apache-airflow/2.2.2/index.html)  |  [Apache Airflow v2\.2\.2 constraints file](https://raw.githubusercontent.com/apache/airflow/constraints-2.2.2/constraints-3.7.txt)  |  [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)  | 
|  v2\.0\.2  |  [Apache Airflow v2\.0\.2 reference guide](http://airflow.apache.org/docs/apache-airflow/2.0.2/index.html)  |  [Apache Airflow v2\.0\.2 constraints file](https://raw.githubusercontent.com/apache/airflow/constraints-2.0.2/constraints-3.7.txt)  |  [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)  | 
|  v1\.10\.12  |  [Apache Airflow v1\.10\.12 reference guide](https://airflow.apache.org/docs/apache-airflow/1.10.12/)  |  [Apache Airflow v1\.10\.12 constraints file](https://raw.githubusercontent.com/apache/airflow/constraints-1.10.12/constraints-3.7.txt)  |  [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)  | 

### Can I specify more than 25 Apache Airflow Workers?<a name="scaling-quota"></a>

 Yes\. Although you can specify up to 25 Apache Airflow workers on the Amazon MWAA console, you can configure up to 50 on an environment by requesting a quota increase\. For more information, see [Requesting a quota increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-quota-increase.html)\. 

## Use cases<a name="t-common-questions"></a>

### When should I use AWS Step Functions vs\. Amazon MWAA?<a name="t-step-functions"></a>

1. You can use Step Functions to process individual customer orders, since Step Functions can scale to meet demand for one order or one million orders\.

1. If you’re running an overnight workflow that processes the previous day’s orders, you can use Step Functions or Amazon MWAA\. Amazon MWAA allows you an open source option to abstract the workflow from the AWS resources you're using\.

## Environment specifications<a name="q-supported-features"></a>

### How much task storage is available to each environment?<a name="worker-storage"></a>

The task storage is limited to 10 GB, and is specified by [Amazon ECS Fargate 1\.3](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/fargate-task-storage.html#fargate-task-storage-pv13)\. The amount of RAM is determined by the environment class you specify\. For more information about environment classes, see [Amazon MWAA environment class](environment-class.md)\.

### What is the default operating system used for Amazon MWAA environments?<a name="default-os"></a>

Amazon MWAA environments are created on instances running Amazon Linux AMI\.

### Can I use a custom image for my Amazon MWAA environment?<a name="custom-image"></a>

Custom images are not supported\. Amazon MWAA uses images that are built on Amazon Linux AMI\. Amazon MWAA installs the additional requirements by running `pip3 -r install` for the requirements specified in the requirements\.txt file you add to the Amazon S3 bucket for the environment\.

### Is MWAA HIPAA compliant?<a name="hipaa-compliance"></a>

Amazon MWAA is not currently HIPAA compliant\.

### Does Amazon MWAA support Spot Instances?<a name="spot-instances"></a>

Amazon MWAA does not currently support on\-demand Amazon EC2 Spot Instance types for Apache Airflow\. However, an Amazon MWAA environment can trigger Spot Instances on, for example, Amazon EMR and Amazon EC2\.

### Does Amazon MWAA support a custom domain?<a name="custom-dns"></a>

 To be able to use a custom domain for your Amazon MWAA hostname, do one of the following: 
+  For Amazon MWAA deployments with public web server access, you can use Amazon CloudFront with Lambda@Edge to direct traffic to your environment, and map a custom domain name to CloudFront\. For more information and an example of setting up a custom domain for a public environment, see the [Amazon MWAA custom domain for public web server](https://github.com/aws-samples/amazon-mwaa-examples/tree/main/usecases/mwaa-public-webserver-custom-domain) sample in the Amazon MWAA examples GitHub repository\. 
+  For Amazon MWAA deployments with private web server access, you can use an Application Load Balancer \(ALB\) to direct traffic to Amazon MWAA and map a custom domain name to the ALB\. For more information, see [Using a Load Balancer \(advanced\)](vpc-vpe-access.md#vpc-vpe-access-load-balancer)\. 

### Can I SSH into my environment?<a name="ssh-dag"></a>

While SSH is not supported on a Amazon MWAA environment, it's possible to use a DAG to run bash commands using the `BashOperator`\. For example:

```
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.utils.dates import days_ago
with DAG(dag_id="any_bash_command_dag", schedule_interval=None, catchup=False, start_date=days_ago(1)) as dag:
      cli_command = BashOperator(
          task_id="bash_command",
          bash_command="{{ dag_run.conf['command'] }}"
      )
```

To trigger the DAG in the Apache Airflow UI, use:

```
{ "command" : "your bash command"}
```

### Why is a self\-referencing rule required on the VPC security group?<a name="sg-rule"></a>

By creating a self\-referencing rule, you're restricting the source to the same security group in the VPC, and it's not open to all networks\. To learn more, see [Security in your VPC on Amazon MWAA](vpc-security.md)\.

### Can I hide environments from different groups in IAM?<a name="hide-environments"></a>

You can limit access by specifying an environment name in AWS Identity and Access Management, however, visibility filtering isn't available in the AWS console—if a user can see one environment, they can see all environments\. 

### Can I store temporary data on the Apache Airflow Worker?<a name="store-data"></a>

Your Apache Airflow Operators can store temporary data on the *Workers*\. Apache Airflow *Workers* can access temporary files in the `/tmp` on the Fargate containers for your environment\.

**Note**  
Total task storage is limited to 10 GB, according to [Amazon ECS Fargate 1\.3](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/fargate-task-storage.html#fargate-task-storage-pv13)\. There's no guarantee that subsequent tasks will run on the same Fargate container instance, which might use a different `/tmp` folder\.

### Does Amazon MWAA support shared Amazon VPCs or shared subnets?<a name="shared-vpc"></a>

 Amazon MWAA does not support shared Amazon VPCs or shared subnets\. The Amazon VPC you select when you create an environment should be owned by the account that is attempting to create the environment\. However, you can route traffic from an Amazon VPC in the Amazon MWAA account to a shared VPC\. For more information, and to see an example of routing traffic to a shared Amazon VPC, see [Centralized outbound routing to the internet](https://docs.aws.amazon.com/vpc/latest/tgw/transit-gateway-nat-igw.html) in the *Amazon VPC Transit Gateways Guide*\. 

## Metrics<a name="q-metrics"></a>

### What metrics are used to determine whether to scale Workers?<a name="metrics-workers"></a>

Amazon MWAA monitors the **QueuedTasks** and **RunningTasks** in CloudWatch to determine whether to scale Apache Airflow *Workers * on your environment\. To learn more, see [Monitoring and metrics for Amazon Managed Workflows for Apache Airflow \(MWAA\)](cw-metrics.md)\.

### Can I create custom metrics in CloudWatch?<a name="metrics-custom"></a>

Not on the CloudWatch console\. However, you can create a DAG that writes custom metrics in CloudWatch\. To learn more, see [Using a DAG to write custom metrics in CloudWatch](samples-custom-metrics.md)\.

## DAGs, Operators, Connections, and other questions<a name="q-dags"></a>

### Can I use the `PythonVirtualenvOperator`?<a name="virtual-env-dags"></a>

The `PythonVirtualenvOperator` is not explicitly supported on Amazon MWAA, but you can create a custom plugin that uses the `PythonVirtualenvOperator`\. For sample code, see [Creating a custom plugin for Apache Airflow PythonVirtualenvOperator](samples-virtualenv.md)\.

### How long does it take Amazon MWAA to recognize a new DAG file?<a name="recog-dag"></a>

 DAGs are periodically synchronized from the Amazon S3 bucket to your environment\. If you add a new DAG file, it takes about 300 seconds for Amazon MWAA to start *using* the new file\. If you update an existing DAG, it takes Amazon MWAA about 30 seconds to recognize your updates\. 

 These values, 300 seconds for new DAGs, and 30 seconds for updates to existing DAGs, correspond to Apache Airflow configuration options [https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#dag-dir-list-interval](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#dag-dir-list-interval), and [https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#min-file-process-interval](https://airflow.apache.org/docs/apache-airflow/stable/configurations-ref.html#min-file-process-interval) respectively\. 

### Why is my DAG file not picked up by Apache Airflow?<a name="dag-file-error"></a>

The following are possible solutions for this issue:

1. Check that your execution role has sufficient permissions to your Amazon S3 bucket\. To learn more, see [Amazon MWAA execution role](mwaa-create-role.md)\.

1. Check that the Amazon S3 bucket has *Block Public Access* configured, and *Versioning* enabled\. To learn more, see [Create an Amazon S3 bucket for Amazon MWAA](mwaa-s3-bucket.md)\.

1. Verify the DAG file itself\. For example, be sure that each DAG has a unique DAG ID\.

### Can I remove a `plugins.zip` or `requirements.txt` from an environment?<a name="remove-plugins-reqs"></a>

Currently, there is no way to remove a plugins\.zip or requirements\.txt from an environment once they’ve been added, but we're working on the issue\. In the interim, a workaround is to point to an empty text or zip file, respectively\. To learn more, see [Deleting files on Amazon S3](working-dags-delete.md)\.

### Why don't I see my plugins in the Apache Airflow v2\.0\.2 Admin Plugins menu?<a name="view-plugins-ui"></a>

For security reasons, the Apache Airflow Web server on Amazon MWAA has limited network egress, and does not install plugins nor Python dependencies directly on the Apache Airflow *web server* for version 2\.0\.2 environments\. The plugin that's shown allows Amazon MWAA to authenticate your Apache Airflow users in AWS Identity and Access Management \(IAM\)\.

 To be able to install plugins and Python dependencies directly on the web server, we recommend creaing a new environment with Apache Airflow v2\.2 and above\. Amazon MWAA installs Python dependencies and and custom plugins directly on the web server for Apache Airflow v2\.2 and above\. 

### Can I use AWS Database Migration Service \(DMS\) Operators?<a name="ops-dms"></a>

Amazon MWAA doesn't currently support [DMS Operators](https://airflow.apache.org/docs/apache-airflow-providers-amazon/stable/operators/dms.html)\. Each environment has its own Amazon Aurora PostgreSQL managed by AWS\. 

## Migrating<a name="q-migrating"></a>

### How do I migrate to Amazon MWAA from an on\-premises or a self\-managed Apache Airflow deployment?<a name="migrate-from-onprem"></a>

While each circumstance is different, in general, the recommendation is to move workloads gradually and run them in parallel if possible until you can verify that everything works as expected before you decommission your on\-premises or self\-managed deployment\.
