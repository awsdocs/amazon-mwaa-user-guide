# Managing access to an Amazon MWAA environment<a name="manage-access"></a>

Amazon Managed Workflows for Apache Airflow \(MWAA\) needs to be permitted to use other AWS services and resources used by an environment\. You also need to be granted permission to access an Amazon MWAA environment and your Apache Airflow UI in AWS Identity and Access Management \(IAM\)\. This section describes the execution role used to grant access to the AWS resources for your environment and how to add permissions, and the AWS account permissions you need to access your Amazon MWAA environment and Apache Airflow UI\. 

**Topics**
+ [Service\-linked role for Amazon MWAA](mwaa-slr.md)
+ [Amazon MWAA execution role](mwaa-create-role.md)
+ [Cross\-service confused deputy prevention](cross-service-confused-deputy-prevention.md)
+ [Apache Airflow access modes](configuring-networking.md)
+ [Accessing an Amazon MWAA environment](access-policies.md)