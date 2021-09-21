# Accessing the Apache Airflow UI<a name="access-airflow-ui"></a>

An Apache Airflow UI link is available on the Amazon Managed Workflows for Apache Airflow \(MWAA\) console after you create an environment\. You can use the Amazon MWAA console to view and invoke a DAG in your Apache Airflow UI, or use Amazon MWAA APIs to get a token and invoke a DAG\. This section describes the permissions needed to access the Apache Airflow UI, how to generate a token to make Amazon MWAA API calls directly in your command shell, and the supported commands in the Apache Airflow CLI\.

**Topics**
+ [Prerequisites](#access-airflow-ui-prereqs)
+ [Open Airflow UI](#access-airflow-ui-onconsole)
+ [Logging into Apache Airflow](#airflow-access-and-login)
+ [Creating an Apache Airflow web login token](call-mwaa-apis-web.md)
+ [Creating an Apache Airflow CLI token](call-mwaa-apis-cli.md)
+ [Apache Airflow CLI command reference](airflow-cli-command-reference.md)

## Prerequisites<a name="access-airflow-ui-prereqs"></a>

The following section describes the preliminary steps required to use the commands and scripts in this section\.

### Access<a name="access-airflow-ui-prereqs-access"></a>
+ AWS account access in AWS Identity and Access Management \(IAM\) to the Amazon MWAA permissions policy in [Apache Airflow UI access policy: AmazonMWAAWebServerAccess](access-policies.md#web-ui-access)\.
+ AWS account access in AWS Identity and Access Management \(IAM\) to the Amazon MWAA permissions policy [Full API and console access policy: AmazonMWAAFullApiAccess](access-policies.md#full-access-policy)\.

### AWS CLI<a name="access-airflow-ui-prereqs-cli"></a>

The AWS Command Line Interface \(AWS CLI\) is an open source tool that enables you to interact with AWS services using commands in your command\-line shell\. To complete the steps on this page, you need the following:
+ [AWS CLI – Install version 2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)\.
+ [AWS CLI – Quick configuration with `aws configure`](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)\.

## Open Airflow UI<a name="access-airflow-ui-onconsole"></a>

The following image shows the link to your Apache Airflow UI on the Amazon MWAA console\.

![\[This image shows the link to your Apache Airflow UI on the Amazon MWAA console.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-aa-ui.png)

## Logging into Apache Airflow<a name="airflow-access-and-login"></a>

You need [Apache Airflow UI access policy: AmazonMWAAWebServerAccess](access-policies.md#web-ui-access) permissions for your AWS account in AWS Identity and Access Management \(IAM\) to view your Apache Airflow UI\. 

**To access your Apache Airflow UI**

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose **Open Airflow UI**\.

**To log\-in to your Apache Airflow UI**
+ Enter the AWS Identity and Access Management \(IAM\) user name and password for your account\.

![\[This image shows how to log-in to your Apache Airflow UI.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-aa-ui-login.png)