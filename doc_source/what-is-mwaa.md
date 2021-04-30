# What Is Amazon Managed Workflows for Apache Airflow \(MWAA\)?<a name="what-is-mwaa"></a>

Amazon Managed Workflows for Apache Airflow \(MWAA\) \(Amazon MWAA\) is a managed service for Apache Airflow that makes it easy for you to build and manage your workflows in the cloud\.
+ With Amazon MWAA, you can easily combine data using any of Apache Airflow's open source integrations\. You can use the same familiar Apache Airflow platform as you do today to manage their workflows and now enjoy improved scalability, availability, and security without the operational burden of having to manage the underlying infrastructure\. Amazon MWAA automatically scales capacity up to meet demand and back down to conserve resources and minimize costs\. Amazon MWAA is integrated with AWS security services to enable secure access to customer data, and supports single sign\-on using the same AWS credentials to access the Apache Airflow UI\.
+ Amazon MWAA also manages the provisioning and ongoing maintenance of Apache Airflow for you\. Amazon MWAA automatically applies patches and updates to Apache Airflow in your Amazon MWAA environments\.
+ The active and growing open source community provides operators \(plugins that simplify connections to services\) for Apache Airflow to integrate with AWS services like Amazon S3, Amazon Redshift, Amazon EMR, AWS Batch, and Amazon SageMaker, as well as services on other cloud platforms\. Using Apache Airflow with Amazon MWAA fully supports integration with AWS services and popular third\-party tools such as Apache Hadoop, Presto, Hive, and Spark to perform data processing tasks\. Amazon MWAA is committed to maintaining compatibility with the Amazon MWAA API, and Amazon MWAA intends to provide reliable integrations to AWS services and make them available to the community, and be involved in community feature development\.
+ Amazon MWAA automatically, if enabled, sends Apache Airflow system metrics and logs to CloudWatch\. You can view logs and metrics for multiple environments from a single location to easily identify task delays or workflow errors without the need for additional third\-party tools\.

## Amazon MWAA regions<a name="regions-mwaa"></a>

Amazon MWAA is available in the following AWS Regions\.
+ Europe \(Stockholm\) \- eu\-north\-1
+ Europe \(Ireland\) \- eu\-west\-1
+ Asia Pacific \(Singapore\) \- ap\-southeast\-1
+ Asia Pacific \(Sydney\) \- ap\-southeast\-2
+ Europe \(Frankfurt\) \- eu\-central\-1
+ Asia Pacific \(Tokyo\) \- ap\-northeast\-1
+ US East \(N\. Virginia\) \- us\-east\-1
+ US East \(Ohio\) \- us\-east\-1
+ US West \(Oregon\) \- us\-west\-2

## Apache Airflow versions<a name="regions-aa-versions"></a>

The following Apache Airflow versions are available on Amazon MWAA\.
+ [Apache Airflow v1\.10\.12](https://airflow.apache.org/docs/apache-airflow/1.10.12/) is available in Python 3\.7

## Amazon MWAA architecture<a name="architecture-mwaa"></a>

The following image shows the architecture of a Amazon MWAA environment\.

![\[This image shows the architecture of a Amazon MWAA environment.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-architecture.png)

## Amazon MWAA benefits<a name="benefits-mwaa"></a>

Amazon MWAA offers the following benefits:
+ **Setup** – Amazon MWAA sets up Apache Airflow for you when you create an environment using the same open\-source Airflow and user interface available from Apache\. You don't need to perform a manual setup or use custom tools to create an environment\. Amazon MWAA is not a "branch" of Airflow, nor is it just "compatible with"\. It is the exact same Apache Airflow that you can download on your own\.
+ **Scaling** – Amazon MWAA uses the Apache [Celery Executor](https://airflow.apache.org/docs/stable/executor/celery.html) to automatically scale workers as needed for your environment\. Amazon MWAA monitors the workers in your environment, and as demand increases, Amazon MWAA adds additional worker containers\. As workers free up, Amazon MWAA removes them\.
+ **Security** – Integrated support with AWS Identity and Access Management \(IAM\), including role\-based authentication and authorization for access to the Airflow user interface\. Workers assume IAM roles for easy and secure access to AWS services\. Workers and Scheduler run in your VPC for access to your resources\.
+ **Public or private network** – Amazon MWAA supports accessing the Apache Airflow UI on either a VPC or a public secured endpoint\.
+ **Upgrades and patches** – Amazon MWAA will provide new versions of Apache Airflow periodically\. The images for these versions will be updated and patched by the Amazon MWAA team\.
+ **Monitoring** – Amazon MWAA is integrated with CloudWatch\. The Apache Airflow logs and performance metrics data for your environment are available in a single location\. This lets you easily identify workflow errors or task delays\.
+ **Integration** – Amazon MWAA supports open\-source integrations with: Amazon Athena, AWS Batch, Amazon CloudWatch, Amazon DynamoDB, AWS DataSync, Amazon EMR, AWS Fargate, Amazon EKS, Amazon Kinesis Data Firehose, AWS Glue, AWS Lambda, Amazon Redshift, Amazon SQS, Amazon SNS, Amazon SageMaker, and Amazon S3, as well as hundreds of built\-in and community\-created operators and sensors\.
+ **Containers** – Amazon MWAA offers support for using containers to scale the worker fleet on demand and reduce scheduler outages, through AWS Fargate\. To learn more about Fargate, see [Amazon ECS on AWS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html)\. Operators that execute tasks on Amazon ECS containers, as well as Kubernetes operators that create and run pods on a Kubernetes cluster, are supported\.

## Apache Airflow workflows<a name="workflows"></a>

Apache Airflow manages data through a series of tasks called a workflow\. When a workflow is created, tasks are configured so that some tasks must finish before the next task can start without needing to loop back to a previous task\. For example, tasks that collect and process data must finish collecting and processing all data before attempting to merge the data\. A workflow comprised of these tasks is referred to as a Directed Acyclic Graph \(DAG\)\.

An example of a DAG workflow is a collection of tasks for a media distribution company\. There is a task for connecting to each content provider service that media is distributed to, requesting the play count and sales for each title, pulling social media impressions, and then loading that data to a storage location, such as an Amazon S3 bucket\. After the data is uploaded, a task to process the data starts and converts the data to another format or modifies specific values\. The task to merge the data together starts only after all of the preceding tasks are completed\. This could be using tools like AWS Glue or Amazon Athena, or perhaps using Amazon SageMaker to identify similar entries that can be combined further\. After all tasks are complete, the result is a clean and complete data set ready for analysis, such as with Amazon Redshift, or storage with Amazon DynamoDB\.

If a task fails, such as a task that retrieves data, the workflow is configured to automatically retry the failed task while the subsequent tasks wait for that task to complete\. If a manual restart is required, the workflows starts at the failed task rather than the first task in the workflow\. This can save time and resources by not repeating tasks that had already completed successfully\.

## Apache Airflow components<a name="components-airflow"></a>

Each environment has an Airflow Scheduler and 1 or more Airflow Workers, managed by auto\-scaling, that are linked to your VPC\. The meta database and web servers are isolated in the service’s account, and there are separate instances of each for each Airflow environment created—there is no shared tenancy of any components, even within the same account\. Web server access can then be exposed through an endpoint within your VPC, or more simply can be exposed through a load balancer to a publicly accessible endpoint, in each case secured by IAM and AWS SSO\.