# Apache Airflow v2 provider packages installed on Amazon MWAA environments<a name="connections-packages"></a>

Amazon MWAA installs some [provider extras](http://airflow.apache.org/docs/apache-airflow/2.0.2/extra-packages-ref.html#providers-extras) for Apache Airflow 2\+ connections when you create an environment\. The installation of these packages allows you to view a connection type in the Apache Airflow UI\. It also means you don't need to specify these packages as a Python dependency in the `requirements.txt` file for your environment\. This page lists the Apache Airflow provider packages used for connections that are installed by Amazon Managed Workflows for Apache Airflow \(MWAA\) to all Airflow 2\+ environments\.

**Note**  
 Amazon MWAA installs [Watchtower version 1\.0\.6](https://pypi.org/project/watchtower/) after performing `pip3 install -r requirements.txt`, to ensure compatibility with CloudWatch logging is not overridden by other Python library installations\. 

**Contents**
+ [Provider packages for Apache Airflow v2\.2\.2 connections](#connections-packages-table-222)
+ [Provider packages for Apache Airflow v2\.0\.2 connections](#connections-packages-table-202)

## Provider packages for Apache Airflow v2\.2\.2 connections<a name="connections-packages-table-222"></a>

When you create an Amazon MWAA environment in Apache Airflow v2\.2\.2, Amazon MWAA installs the following provider packages used for Apache Airflow connections\.


| Connection type | Package | 
| --- | --- | 
|  AWS Connection  |  [apache\-airflow\-providers\-amazon==2\.4\.0](https://airflow.apache.org/docs/apache-airflow-providers-amazon/stable/index.html)  | 
|  Postgres Connection  |  [apache\-airflow\-providers\-postgres==2\.3\.0](https://airflow.apache.org/docs/apache-airflow-providers-postgres/stable/index.html)  | 
|  FTP Connection  |  [apache\-airflow\-providers\-ftp==2\.0\.1](https://airflow.apache.org/docs/apache-airflow-providers-ftp/stable/index.html)  | 
|  Celery Connection  |  [apache\-airflow\-providers\-celery==2\.1\.0](https://airflow.apache.org/docs/apache-airflow-providers-celery/stable/index.html)  | 
|  HTTP Connection  |  [apache\-airflow\-providers\-http==2\.0\.1](https://airflow.apache.org/docs/apache-airflow-providers-http/stable/index.html)  | 
|  IMAP Connection  |  [apache\-airflow\-providers\-imap==2\.0\.1](https://airflow.apache.org/docs/apache-airflow-providers-imap/stable/index.html)  | 
|  SQLite Connection  |  [apache\-airflow\-providers\-sqlite==2\.0\.1](https://airflow.apache.org/docs/apache-airflow-providers-sqlite/stable/index.html)  | 

## Provider packages for Apache Airflow v2\.0\.2 connections<a name="connections-packages-table-202"></a>

When you create an Amazon MWAA environment in Apache Airflow v2\.0\.2, Amazon MWAA installs the following provider packages used for Apache Airflow connections\.


| Connection type | Package | 
| --- | --- | 
|  Tableau Connection  |  [apache\-airflow\-providers\-tableau==1\.0\.0](https://airflow.apache.org/docs/apache-airflow-providers-tableau/stable/index.html)  | 
|  Databricks Connection  |  [apache\-airflow\-providers\-databricks==1\.0\.1](https://airflow.apache.org/docs/apache-airflow-providers-databricks/stable/index.html)  | 
|  SSH Connection  |  [apache\-airflow\-providers\-ssh==1\.3\.0](https://airflow.apache.org/docs/apache-airflow-providers-ssh/stable/index.html)  | 
|  Postgres Connection  |  [apache\-airflow\-providers\-postgres==1\.0\.2](https://airflow.apache.org/docs/apache-airflow-providers-postgres/stable/index.html)  | 
|  Docker Connection  |  [apache\-airflow\-providers\-docker==1\.2\.0](https://airflow.apache.org/docs/apache-airflow-providers-docker/stable/index.html)  | 
|  Oracle Connection  |  [apache\-airflow\-providers\-oracle==1\.1\.0](https://airflow.apache.org/docs/apache-airflow-providers-oracle/stable/index.html)  | 
|  Presto Connection  |  [apache\-airflow\-providers\-presto==1\.0\.2](https://airflow.apache.org/docs/apache-airflow-providers-presto/stable/index.html)  | 
|  SFTP Connection  |  [apache\-airflow\-providers\-sftp==1\.2\.0](https://airflow.apache.org/docs/apache-airflow-providers-sftp/stable/index.html)  | 