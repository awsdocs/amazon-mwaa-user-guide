# Create an Amazon MWAA environment<a name="create-environment"></a>

When you create an Amazon MWAA environment, it uses the [VPC network that you created for Amazon MWAA](vpc-create.md), and adds the other necessary networking components\. Amazon MWAA automatically installs the version of Apache Airflow that you specify, including workers, scheduler, and web server\. The environment includes a link to access the Apache Airflow UI in the environment\. You can create up to 10 environments per account per Region, and each environment can include multiple DAGs\. Before you create an environment, ensure that your environment meets the [prerequisites](get-started.md#prerequisites) for Amazon MWAA\.

**To create an environment**

1. Log in to AWS using an account that has [permissions to use Amazon MWAA](manage-access.md), and then open the [Amazon MWAA](https://console.aws.amazon.com/mwaa/home/) console\.

   Confirm that the Region selection is correct\.

1. Choose **Create environment**\.

1. On the **Specify details** page, under **Environment details**, provide a name for your environment and select the Apache Airflow version to use\.

   The environment name must be unique to the account and Region\.

1. Under **DAG code in Amazon S3**:

   For **S3 bucket**, do one of the following:
   + Choose **Browse S3**, choose the bucket that you created for Amazon MWAA, and then choose **Choose**\.
   + Enter the Amazon S3 URI to the bucket\.

1. For **DAGs folder**, do one of the following:
   + Choose **Browse S3**, choose the DAG folder that you added to the bucket for Amazon MWAA, and then choose **Choose**\.
   + Enter the Amazon S3 URI to the DAG folder in the bucket\.

1. \(Optional\)\. For **Plugins file**, do one of the following:
   + Choose **Browse S3** and select the plugins\.zip file that you added to the bucket\. You must also select a version from the drop\-down menu\.
   + Enter the Amazon S3 URI to the plugin\.zip file that you added to the bucket\.

   You can create an environment and then add a plugins\.zip file later\.

1. \(Optional\) For **Requirements file**, do one of the following:
   + Choose **Browse S3** and then select the Python requirements\.txt that you added to the bucket\. Then select a version for the file from the drop\-down menu\.
   + Enter the Amazon S3 URI to the requirements\.txt file in the bucket\.

   You can add a requirements file to your bucket after you create an environment\. After you add or update the file you can edit the environment to modify these settings\.

1. Choose **Next**\. 

1. On the **Configure advanced settings** page, under **Networking**, choose the VPC that was you [created for Amazon MWAA](vpc-create.md)\.

1. Under **Subnets**, ensure that only private subnets are selected\. Only private subnets are supported\. You can't change the VPC for an environment after you create it\.

1. Under **Web server access**, select **Public Network**\. This creates a public URL to access the Apache Airflow user interface in the environment\.
**Note**  
To restrict access to the Apache Airflow UI to be accessible only from within the VPC selected, choose **Private Network**\. This creates a VPC endpoint that requires additional configuration to allow access, including a Linux Bastion\. The VPC endpoint for to access the Apache Airflow UI is listed on the Environment details page after you create the environment\.

1. Under **Security group\(s\)**, do one of the following:
   + Choose **Create new security group** to have Amazon MWAA create a new security group with inbound and outbound rules based on your **Web server access** selection\.
   + Deselect the check box to create a new security group, and then select up to 5 security groups from your account to use for the environment\.

1. Under **Environment class**, select the environment size to use for your environment\. Choose the smallest size necessary to support your workload\. You can increase the environment size later as appropriate\. The environment size determines the approximate number of workflows that an environment supports\. 

1. For **Maximum worker count**, specify the maximum number of workers, up to 25, to run concurrently in the environment\. Amazon MWAA automatically handles working scaling up to the maximum worker count\. To learn more about scaling, see [Amazon MWAA automatic scaling](mwaa-autoscaling.md)\.

1. Under **Encryption**, Amazon MWAA by default uses an AWS owned key to encrypt your data\. To use a different key, choose **Customize encryption settings \(advanced\)**, then choose the key to use, or enter the ARN for the key to use\.

   You must have permissions to the key to be able to select it for your environment\. To learn more about using AWS KMS keys for your environment, see [Customer managed CMKs](custom-keys-certs.md)\.

   If you choose to use your own key, and you enabled server\-side encryption for the S3 bucket you created for Amazon MWAA, you must use the same key for both the S3 bucket and your Amazon MWAA environment\.

   You must also grant permissions for Amazon MWAA to use the key by attaching the policy described in [Attach key policy](custom-keys-certs.md#custom-keys-certs-grant-policies-attach)\.

1. Under **Monitoring**, choose whether to enable **CloudWatch Metrics**\. To learn more about metrics sent to CloudWatch, see [Amazon MWAA metrics](cw-metrics.md)\.

1. For **Airflow logging configuration**, choose whether to enable sending log data to CloudWatch Logs for the following Apache Airflow log categories:
   + **Airflow task logs**
   + **Airflow web server logs**
   + **Airflow scheduler logs**
   + **Airflow worker logs**
   + **Airflow DAG processing logs**

   After you enable a log category, choose the **Log level** for each as appropriate for your environment\.

1. For **Airflow configuration options**, to add a customer configuration option, choose **Add custom configuration option**\. Select the configuration option to use a custom value for, then enter the **Custom value**\.

   When you create an environment Apache Airflow is installed using the default configuration options\. If you add a custom configuration option, Apache Airflow uses the value from the custom configuration instead of the default\. To learn more, see [Amazon MWAA Apache Airflow configuration options](configuring-env-variables.md)\.

1. Under **Tags**, add any tags as appropriate for your environment\. Choose **Add new tag**, and then enter a **Key** and optionally, a **Value** for the key\.

1. Under **Permissions**, choose the role to use as the execution role\. To have Amazon MWAA create a role for this environment, choose **Create new role**\. You must have permission to create IAM roles to use this option\.

   If you or someone in your organization created a role to use for Amazon MWAA, choose that role\. To learn more, see [Amazon MWAA Execution role](mwaa-create-role.md)\.

1. Choose **Create environment**\.

   It takes about twenty to thirty minutes to create an environment\.

## What's next?<a name="mwaa-env-next-up"></a>
+ Learn how to grant users access to your Apache Airflow UI in [Managing access to an Amazon MWAA environment](manage-access.md)\.