# Apache Airflow v2 provider packages installed on Amazon MWAA environments<a name="connections-packages"></a>

Amazon MWAA installs [provider extras](http://airflow.apache.org/docs/apache-airflow/2.0.2/extra-packages-ref.html#providers-extras) for Apache Airflow v2 and above connection types when you create a new environment\. Installing provider packages allows you to view a connection type in the Apache Airflow UI\. It also means you don't need to specify these packages as a Python dependency in your `requirements.txt` file\. This page lists the Apache Airflow provider packages installed by Amazon MWAA for all Apache Airflow v2 environments\.

**Note**  
 For Apache Airflow v2 and above, Amazon MWAA installs [Watchtower version 2\.0\.1](https://pypi.org/project/watchtower/2.0.1/) after performing `pip3 install -r requirements.txt`, to ensure compatibility with CloudWatch logging is not overridden by other Python library installations\. 

**Contents**
+ [Provider packages for Apache Airflow v2\.2\.2 connections](#connections-packages-table-222)
+ [Provider packages for Apache Airflow v2\.0\.2 connections](#connections-packages-table-202)
+ [Specifying newer provider packages](#connections-packages-newer-packages)

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

## Specifying newer provider packages<a name="connections-packages-newer-packages"></a>

 Apache Airflow constraints files, which we recommend using whenever you create a `requirements.txt` for your environment, specify the provider packages versions available at the time of that Apache Airflow release\. In many cases, newer providers will be compatible with that version of Apache Airflow\. Rather than omitting constraints altogether, which we do not recommend, it is possible to simply modify the constraints file for a specific provider version\. 

 For example, to use Amazon provider package version 6\.0\.0 with Amazon MWAA and Apache Airflow v2\.2\.2, do the following: 

1.  Download the version\-specific constraints file from [https://raw\.githubusercontent\.com/apache/airflow/constraints\-2\.2\.2/constraints\-3\.7\.txt](https://raw.githubusercontent.com/apache/airflow/constraints-2.2.2/constraints-3.7.txt) 

1.  Modify the `apache-airflow-providers-amazon` version in the constraints file from `2.4.0` to `6.0.0`, and modify `watchtower` from `1.0.6` to `2.0.1`, the latter due to the dependency listed on [PyPI](https://pypi.org)\. 

1.  Save the modified constraints file to the Amazon S3 dags folder of your Amazon MWAA environment, for example, as `constraints-3.7-mod.txt` 

1.  Specify your requirements as shown in the following\. 

   ```
   --constraint "/usr/local/airflow/dags/constraints-3.7-mod.txt"
   
   apache-airflow-providers-amazon==6.0.0
   ```

**Note**  
 If you are using a private web server, we recommend you [package the required libraries as WHL files](best-practices-dependencies.md#best-practices-dependencies-python-wheels) by using the Amazon MWAA [local\-runner](https://github.com/aws/aws-mwaa-local-runner)\. 