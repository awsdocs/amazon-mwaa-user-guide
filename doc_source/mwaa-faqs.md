# Amazon MWAA frequently asked questions<a name="mwaa-faqs"></a>

This page describes common questions you may encounter when using Amazon Managed Workflows for Apache Airflow \(MWAA\)\.

**Contents**
+ [Supported versions](#q-supported-versions)
  + [Why are older versions of Apache Airflow not supported?](#airflow-version)
  + [What Python version should I use?](#python-version)
  + [Can I specify more than 25 Apache Airflow Workers?](#scaling-quota)
+ [Use cases](#t-common-questions)
  + [When should I use AWS Step Functions vs\. Amazon MWAA?](#t-step-functions)
+ [Environment specifications](#q-supported-features)
  + [How much ephemeral storage is available to each environment?](#worker-storage)
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
  + [Why don't I see my plugins in the Airflow 2\.0 Admin > Plugins menu?](#view-plugins-ui)
  + [Can I use AWS Database Migration Service \(DMS\) Operators?](#ops-dms)
+ [Migrating](#q-migrating)
  + [How do I migrate to Amazon MWAA from an on\-premises or a self\-managed Apache Airflow deployment?](#migrate-from-onprem)

## Supported versions<a name="q-supported-versions"></a>

### Why are older versions of Apache Airflow not supported?<a name="airflow-version"></a>

We are only supporting the latest \(as of launch\) Apache Airflow version Apache Airflow v1\.10\.12 due to security concerns with older versions\.

### What Python version should I use?<a name="python-version"></a>

The following Apache Airflow versions are supported on Amazon Managed Workflows for Apache Airflow \(MWAA\)\.


| Airflow version | Airflow guide | Airflow constraints | Python version | 
| --- | --- | --- | --- | 
|  v2\.0\.2  |  [Apache Airflow v2\.0\.2 reference guide](http://airflow.apache.org/docs/apache-airflow/2.0.2/index.html)  |  [https://raw\.githubusercontent\.com/apache/airflow/constraints\-2\.0\.2/constraints\-3\.7\.txt](https://raw.githubusercontent.com/apache/airflow/constraints-2.0.2/constraints-3.7.txt)  |  [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)  | 
|  v1\.10\.12  |  [Apache Airflow v1\.10\.12 reference guide](https://airflow.apache.org/docs/apache-airflow/1.10.12/)  |  [https://raw\.githubusercontent\.com/apache/airflow/constraints\-1\.10\.12/constraints\-3\.7\.txt](https://raw.githubusercontent.com/apache/airflow/constraints-1.10.12/constraints-3.7.txt)  |  [Python 3\.7](https://www.python.org/dev/peps/pep-0537/)  | 

### Can I specify more than 25 Apache Airflow Workers?<a name="scaling-quota"></a>

Yes\. Although you can specify up to 25 Apache Airflow *Workers* on the Amazon MWAA console, you can configure up to 50 on an environment by requesting a quota increase\. We recommend using Apache Airflow v2\.0\.2 for an environment with more than 25 *Workers*\. To learn more, see [Requesting a quota increase](https://docs.aws.amazon.com/servicequotas/latest/userguide/request-quota-increase.html)\.

## Use cases<a name="t-common-questions"></a>

### When should I use AWS Step Functions vs\. Amazon MWAA?<a name="t-step-functions"></a>

1. You can use Step Functions to process individual customer orders, since Step Functions can scale to meet demand for one order or one million orders\.

1. If you’re running an overnight workflow that processes the previous day’s orders, you can use Step Functions or Amazon MWAA\. Amazon MWAA allows you an open source option to abstract the workflow from the AWS resources you're using\.

## Environment specifications<a name="q-supported-features"></a>

### How much ephemeral storage is available to each environment?<a name="worker-storage"></a>

The ephemeral storage \(RAM\) is determined by the environment class you specify\. To learn more, see [Amazon MWAA environment class](environment-class.md)\. 

### What is the default operating system used for Amazon MWAA environments?<a name="default-os"></a>

Amazon MWAA environments are created on instances running Amazon Linux AMI\.

### Can I use a custom image for my Amazon MWAA environment?<a name="custom-image"></a>

Custom images are not supported\. Amazon MWAA uses images that are built on Amazon Linux AMI\. Amazon MWAA installs the additional requirements by running `pip3 -r install` for the requirements specified in the requirements\.txt file you add to the Amazon S3 bucket for the environment\.

### Is MWAA HIPAA compliant?<a name="hipaa-compliance"></a>

Amazon MWAA is not currently HIPAA compliant\.

### Does Amazon MWAA support Spot Instances?<a name="spot-instances"></a>

Amazon MWAA does not currently support on\-demand Amazon EC2 Spot Instance types for Apache Airflow\. However, an Amazon MWAA environment can trigger Spot Instances on, for example, Amazon EMR and Amazon EC2\.

### Does Amazon MWAA support a custom domain?<a name="custom-dns"></a>

Yes\. You can use a custom domain for your Apache Airflow host name using [Amazon Route 53 ](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html)\. Apply an [AWS Certificate Manager \(ACM\) certificate](https://docs.aws.amazon.com/acm/latest/userguide/gs.html) to the Application Load Balancer, and then apply the Route 53 CNAME to the Application Load Balancer to match the fully qualified domain name \(FQDN\) of the certificate\.

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
Total ephemeral storage is limited to 4 GB\. There's no guarantee that subsequent tasks will be run on the same Fargate container instance, which might use a different `/tmp` folder\.

### Does Amazon MWAA support shared Amazon VPCs or shared subnets?<a name="shared-vpc"></a>

Amazon MWAA doesn't currently support shared Amazon VPCs or shared subnets\. The VPC you select when you create an environment should be owned by the account that is attempting to create environment\. To learn more, see [Work with shared VPCs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-sharing.html#vpc-share-unsupported-services)\.

## Metrics<a name="q-metrics"></a>

### What metrics are used to determine whether to scale Workers?<a name="metrics-workers"></a>

Amazon MWAA monitors the **QueuedTasks** and **RunningTasks** in CloudWatch to determine whether to scale Apache Airflow *Workers * on your environment\. To learn more, see [Monitoring and metrics for Amazon Managed Workflows for Apache Airflow \(MWAA\)](cw-metrics.md)\.

### Can I create custom metrics in CloudWatch?<a name="metrics-custom"></a>

Not on the CloudWatch console\. However, you can create a DAG that writes custom metrics in CloudWatch\. To learn more, see [Using a DAG to write custom metrics in CloudWatch](samples-custom-metrics.md)\. 

We'll also be releasing a serverless\-native executor similar to the Kubernetes executor via Open Source in September 2021\.

## DAGs, Operators, Connections, and other questions<a name="q-dags"></a>

### Can I use the `PythonVirtualenvOperator`?<a name="virtual-env-dags"></a>

The `PythonVirtualenvOperator` is not explicitly supported on Amazon MWAA, but you can create a custom plugin that uses the `PythonVirtualenvOperator`\. For sample code, see [Creating a custom plugin for Apache Airflow PythonVirtualenvOperator](samples-virtualenv.md)\.

### How long does it take Amazon MWAA to recognize a new DAG file?<a name="recog-dag"></a>

DAGs are synchronized from the S3 bucket the environment\. If you add a new DAG file, it takes about a minute for Amazon MWAA to start *using* the new file\. If you update an existing DAG file, it takes Amazon MWAA about 10 seconds to recognize the updates\.

### Why is my DAG file not picked up by Apache Airflow?<a name="dag-file-error"></a>

The following are possible solutions for this issue:

1. Check that your execution role has sufficient permissions to your Amazon S3 bucket\. To learn more, see [Amazon MWAA execution role](mwaa-create-role.md)\.

1. Check that the Amazon S3 bucket has *Block Public Access* configured, and *Versioning* enabled\. To learn more, see [Create an Amazon S3 bucket for Amazon MWAA](mwaa-s3-bucket.md)\.

1. Verify the DAG file itself\. For example, be sure that each DAG has a unique DAG ID\.

### Can I remove a `plugins.zip` or `requirements.txt` from an environment?<a name="remove-plugins-reqs"></a>

Currently, there is no way to remove a plugins\.zip or requirements\.txt from an environment once they’ve been added, but we're working on the issue\. In the interim, a workaround is to point to an empty text or zip file, respectively\. To learn more, see [Deleting files on Amazon S3](working-dags-delete.md)\.

### Why don't I see my plugins in the Airflow 2\.0 Admin > Plugins menu?<a name="view-plugins-ui"></a>

For security reasons, the Apache Airflow Web server on Amazon MWAA has limited network egress, and does not install plugins nor Python dependencies directly on the Apache Airflow *Web server*\. The plugin that's shown allows Amazon MWAA to authenticate your Apache Airflow users in AWS Identity and Access Management \(IAM\)\.

### Can I use AWS Database Migration Service \(DMS\) Operators?<a name="ops-dms"></a>

Amazon MWAA doesn't currently support [DMS Operators](https://airflow.apache.org/docs/apache-airflow-providers-amazon/stable/operators/dms.html)\. Each environment has its own Amazon Aurora PostgreSQL managed by AWS\. 

## Migrating<a name="q-migrating"></a>

### How do I migrate to Amazon MWAA from an on\-premises or a self\-managed Apache Airflow deployment?<a name="migrate-from-onprem"></a>

While each circumstance is different, in general, the recommendation is to move workloads gradually and run them in parallel if possible until you can verify that everything works as expected before you decommission your on\-premises or self\-managed deployment\.