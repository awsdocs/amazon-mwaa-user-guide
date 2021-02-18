# Working with DAGs on Amazon MWAA<a name="working-dags"></a>

To run Directed Acyclic Graphs \(DAGs\) on an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment, you copy your files to the Amazon S3 storage bucket attached to your environment, then let Amazon MWAA know where your DAGs and supporting files are located on the Amazon MWAA console\. Amazon MWAA takes care of synchronizing the DAGs among workers, schedulers, and the web server\. This guide describes how to add or update your DAGs, install custom plugins, and Python dependencies \(referred to as "extra packages" by Apache Airflow\) on an Amazon MWAA environment\.

**Topics**
+ [Adding or updating DAGs](configuring-dag-folder.md)
+ [Installing custom plugins](configuring-dag-import-plugins.md)
+ [Installing Python dependencies](working-dags-dependencies.md)