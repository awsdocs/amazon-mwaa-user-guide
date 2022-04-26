# Security best practices on Amazon MWAA<a name="security-best-practices"></a>

Amazon MWAA provides a number of security features to consider as you develop and implement your own security policies\. The following best practices are general guidelines and don’t represent a complete security solution\. Because these best practices might not be appropriate or sufficient for your environment, treat them as helpful considerations rather than prescriptions\.
+ Use least\-permissive permission policies\. Grant permissions to only the resources or actions that users need to perform tasks\.
+ Use AWS CloudTrail to monitor user activity in your account\.
+ Ensure that the Amazon S3 bucket policy and object ACLs grant permissions to the users from the associated Amazon MWAA environment to put objects into the bucket\. This ensures that users with permissions to add workflows to the bucket also have permissions to run the workflows in Airflow\.
+ Use the Amazon S3 buckets associated with Amazon MWAA environments\. Your Amazon S3 bucket can be any name\. Do not store other objects in the bucket, or use the bucket with another service\. 

## Security best practices in Apache Airflow<a name="security-best-practices-for-airflow"></a>

 Apache Airflow is not multi\-tenant\. While there are [access control measures](https://airflow.apache.org/docs/apache-airflow/2.0.2/security/access-control.html) to limit some features to specific users, which [Amazon MWAA implements](access-policies.md#web-ui-access), DAG creators do have the ability to write DAGs that can change Apache Airflow user privileges and interact with the underlying metadatabase\. 

 We recommend the following steps when working with Apache Airflow on Amazon MWAA to ensure your environment's metadatabase and DAGs are secure\. 
+ Use separate environments for separate teams with DAG writing access, or the ability to add files to your Amazon S3 `/dags` folder, assuming anything accessible by the [Amazon MWAA Execution Role](mwaa-create-role.md) or [Apache Airflow connections](https://airflow.apache.org/docs/apache-airflow/2.0.2/howto/connection.html) will also be accessible to users who can write to the environment\.
+ Do not provide direct Amazon S3 DAGs folder access\. Instead, use CI/CD tools to write DAGs to Amazon S3, with a validation step ensuring that the DAG code meets your team's security guidelines\.
+ Prevent user access to your environment's Amazon S3 bucket\. Instead, use a DAG factory that generates DAGs based on a YAML, JSON, or other definition file stored in a separate location from your Amazon MWAA Amazon S3 bucket where you store DAGs\. 
+ Store secrets in [Secrets Manager](connections-secrets-manager.md)\. While this will not prevent users who can write DAGs from reading secrets, it will prevent them from modifying the secrets that your environment uses\.

### Detecting changes to Apache Airflow user privileges<a name="detecting-user-privilege-changes"></a>

 You can use CloudWatch Logs Insights to detect occurences of DAGs changing Apache Airflow user privileges\. To do so, you can use an EventBridge scheduled rule, a Lambda function, and CloudWatch Logs Insights to deliver notifications to CloudWatch metrics whenever one of your DAGs changes Apache Airflow user privileges\. 

#### Prerequisites<a name="prerequisites"></a>

 To complete the following steps, you will need the following: 
+  An Amazon MWAA environment with all Apache Airflow log types enabled at the `INFO` log level\. For more information, see [Viewing Airflow logs in Amazon CloudWatch](monitoring-airflow.md)\. 

**To configure notifications for changes to Apache Airflow user privileges**

1.  [Create a Lambda function](https://docs.aws.amazon.com/lambda/latest/dg/getting-started-create-function.html) that runs the following CloudWatch Logs Insights query string against the five Amazon MWAA environment log groups \(`DAGProcessing`, `Scheduler`, `Task`, `WebServer`, and `Worker`\)\. 

   ```
   fields @log, @timestamp, @message | filter @message like "add-role" | stats count() by @log
   ```

1.  [Create an EventBridge rule that runs on a schedule](https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-create-rule-schedule.html), with the Lambda function you created in the previous step as the rule's target\. Configure your schedule using a cron or rate expression to run at regular intervals\. 