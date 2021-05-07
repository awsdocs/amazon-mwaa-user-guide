# Amazon MWAA API actions and resources<a name="mwaa-actions-resources"></a>

The Amazon Managed Workflows for Apache Airflow \(MWAA\) API includes the following `airflow` actions\. For more information about the Amazon MWAA API, see the AWS SDK, or type `aws mwaa help` in the AWS CLI\.

 `CreateEnvironment`   
Creates an Amazon MWAA environment\.

 `ListEnvironments`   
Retrieves a list of the Amazon MWAA environments in your account\.

 `GetEnvironment`   
Retrieves details about an Amazon MWAA environment\.

 `UpdateEnvironment`   
Updates the specified environment\.

 `DeleteEnvironment`   
Deletes the specified\.

 `CreateWebLoginToken`   
Creates a URL to access the Apache Airflow Web UI in an Amazon MWAA environment\.

 `CreateCliToken`   
Creates a token used to authenticate an AWS CLI request for an Apache Airflow operation in an Amazon MWAA environment\.

 `PublishMetrics`   
Publishes metrics for an Amazon MWAA environment in your account\.

 `TagResource`   
Adds a tag to a resource\.

 `ListTagsForResource`   
Lists the tags added to a resource\.

 `UntagResource`   
Removes a tag from a resource\.

## Amazon MWAA resources<a name="mwaa-resources"></a>

Amazon MWAA supports the following resources\.

 `environment`   
An Amazon MWAA environment\.  
ARN pattern: `arn:${Partition}:airflow:${Region}:${Account}:environment/${EnvironmentName}`

 `rbac-role`   
An Apache Airflow roles used to manage access to Apache Airflow in your environment\.  
ARN pattern: `arn:${Partition}:airflow:${Region}:${Account}:role/${EnvironmentName}/${RoleName}`