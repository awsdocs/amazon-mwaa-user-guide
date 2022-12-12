# Amazon MWAA execution role<a name="mwaa-create-role"></a>

An execution role is an AWS Identity and Access Management \(IAM\) role with a permissions policy that grants Amazon Managed Workflows for Apache Airflow \(MWAA\) permission to invoke the resources of other AWS services on your behalf\. This can include resources such as your Amazon S3 bucket, [AWS owned key](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk), and CloudWatch Logs\. Amazon MWAA environments need one execution role per environment\. This page describes how to use and configure the execution role for your environment to allow Amazon MWAA to access other AWS resources used by your environment\.

**Contents**
+ [Execution role overview](#mwaa-create-role-how)
  + [Permissions attached by default](#mwaa-create-role-how-create-role)
  + [How to add permission to use other AWS services](#mwaa-create-role-how-adding)
  + [How to associate a new execution role](#mwaa-create-role-how-associating)
+ [Create a new role](#mwaa-create-role-mwaa-onconsole)
+ [View and update an execution role policy](#mwaa-create-role-update)
  + [Attach a JSON policy to use other AWS services](#mwaa-create-role-attach-json-policy)
+ [Grant access to Amazon S3 bucket with account\-level public access block](#mwaa-create-role-s3-publicaccessblock)
+ [Use Apache Airflow connections](#mwaa-create-role-airflow-connections)
+ [Sample JSON policies for an execution role](#mwaa-create-role-json)
  + [Sample policy for a customer managed key](#mwaa-create-role-cmk)
  + [Sample policy for an AWS owned key](#mwaa-create-role-aocmk)
+ [What's next?](#mwaa-create-role-next-up)

## Execution role overview<a name="mwaa-create-role-how"></a>

Permission for Amazon MWAA to use other AWS services used by your environment are obtained from the execution role\. An Amazon MWAA execution role needs permission to the following AWS services used by an environment:
+ Amazon CloudWatch \(CloudWatch\) – to send Apache Airflow metrics and logs\.
+ Amazon Simple Storage Service \(Amazon S3\) – to parse your environment's DAG code and supporting files \(such as a `requirements.txt`\)\.
+ Amazon Simple Queue Service \(Amazon SQS\) – to queue your environment's Apache Airflow tasks in an Amazon SQS queue owned by Amazon MWAA\.
+ AWS Key Management Service \(AWS KMS\) – for your environment's data encryption \(using either an [AWS owned key](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk) or your [Customer managed key](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk)\)\.
**Note**  
 If you have elected for Amazon MWAA to use an AWS managed KMS key to encrypt your data, then you must define permissions in a policy attached to your Amazon MWAA execution role that grant access to arbitrary KMS keys stored outside of your account via Amazon SQS\. The following two conditions are required in order for your environment's execution role to access arbitrary KMS keys:   
A KMS key in a third\-party account needs to allow this cross account access via its resource policy\.
Your DAG code needs to access an Amazon SQS queue that starts with `airflow-celery-` in the third\-party account and uses the same KMS key for encryption\.
 In order to mitigate the risks associated with cross\-account access to resources, we recommend reviewing the code placed in your DAGs to ensure that your workflows are not accessing arbitrary Amazon SQS queues outside your account\. Furthermore, you can use a customer managed KMS key stored in your own account to manage encryption on Amazon MWAA\. This limits your environment's execution role to access only the KMS key in your account\.   
 Keep in mind that after you choose an encryption option, you cannot change your selection for an existing environment\. 

An execution role also needs permission to the following IAM actions:
+ `airflow:PublishMetrics` – to allow Amazon MWAA to monitor the health of an environment\.

### Permissions attached by default<a name="mwaa-create-role-how-create-role"></a>

You can use the default options on the Amazon MWAA console to create an execution role and an [AWS owned key](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk), then use the steps on this page to add permission policies to your execution role\.
+ When you choose the **Create new role** option on the console, Amazon MWAA attaches the minimal permissions needed by an environment to your execution role\.
+ In some cases, Amazon MWAA attaches the maximum permissions\. For example, we recommend choosing the option on the Amazon MWAA console to create an execution role when you create an environment\. Amazon MWAA adds the permissions policies for all CloudWatch Logs groups automatically by using the regex pattern in the execution role as `"arn:aws:logs:your-region:your-account-id:log-group:airflow-your-environment-name-*"`\.

### How to add permission to use other AWS services<a name="mwaa-create-role-how-adding"></a>

Amazon MWAA can't add or edit permission policies to an existing execution role after an environment is created\. You must update your execution role with additional permission policies needed by your environment\. For example, if your DAG requires access to AWS Glue, Amazon MWAA can't automatically detect these permissions are required by your environment, or add the permissions to your execution role\.

You can add permissions to an execution role in two ways:
+ By modifying the JSON policy for your execution role inline\. You can use the sample [JSON policy documents](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html) on this page to either add to or replace the JSON policy of your execution role on the IAM console\.
+ By creating a JSON policy for an AWS service and attaching it to your execution role\. You can use the steps on this page to associate a new JSON policy document for an AWS service to your execution role on the IAM console\.

Assuming the execution role is already associated to your environment, Amazon MWAA can start using the added permission policies immediately\. This also means if you remove any required permissions from an execution role, your DAGs may fail\.

### How to associate a new execution role<a name="mwaa-create-role-how-associating"></a>

You can change the execution role for your environment at any time\. If a new execution role is not already associated with your environment, use the steps on this page to create a new execution role policy, and associate the role to your environment\. 

## Create a new role<a name="mwaa-create-role-mwaa-onconsole"></a>

By default, Amazon MWAA creates an [AWS owned key](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk) for data encryption and an execution role on your behalf\. You can choose the default options on the Amazon MWAA console when you create an environment\. The following image shows the default option to create an execution role for an environment\.

![\[This is an image with the default option to create a new role.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-permissions.png)

## View and update an execution role policy<a name="mwaa-create-role-update"></a>

You can view the execution role for your environment on the Amazon MWAA console, and update the JSON policy for the role on the IAM console\.

**To update an execution role policy**

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose the execution role on the **Permissions** pane to open the permissions page in IAM\.

1. Choose the execution role name to open the permissions policy\.

1. Choose **Edit policy**\.

1. Choose the **JSON** tab\.

1. Update your JSON policy\.

1. Choose **Review policy**\.

1. Choose **Save changes**\.

### Attach a JSON policy to use other AWS services<a name="mwaa-create-role-attach-json-policy"></a>

You can create a JSON policy for an AWS service and attach it to your execution role\. For example, you can attach the following JSON policy to grant read\-only access to all resources in AWS Secrets Manager\.

```
{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Effect":"Allow",
         "Action":[
            "secretsmanager:GetResourcePolicy",
            "secretsmanager:GetSecretValue",
            "secretsmanager:DescribeSecret",
            "secretsmanager:ListSecretVersionIds"
         ],
         "Resource":[
            "*"
         ]
      }
   ]
}
```

**To attach a policy to your execution role**

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Choose your execution role on the **Permissions** pane\.

1. Choose **Attach policies**\.

1. Choose **Create policy**\.

1. Choose **JSON**\.

1. Paste the JSON policy\.

1. Choose **Next: Tags**, **Next: Review**\.

1. Enter a descriptive name \(such as `SecretsManagerReadPolicy`\) and a description for the policy\.

1. Choose **Create policy**\.

## Grant access to Amazon S3 bucket with account\-level public access block<a name="mwaa-create-role-s3-publicaccessblock"></a>

 You might want to block access to all buckets in your account by using the [https://docs.aws.amazon.com/AmazonS3/latest/API/API_control_PutPublicAccessBlock.html](https://docs.aws.amazon.com/AmazonS3/latest/API/API_control_PutPublicAccessBlock.html) Amazon S3 operation\. When you block access to all buckets in your account, your environment execution role must include the `s3:GetAccountPublicAccessBlock` action in a permission policy\. 

 The following example demonstrates the policy you must attach to your execution role when blocking access to all Amazon S3 buckets in your account\. 

```
{
  "Version": "2012-10-17",
  "Statement": [
     {
       "Effect": "Allow",
       "Action": "s3:GetAccountPublicAccessBlock",
       "Resource": "*"
     }
  ]
}
```

 For more information about restricting access to your Amazon S3 buckets, see [Blocking public access to your Amazon S3 storage](https://docs.aws.amazon.com/) in the *Amazon Simple Storage Service User Guide*\. 

## Use Apache Airflow connections<a name="mwaa-create-role-airflow-connections"></a>

You can also create an Apache Airflow connection and specify your execution role and its ARN in your Apache Airflow connection object\. To learn more, see [Managing connections to Apache Airflow](manage-connections.md)\.

## Sample JSON policies for an execution role<a name="mwaa-create-role-json"></a>

The sample permission policies in this section show two policies you can use to replace the permissions policy used for your existing execution role, or to create a new execution role and use for your environment\. These policies contain [Resource ARN](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_resource.html) placeholders for Apache Airflow log groups, an [Amazon S3 bucket](mwaa-s3-bucket.md), and an [Amazon MWAA environment](create-environment.md)\.

We recommend copying the example policy, replacing the sample ARNs or placeholders, then using the JSON policy to create or update an execution role\. For example, replacing `{your-region}` with `us-east-1`\.

### Sample policy for a customer managed key<a name="mwaa-create-role-cmk"></a>

The following example shows an execution role policy you can use for an [Customer managed key](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk)\. 

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "airflow:PublishMetrics",
            "Resource": "arn:aws:airflow:{your-region}:{your-account-id}:environment/{your-environment-name}"
        },
        { 
            "Effect": "Deny",
            "Action": "s3:ListAllMyBuckets",
            "Resource": [
                "arn:aws:s3:::{your-s3-bucket-name}",
                "arn:aws:s3:::{your-s3-bucket-name}/*"
            ]
        }, 
        { 
            "Effect": "Allow",
            "Action": [ 
                "s3:GetObject*",
                "s3:GetBucket*",
                "s3:List*"
            ],
            "Resource": [
                "arn:aws:s3:::{your-s3-bucket-name}",
                "arn:aws:s3:::{your-s3-bucket-name}/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "logs:CreateLogGroup",
                "logs:PutLogEvents",
                "logs:GetLogEvents",
                "logs:GetLogRecord",
                "logs:GetLogGroupFields",
                "logs:GetQueryResults"
            ],
            "Resource": [
                "arn:aws:logs:{your-region}:{your-account-id}:log-group:airflow-{your-environment-name}-*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "logs:DescribeLogGroups"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetAccountPublicAccessBlock"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": "cloudwatch:PutMetricData",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "sqs:ChangeMessageVisibility",
                "sqs:DeleteMessage",
                "sqs:GetQueueAttributes",
                "sqs:GetQueueUrl",
                "sqs:ReceiveMessage",
                "sqs:SendMessage"
            ],
            "Resource": "arn:aws:sqs:{your-region}:*:airflow-celery-*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "kms:Decrypt",
                "kms:DescribeKey",
                "kms:GenerateDataKey*",
                "kms:Encrypt"
            ],
            "Resource": "arn:aws:kms:{your-region}:{your-account-id}:key/{your-kms-cmk-id}",
            "Condition": {
                "StringLike": {
                    "kms:ViaService": [
                        "sqs.{your-region}.amazonaws.com",
                        "s3.{your-region}.amazonaws.com"
                    ]
                }
            }
        }
    ]
}
```

Next, you need to allow Amazon MWAA to assume this role in order to perform actions on your behalf\. This can be done by adding `"airflow.amazonaws.com"` and `"airflow-env.amazonaws.com"` service principals to the list of trusted entities for this execution role [using the IAM console](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_create_for-service.html#roles-creatingrole-service-console), or by placing these service principals in the assume role policy document for this execution role via the IAM [create\-role](https://docs.aws.amazon.com/cli/latest/reference/iam/create-role.html) command using the AWS CLI\. A sample assume role policy document can be found below:

```
{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {
            "Service": ["airflow.amazonaws.com","airflow-env.amazonaws.com"]
        },
        "Action": "sts:AssumeRole"
      }
   ]
}
```

Then attach the following JSON policy to your [Customer managed key](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk)\. This policy uses the [https://docs.aws.amazon.com/kms/latest/developerguide/policy-conditions.html#conditions-kms-encryption-context](https://docs.aws.amazon.com/kms/latest/developerguide/policy-conditions.html#conditions-kms-encryption-context) condition key prefix to permit access to your Apache Airflow logs group in CloudWatch Logs\.

```
{
    "Sid": "Allow logs access",
    "Effect": "Allow",
    "Principal": {
        "Service": "logs.{your-region}.amazonaws.com"
    },
    "Action": [
        "kms:Encrypt*",
        "kms:Decrypt*",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*",
        "kms:Describe*"
    ],
    "Resource": "*",
    "Condition": {
        "ArnLike": {
            "kms:EncryptionContext:aws:logs:arn": "arn:aws:logs:{your-region}:{your-account-id}:*"
        }
    }
}
```

### Sample policy for an AWS owned key<a name="mwaa-create-role-aocmk"></a>

The following example shows an execution role policy you can use for an [AWS owned key](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk)\. 

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "airflow:PublishMetrics",
            "Resource": "arn:aws:airflow:{your-region}:{your-account-id}:environment/{your-environment-name}"
        },
        { 
            "Effect": "Deny",
            "Action": "s3:ListAllMyBuckets",
            "Resource": [
                "arn:aws:s3:::{your-s3-bucket-name}",
                "arn:aws:s3:::{your-s3-bucket-name}/*"
            ]
        },
        { 
            "Effect": "Allow",
            "Action": [ 
                "s3:GetObject*",
                "s3:GetBucket*",
                "s3:List*"
            ],
            "Resource": [
                "arn:aws:s3:::{your-s3-bucket-name}",
                "arn:aws:s3:::{your-s3-bucket-name}/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "logs:CreateLogGroup",
                "logs:PutLogEvents",
                "logs:GetLogEvents",
                "logs:GetLogRecord",
                "logs:GetLogGroupFields",
                "logs:GetQueryResults"
            ],
            "Resource": [
                "arn:aws:logs:{your-region}:{your-account-id}:log-group:airflow-{your-environment-name}-*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "logs:DescribeLogGroups"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetAccountPublicAccessBlock"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": "cloudwatch:PutMetricData",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "sqs:ChangeMessageVisibility",
                "sqs:DeleteMessage",
                "sqs:GetQueueAttributes",
                "sqs:GetQueueUrl",
                "sqs:ReceiveMessage",
                "sqs:SendMessage"
            ],
            "Resource": "arn:aws:sqs:{your-region}:*:airflow-celery-*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "kms:Decrypt",
                "kms:DescribeKey",
                "kms:GenerateDataKey*",
                "kms:Encrypt"
            ],
            "NotResource": "arn:aws:kms:*:{your-account-id}:key/*",
            "Condition": {
                "StringLike": {
                    "kms:ViaService": [
                        "sqs.{your-region}.amazonaws.com"
                    ]
                }
            }
        }
    ]
}
```

## What's next?<a name="mwaa-create-role-next-up"></a>
+ Learn about the required permissions you and your Apache Airflow users need to access your environment in [Accessing an Amazon MWAA environment](access-policies.md)\.
+ Learn about [Customer managed keys for Data Encryption](custom-keys-certs.md)\.
+ Explore more [Customer managed policy examples](https://docs.aws.amazon.com/kms/latest/developerguide/customer-managed-policies.html)\.