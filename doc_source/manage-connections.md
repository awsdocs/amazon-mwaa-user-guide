# Managing connections to Apache Airflow<a name="manage-connections"></a>

This section describes the different ways to configure an Apache Airflow connection for an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment\.

**Topics**
+ [Overview of Apache Airflow variables and connections](#manage-connections-t-overview)
+ [Apache Airflow v2 provider packages installed on Amazon MWAA environments](connections-packages.md)
+ [Overview of connection types](manage-connection-types.md)
+ [Configuring an Apache Airflow connection using a Secrets Manager secret](connections-secrets-manager.md)

## Overview of Apache Airflow variables and connections<a name="manage-connections-t-overview"></a>

In some cases, you may want to specify additional connections or variables for an environment, such as an AWS profile, or to add your execution role in a connection object in the Apache Airflow metastore, then refer to the connection from within a DAG\.
+ **Self\-managed Apache Airflow**\. On a self\-managed Apache Airflow installation, you set [Apache Airflow configuration options in `airflow.cfg`](https://airflow.apache.org/docs/apache-airflow/stable/howto/set-config.html)\.

  ```
  [secrets]
  backend = airflow.providers.amazon.aws.secrets.secrets_manager.SecretsManagerBackend
  backend_kwargs = {"connections_prefix" : "airflow/connections", "variables_prefix" : "airflow/variables"}
  ```
+ **Apache Airflow on Amazon MWAA**\. On Amazon MWAA, you need to add these configuration settings as [Apache Airflow configuration options](configuring-env-variables.md) on the Amazon MWAA console\. Apache Airflow configuration options are written as environment variables to your environment and override all other existing configurations for the same setting\.