# Working with DAGs on Amazon MWAA<a name="working-dags"></a>

To run Directed Acyclic Graphs \(DAGs\) on an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment, you copy your files to the Amazon S3 storage bucket attached to your environment, then let Amazon MWAA know where your DAGs and supporting files are located on the Amazon MWAA console\. Amazon MWAA takes care of synchronizing the DAGs among workers, schedulers, and the web server\. This guide describes how to add or update your DAGs, and install custom plugins and Python dependencies on an Amazon MWAA environment\.

**Topics**
+ [Amazon S3 bucket overview](#working-dags-s3-about)
+ [Adding or updating DAGs](configuring-dag-folder.md)
+ [Installing custom plugins](configuring-dag-import-plugins.md)
+ [Installing Python dependencies](working-dags-dependencies.md)
+ [Deleting files on Amazon S3](working-dags-delete.md)

## Amazon S3 bucket overview<a name="working-dags-s3-about"></a>

An Amazon S3 bucket for an Amazon MWAA environment must have *Public Access Blocked*\. By default, all Amazon S3 resources—buckets, objects, and related sub\-resources \(for example, lifecycle configuration\)—are private\. 
+ Only the resource owner, the AWS account that created the bucket, can access the resource\. The resource owner \(for example, your administrator\) can grant access permissions to others by writing an access control policy\. 
+ The access control policy you must have been granted access to add DAGs, custom plugins in `plugins.zip`, and Python dependencies in `requirements.txt` to your Amazon S3 bucket is [AmazonMWAAFullConsoleAccess](access-policies.md#console-full-access)\. 

An Amazon S3 bucket for an Amazon MWAA environment must have *Versioning Enabled*\. When Amazon S3 bucket versioning is enabled, anytime a new version is created, a new copy is created\.
+ Versioning is enabled for the custom plugins in a `plugins.zip`, and Python dependencies in a `requirements.txt` on your Amazon S3 bucket\.
+ You must specify the version of a `plugins.zip`, and `requirements.txt` on the Amazon MWAA console each time these files are updated on your Amazon S3 bucket\.