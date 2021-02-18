# Amazon MWAA Execution role<a name="mwaa-create-role"></a>

An execution role is an AWS Identity and Access Management \(IAM\) role with a permissions policy that grants Amazon Managed Workflows for Apache Airflow \(MWAA\) permission to invoke the resources of other AWS services on your behalf\. This can include resources such as your Amazon S3 bucket, [AWS owned CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk), and CloudWatch Logs\. Amazon MWAA environments need one execution role per environment\. This page describes how to use the Amazon MWAA console to create an execution role, and the steps to modify a permissions policy in JSON and update your execution role\.

**Topics**
+ [How it works](#mwaa-create-role-how)
+ [Create a new role](#mwaa-create-role-mwaa-onconsole)
+ [Viewing and updating an execution role policy](#mwaa-create-role-update)
+ [Using Apache Airflow connections](#mwaa-create-role-airflow-connections)
+ [Sample JSON policies for an execution role](#mwaa-create-role-json)
+ [What's next?](#mwaa-create-role-next-up)

## How it works<a name="mwaa-create-role-how"></a>

You can use the Amazon MWAA console to create an execution role and an [AWS owned CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk) on your behalf when you create a MWAA environment\.
+ When you choose the **Create new role** option on the console, Amazon MWAA attaches the minimal permissions needed by an environment to your execution role\.
+ In some cases, Amazon MWAA attaches the maximum permissions\. For example, if you choose the option on the Amazon MWAA console to create an execution role, Amazon MWAA adds the permissions policies for all CloudWatch Logs groups automatically by using the regex pattern in the execution role as `"arn:aws:logs:your-region:your-account-id:log-group:airflow-your-environment-name-*"`\.
+ Amazon MWAA can't add or edit permission policies to an existing execution role after an environment is created\. You must update your execution role with additional permission policies needed by your environment\. For example, if your DAG requires access to Glue, Amazon MWAA can't automatically detect these permissions are required by your environment and add the permissions to your execution role\.

The sample permission policies on this page show two policies you can use to replace the permissions policy used for your existing execution role, or to create a new execution role\. In detail:
+ **Updating an existing role** – If you are modifying an existing execution role for your environment, you can use the sample [JSON policy documents](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html) on this page to either add to or replace the JSON policy of your execution role on the IAM console\. Assuming the execution role is already associated to your environment, Amazon MWAA can start using the added permission policies immediately\. This also means if you remove any permissions from your execution role that are required, your DAG executions may fail\.
+ **Creating a new role** – If you are creating a new execution role, you can use the default option on the Amazon MWAA console to create an execution role, then use the steps on this page to modify your permissions policy\.

You can change the execution role for your environment at any time\. If a new execution role is not already associated with your environment, use the steps on this page to associate a new execution role and specify the role when you create your environment\.

## Create a new role<a name="mwaa-create-role-mwaa-onconsole"></a>

By default, Amazon MWAA creates an [AWS owned CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk) for data encryption and an execution role on your behalf\. You can choose the default options on the Amazon MWAA console when you create an environment\. The following image shows the default option to create an execution role for an environment\.

![\[This is an image with the default option to create a new role.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-permissions.png)

## Viewing and updating an execution role policy<a name="mwaa-create-role-update"></a>

You can view the execution role for your environment on the Amazon MWAA console, and update the JSON policy for the role on the IAM console\.

**To update a JSON policy**

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose the policy name\.

1. Choose **Edit policy**\.

1. Choose the **JSON** tab\.

1. Update your JSON policy\.

1. Choose **Review policy**\.

1. Choose **Save changes**\.

## Using Apache Airflow connections<a name="mwaa-create-role-airflow-connections"></a>

You can also add your execution role to your Apache Airflow UI\. If your DAG requires access to any AWS resources that are not already permitted by your execution role, you can use Apache Airflow connections to pass your execution role credentials into your environment\. You can then use this connection by referring to the connection in your DAG\(s\)\.

While Amazon MWAA does not support environment variables directly, you can add your execution role and its ARN as an Airflow configuration option on the Amazon MWAA console\.

To learn more, see [Amazon Web Services Connection](https://airflow.apache.org/docs/apache-airflow-providers-amazon/stable/connections/aws.html) in the *Apache Airflow reference guide*\.

## Sample JSON policies for an execution role<a name="mwaa-create-role-json"></a>

The sample policies in this section contain [Resource ARN](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_resource.html) placeholders for Apache Airflow log groups, an [Amazon S3 bucket](mwaa-s3-bucket.md), and an [Amazon MWAA environment](create-environment.md)\.

We recommend copying the example policy, replacing the sample ARNs or placeholders, then using the JSON policy to create or update an execution role\. For example, replacing `{your-region}` with `us-east-1`\.

### Sample policy for a customer managed CMK<a name="mwaa-create-role-cmk"></a>

The following example shows an execution role policy you can use for an [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk)\. 

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

Then attach the following JSON policy to your [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk)\. This policy uses the [https://docs.aws.amazon.com/kms/latest/developerguide/policy-conditions.html#conditions-kms-encryption-context](https://docs.aws.amazon.com/kms/latest/developerguide/policy-conditions.html#conditions-kms-encryption-context) condition key prefix to permit access to your Apache Airflow logs group in CloudWatch Logs\.

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

### Sample policy for an AWS owned CMK<a name="mwaa-create-role-aocmk"></a>

The following example shows an execution role policy you can use for an [AWS owned CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk)\. 

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
                "arn:aws:logs:{{region}}:{{accountId}}:log-group:airflow-{{envName}}-*"
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
+ Learn about [Customer managed CMKs for Data Encryption](custom-keys-certs.md)\.
+ Explore more [Customer managed policy examples](https://docs.aws.amazon.com/kms/latest/developerguide/customer-managed-policies.html)\.