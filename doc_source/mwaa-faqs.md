# Amazon MWAA frequently asked questions<a name="mwaa-faqs"></a>

Review the following answers to frequently asked questions about Amazon MWAA\.

**Topics**
+ [Why are older versions of Apache Airflow not supported?](#airflow-version)
+ [What is the default operating system used for Amazon MWAA environments?](#default-os)
+ [Can I use a custom image for my Amazon MWAA environment?](#custom-image)
+ [Is MWAA HIPAA compliant?](#hipaa-compliance)
+ [How do I trigger or backfill DAGs?](#backfill-dags)
+ [How do I migrate to Amazon MWAA from an on\-premises or a self\-managed Apache Airflow deployment?](#migrate-from-onprem)
+ [How long does it take Amazon MWAA to recognize a new DAG file?](#recog-dag)
+ [Why is my DAG file not picked up by Apache Airflow?](#dag-file-error)
+ [Does Amazon MWAA support SPOT instances?](#spot-instances)
+ [How do I specify a newer version of the AWS boto3 Python library?](#new-boto-version)
+ [How much local storage is available to each worker?](#worker-storage)
+ [How do I enable the Log UI link?](#enable-log-ui)

## Why are older versions of Apache Airflow not supported?<a name="airflow-version"></a>

We are only supporting the latest \(as of launch\) version 1\.10\.12 due to security concerns with older versions\.

## What is the default operating system used for Amazon MWAA environments?<a name="default-os"></a>

Amazon MWAA environments are created on instances running Amazon Linux AMI\.

## Can I use a custom image for my Amazon MWAA environment?<a name="custom-image"></a>

Custom images are not supported\. Amazon MWAA uses images that are built on Amazon Linux AMI\. Amazon MWAA installs the additional requirements by running `pip3 -r install` for the requirements specified in the requirements\.txt file you add to the Amazon S3 bucket for the environment\.

## Is MWAA HIPAA compliant?<a name="hipaa-compliance"></a>

Amazon MWAA is not currently HIPAA compliant\.

## How do I trigger or backfill DAGs?<a name="backfill-dags"></a>

To trigger or backfill a DAG, you can use the Apache Airflow CLI\. You can access the Apache Airflow CLI via the `CreateCliToken` of the Amazon MWAA API\. To learn more, see [Accessing the Apache Airflow UI](access-airflow-ui.md)\.

When you create a request, include the Apache Airflow CLI command in the `--data-raw` header\. See the [Apache Airflow Command Line Interface Reference](https://airflow.apache.org/docs/stable/cli-ref) for more information on Apache Airflow CLI commands\.

## How do I migrate to Amazon MWAA from an on\-premises or a self\-managed Apache Airflow deployment?<a name="migrate-from-onprem"></a>

While each circumstance is different, in general, the recommendation is to move workloads gradually and run them in parallel if possible until you can verify that everything works as expected before you decommission your on\-premises or self\-managed deployment\.

## How long does it take Amazon MWAA to recognize a new DAG file?<a name="recog-dag"></a>

DAGs are synchronized from the S3 bucket the environment\. If you add a new DAG file, it takes about a minute for Amazon MWAA to start using the new file\. It takes Amazon MWAA about 10 seconds to recognize updates to an existing DAG file\.

## Why is my DAG file not picked up by Apache Airflow?<a name="dag-file-error"></a>

The following are possible solutions for this issue:
+ Ensure that the execution role you chose for the environment has sufficient permissions to the S3 bucket\. To learn more, see [Amazon MWAA Execution role](mwaa-create-role.md)\.
+ Ensure that the S3 bucket for the environment has *Block Public Access* configured\. To prevent malicious code injection, Amazon MWAA does not copy DAGs from a bucket that potentially allows public access\. To learn more, see [Create an Amazon S3 bucket for Amazon MWAA](mwaa-s3-bucket.md)\.
+ Verify the DAG file itself\. For example, be sure that each DAG has a unique DAG ID\.

## Does Amazon MWAA support SPOT instances?<a name="spot-instances"></a>

Amazon MWAA does not currently support SPOT instances for managed Apache Airflow\. However, an Amazon MWAA environment can trigger SPOT instances on, for example, Amazon EMR and Amazon EC2\.

## How do I specify a newer version of the AWS boto3 Python library?<a name="new-boto-version"></a>

To use a different version, specify the minimum version in your `requirements.txt`\. For example, to use version 1\.16\.32, add the following line:

```
boto3 >= 1.16.32
```

## How much local storage is available to each worker?<a name="worker-storage"></a>

The container for workers is determined by the environment class you specify\. To learn more, see [Amazon MWAA environment class](environment-class.md)\. 

## How do I enable the Log UI link?<a name="enable-log-ui"></a>

Use the following steps to enable the Log UI link\.

1. From the **Environments** page of the Amazon MWAA console, choose the name of the environment in the **Name** column to open the **Details** page for the environment to enable the link for\.

1. Copy the **Airflow UI** link in the **Details** section of the page\.

1. Choose **Edit** to update the environment settings, then on the **Specify details** page choose **Next** to open the next page of settings\.

1. Under **Airflow configuration options**, choose **Add custom configuration value**\.

1. For **Configuration option**, enter `webserver.base_url`\. For **Custom value**, type `https://` and then paste in the URL that you copied to access the Apache Airflow UI for the environment\.