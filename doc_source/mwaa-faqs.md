# Amazon MWAA frequently asked questions<a name="mwaa-faqs"></a>

This page describes common questions you may encounter when using Amazon Managed Workflows for Apache Airflow \(MWAA\)\.

**Contents**
+ [Supported versions](#q-supported-versions)
  + [Why are older versions of Apache Airflow not supported?](#airflow-version)
  + [What Python version should I use?](#python-version)
+ [Environment specifications](#q-supported-features)
  + [How much ephemeral storage is available to each environment?](#worker-storage)
  + [What is the default operating system used for Amazon MWAA environments?](#default-os)
  + [Can I use a custom image for my Amazon MWAA environment?](#custom-image)
  + [Is MWAA HIPAA compliant?](#hipaa-compliance)
  + [Does Amazon MWAA support SPOT instances?](#spot-instances)
  + [Does Amazon MWAA support a custom domain?](#custom-dns)
  + [Can I remove a `plugins.zip` or `requirements.txt` from an environment?](#remove-plugins-reqs)
+ [DAGs](#q-dags)
  + [Can I use the `PythonVirtualenvOperator`?](#virtual-env-dags)
  + [How long does it take Amazon MWAA to recognize a new DAG file?](#recog-dag)
  + [Why is my DAG file not picked up by Apache Airflow?](#dag-file-error)
  + [Can I SSH into my environment?](#ssh-dag)
+ [Migrating](#q-migrating)
  + [How do I migrate to Amazon MWAA from an on\-premises or a self\-managed Apache Airflow deployment?](#migrate-from-onprem)

## Supported versions<a name="q-supported-versions"></a>

### Why are older versions of Apache Airflow not supported?<a name="airflow-version"></a>

We are only supporting the latest \(as of launch\) version 1\.10\.12 due to security concerns with older versions\.

### What Python version should I use?<a name="python-version"></a>

We recommend using Python 3\.7\.

## Environment specifications<a name="q-supported-features"></a>

### How much ephemeral storage is available to each environment?<a name="worker-storage"></a>

The ephemeral storage \(RAM\) is determined by the environment class you specify\. To learn more, see [Amazon MWAA environment class](environment-class.md)\. 

### What is the default operating system used for Amazon MWAA environments?<a name="default-os"></a>

Amazon MWAA environments are created on instances running Amazon Linux AMI\.

### Can I use a custom image for my Amazon MWAA environment?<a name="custom-image"></a>

Custom images are not supported\. Amazon MWAA uses images that are built on Amazon Linux AMI\. Amazon MWAA installs the additional requirements by running `pip3 -r install` for the requirements specified in the requirements\.txt file you add to the Amazon S3 bucket for the environment\.

### Is MWAA HIPAA compliant?<a name="hipaa-compliance"></a>

Amazon MWAA is not currently HIPAA compliant\.

### Does Amazon MWAA support SPOT instances?<a name="spot-instances"></a>

Amazon MWAA does not currently support SPOT instances for managed Apache Airflow\. However, an Amazon MWAA environment can trigger SPOT instances on, for example, Amazon EMR and Amazon EC2\.

### Does Amazon MWAA support a custom domain?<a name="custom-dns"></a>

Yes\. You can use a custom domain for your Apache Airflow host name using [Amazon Route 53](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html)\. Apply an [AWS Certificate Manager \(ACM\) certificate](https://docs.aws.amazon.com/acm/latest/userguide/gs.html) to the Application Load Balancer, and then apply the Route 53 CNAME to the Application Load Balancer to match the fully qualified domain name \(FQDN\) of the certificate\.

### Can I remove a `plugins.zip` or `requirements.txt` from an environment?<a name="remove-plugins-reqs"></a>

Currently, there is no way to remove a `plugins.zip` or `requirements.txt` from an environment once they’ve been added, but we're working on the issue\. In the interim, a workaround is to point to an empty text or zip file, respectively\.

## DAGs<a name="q-dags"></a>

### Can I use the `PythonVirtualenvOperator`?<a name="virtual-env-dags"></a>

The `PythonVirtualenvOperator` is not currently supported on Amazon MWAA, and though we're working to include it, we recommend using external containers to run custom workloads\. For sample code, see [Using Amazon MWAA with Amazon EKS](mwaa-eks-example.md)\.

### How long does it take Amazon MWAA to recognize a new DAG file?<a name="recog-dag"></a>

DAGs are synchronized from the S3 bucket the environment\. If you add a new DAG file, it takes about a minute for Amazon MWAA to start using the new file\. It takes Amazon MWAA about 10 seconds to recognize updates to an existing DAG file\.

### Why is my DAG file not picked up by Apache Airflow?<a name="dag-file-error"></a>

The following are possible solutions for this issue:
+ Ensure that the execution role you chose for the environment has sufficient permissions to the S3 bucket\. To learn more, see [Amazon MWAA Execution role](mwaa-create-role.md)\.
+ Ensure that the S3 bucket for the environment has *Block Public Access* configured\. To prevent malicious code injection, Amazon MWAA does not copy DAGs from a bucket that potentially allows public access\. To learn more, see [Create an Amazon S3 bucket for Amazon MWAA](mwaa-s3-bucket.md)\.
+ Verify the DAG file itself\. For example, be sure that each DAG has a unique DAG ID\.

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

## Migrating<a name="q-migrating"></a>

### How do I migrate to Amazon MWAA from an on\-premises or a self\-managed Apache Airflow deployment?<a name="migrate-from-onprem"></a>

While each circumstance is different, in general, the recommendation is to move workloads gradually and run them in parallel if possible until you can verify that everything works as expected before you decommission your on\-premises or self\-managed deployment\.