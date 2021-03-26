# Accessing an Amazon MWAA environment<a name="access-policies"></a>

To use Amazon Managed Workflows for Apache Airflow \(MWAA\), you must use an account, user, or role with the necessary permissions\. This page describes the access policies you can attach to your team and Apache Airflow users to access your Amazon Managed Workflows for Apache Airflow \(MWAA\) environment and its services\.

**Topics**
+ [How it works](#access-policies-how)
+ [Full console access policy: AmazonMWAAFullConsoleAccess](#console-full-access)
+ [Full API and console access policy: AmazonMWAAFullApiAccess](#full-access-policy)
+ [Read\-only console access policy: AmazonMWAAReadOnlyAccess](#mwaa-read-only)
+ [Apache Airflow UI access policy: AmazonMWAAWebServerAccess](#web-ui-access)
+ [Apache Airflow CLI policy: AmazonMWAAAirflowCliAccess](#cli-access)
+ [Creating a JSON policy](#access-policy-iam-console-create)
+ [Example use case to attach policies to a developer group](#access-policy-use-case)
+ [What's next?](#access-policy-next-up)

## How it works<a name="access-policies-how"></a>

The resources and services used in an Amazon MWAA environment are not accessible to all IAM entities \(users, roles, or groups\)\. You must create a policy that grants your Apache Airflow users permission to access these resources\. For example, you need to grant access to your Apache Airflow development team\.

Amazon MWAA uses these policies to validate whether a user has the permissions needed to perform an action on the AWS console or via the APIs used by an environment\.

You can use the JSON policies in this topic to create a policy for your Apache Airflow users in IAM, and then attach the policy to a user, group, or role in IAM\. Here are the policies available:
+ [AmazonMWAAFullConsoleAccess](#console-full-access) – A user may need access to this permissions policy if they need to configure an environment on the Amazon MWAA console\.
+ [AmazonMWAAFullApiAccess](#full-access-policy) – A user may need access to this permissions policy if they need access to all Amazon MWAA APIs used to manage an environment\.
+ [AmazonMWAAReadOnlyAccess](#mwaa-read-only) – A user may need access to this permissions policy if they need to view the resources used by an environment on the Amazon MWAA console\.
+ [AmazonMWAAWebServerAccess](#web-ui-access) – A user may need access to this permissions policy if they need to access the Apache Airflow UI\.
+ [AmazonMWAAAirflowCliAccess](#cli-access) – A user may need access to this permissions policy to run Apache Airflow CLI commands\.

The sample policies on this page contain placeholders\. For example, replace *\{your\-account\-id\}* with your account ID as `0123456789`\.

## Full console access policy: AmazonMWAAFullConsoleAccess<a name="console-full-access"></a>

A user may need access to the `AmazonMWAAFullConsoleAccess` permissions policy if they need to configure an environment on the Amazon MWAA console\. 

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "airflow:*",
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:ListRoles"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:CreateServiceLinkedRole"
            ],
            "Resource": "arn:aws:iam::*:role/aws-service-role/airflow.amazonaws.com/AWSServiceRoleForAmazonMWAA"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetBucketLocation",
                "s3:ListAllMyBuckets",
                "s3:ListBucket",
                "s3:ListBucketVersions"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:CreateBucket",
                "s3:PutObject",
                "s3:GetEncryptionConfiguration"
            ],
            "Resource": "arn:aws:s3:::*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeSubnets",
                "ec2:DescribeVpcs",
                "ec2:DescribeRouteTables"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:AuthorizeSecurityGroupIngress",
                "ec2:CreateSecurityGroup"
            ],
            "Resource": "arn:aws:ec2:*:*:security-group/airflow-security-group-*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "kms:ListAliases"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "kms:DescribeKey",
                "kms:ListGrants",
                "kms:CreateGrant",
                "kms:RevokeGrant",
                "kms:Decrypt", 
                "kms:Encrypt", 
                "kms:GenerateDataKey*", 
                "kms:ReEncrypt*"
            ],
            "Resource": "arn:aws:kms:*:{your-account-id}:key/{your-kms-id}"
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:PassRole"
            ],
            "Resource": "*",
            "Condition": {
                "StringLike": {
                    "iam:PassedToService": "airflow.amazonaws.com"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "iam:AttachRolePolicy",
                "iam:CreateRole"
            ],
            "Resource": "arn:aws:iam::{your-account-id}:policy/service-role/MWAA-Execution-Policy-*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetEncryptionConfiguration"
            ],
            "Resource": "arn:aws:s3:::*"
        },
        {
            "Effect": "Allow",
            "Action": "ec2:CreateVpcEndpoint",
            "Resource": [
                "arn:aws:ec2:*:*:vpc-endpoint/*",
                "arn:aws:ec2:*:*:vpc/*",
                "arn:aws:ec2:*:*:subnet/*",
                "arn:aws:ec2:*:*:security-group/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:CreateNetworkInterface"
            ],
            "Resource": [
                "arn:aws:ec2:*:*:subnet/*",
                "arn:aws:ec2:*:*:network-interface/*"
            ]
        }
    ]
}
```

## Full API and console access policy: AmazonMWAAFullApiAccess<a name="full-access-policy"></a>

A user may need access to the `AmazonMWAAFullApiAccess` permissions policy if they need access to all Amazon MWAA APIs used to manage an environment\. It does not grant permissions to access the Apache Airflow UI\. 

```
{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Effect":"Allow",
         "Action":"airflow:*",
         "Resource":"*"
      },
      {
         "Effect":"Allow",
         "Action":[
            "iam:CreateServiceLinkedRole"
         ],
         "Resource":"arn:aws:iam::*:role/aws-service-role/airflow.amazonaws.com/AWSServiceRoleForAmazonMWAA"
      },
      {
         "Effect":"Allow",
         "Action":[
            "ec2:DescribeSecurityGroups",
            "ec2:DescribeSubnets",
            "ec2:DescribeVpcs",
            "ec2:DescribeRouteTables"
         ],
         "Resource":"*"
      },
      {
         "Effect":"Allow",
         "Action":[
            "kms:DescribeKey",
            "kms:ListGrants",
            "kms:CreateGrant",
            "kms:RevokeGrant",
            "kms:Decrypt",
            "kms:Encrypt",
            "kms:GenerateDataKey*",
            "kms:ReEncrypt*"
         ],
         "Resource":"arn:aws:kms:*:{your-account-id}:key/{your-kms-id}"
      },
      {
         "Effect":"Allow",
         "Action":[
            "iam:PassRole"
         ],
         "Resource":"*",
         "Condition":{
            "StringLike":{
               "iam:PassedToService":"airflow.amazonaws.com"
            }
         }
      },
      {
         "Effect":"Allow",
         "Action":[
            "s3:GetEncryptionConfiguration"
         ],
         "Resource":"arn:aws:s3:::*"
      },
      {
         "Effect":"Allow",
         "Action":"ec2:CreateVpcEndpoint",
         "Resource":[
            "arn:aws:ec2:*:*:vpc-endpoint/*",
            "arn:aws:ec2:*:*:vpc/*",
            "arn:aws:ec2:*:*:subnet/*",
            "arn:aws:ec2:*:*:security-group/*"
         ]
      },
      {
         "Effect":"Allow",
         "Action":[
            "ec2:CreateNetworkInterface"
         ],
         "Resource":[
            "arn:aws:ec2:*:*:subnet/*",
            "arn:aws:ec2:*:*:network-interface/*"
         ]
      }
   ]
}
```

## Read\-only console access policy: AmazonMWAAReadOnlyAccess<a name="mwaa-read-only"></a>

A user may need access to the `AmazonMWAAReadOnlyAccess` permissions policy if they need to view the resources used by an environment on the Amazon MWAA console environment details page\. It doesn't allow a user to create new environments, edit existing environments, or allow a user to view the Apache Airflow UI\.

```
{
        "Version": "2012-10-17",
        "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "airflow:ListEnvironments",
                "airflow:GetEnvironment",
                "airflow:ListTagsForResource"
            ],
            "Resource": "*"
        }
    ]
}
```

## Apache Airflow UI access policy: AmazonMWAAWebServerAccess<a name="web-ui-access"></a>

A user may need access to the `AmazonMWAAWebServerAccess` permissions policy if they need to access the Apache Airflow UI\. It does not allow the user to view environments on the Amazon MWAA console or use the Amazon MWAA APIs to perform any actions\. Specify the `Admin`, `Op`, `User`, `Viewer` or the `Public` role in `{airflow-role}` to customize the level of access for the user of the web token\. For more information, see [Default Roles](https://airflow.apache.org/docs/apache-airflow/1.10.6/security.html?highlight=ldap#default-roles) in the *Apache Airflow reference guide*\.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "airflow:CreateWebLoginToken",
            "Resource": [
                "arn:aws:airflow:{your-region}:{your-account-id}:role/{your-environment-name}/{airflow-role}"
            ]
        }
    ]
}
```

**Note**  
Amazon MWAA does not support custom Apache Airflow role\-based access control \(RBAC\) roles as of yet\.

## Apache Airflow CLI policy: AmazonMWAAAirflowCliAccess<a name="cli-access"></a>

A user may need access to the `AmazonMWAAAirflowCliAccess` permissions policy if they need to run Apache Airflow CLI commands \(such as `trigger_dag`\)\. It does not allow the user to view environments on the Amazon MWAA console or use the Amazon MWAA APIs to perform any actions\. 

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "airflow:CreateCliToken"
            ],
            "Resource": "*"
        }
    ]
}
```

## Creating a JSON policy<a name="access-policy-iam-console-create"></a>

You can create the JSON policy, and attach the policy to your user, role, or group on the IAM console\. The following steps describe how to create a JSON policy in IAM\.

**To create the JSON policy**

1. Open the [Policies page](https://console.aws.amazon.com/iam/home#/policies) on the IAM console\.

1. Choose **Create policy**\.

1. Choose the **JSON** tab\.

1. Add your JSON policy\.

1. Choose **Review policy**\.

1. Enter a value in the text field for **Name** and **Description** \(optional\)\.

   For example, you could name the policy `AmazonMWAAReadOnlyAccess`\.

1. Choose **Create policy**\.

## Example use case to attach policies to a developer group<a name="access-policy-use-case"></a>

Let's say you're using a group in IAM named `AirflowDevelopmentGroup` to apply permissions to all of the developers on your Apache Airflow development team\. These users need access to the `AmazonMWAAFullConsoleAccess`, `AmazonMWAAAirflowCliAccess`, and `AmazonMWAAWebServerAccess` permission policies\. This section describes how to create a group in IAM, create and attach these policies, and associate the group to an IAM user\. The steps assume you're using an [AWS owned CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk)\.

**To create the AmazonMWAAFullConsoleAccess policy**

1. Download the [AmazonMWAAFullConsoleAccess access policy](./samples/AmazonMWAAFullConsoleAccess.zip)\.

1. Open the [Policies page](https://console.aws.amazon.com/iam/home#/policies) on the IAM console\.

1. Choose **Create policy**\.

1. Choose the **JSON** tab\.

1. Paste the JSON policy for `AmazonMWAAFullConsoleAccess`\.

1. Substitute the following values:

   1. *\{your\-account\-id\}* – your AWS account ID \(such as `0123456789`\)

   1. *\{your\-kms\-id\}* – the `aws/airflow` identifer for an [AWS owned CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk)

1. Choose the **Review policy**\.

1. Type `AmazonMWAAFullConsoleAccess` in **Name**\.

1. Choose **Create policy**\.

**To create the AmazonMWAAWebServerAccess policy**

1. Download the [AmazonMWAAWebServerAccess access policy](./samples/AmazonMWAAWebServerAccess.zip)\.

1. Open the [Policies page](https://console.aws.amazon.com/iam/home#/policies) on the IAM console\.

1. Choose **Create policy**\.

1. Choose the **JSON** tab\.

1. Paste the JSON policy for `AmazonMWAAWebServerAccess`\.

1. Substitute the following values:

   1. *\{your\-region\}* – the region of your Amazon MWAA environment \(such as `us-east-1`\)

   1. *\{your\-account\-id\}* – your AWS account ID \(such as `0123456789`\)

   1. *\{your\-environment\-name\}* – your Amazon MWAA environment name \(such as `MyAirflowEnvironment`\)

   1. *\{airflow\-role\}* – the `Admin` Apache Airflow [Default Role](https://airflow.apache.org/docs/apache-airflow/1.10.6/security.html?highlight=ldap#default-roles)

1. Choose **Review policy**\.

1. Type `AmazonMWAAWebServerAccess` in **Name**\.

1. Choose **Create policy**\.

**To create the AmazonMWAAAirflowCliAccess policy**

1. Download the [AmazonMWAAAirflowCliAccess access policy](./samples/AmazonMWAAAirflowCliAccess.zip)\.

1. Open the [Policies page](https://console.aws.amazon.com/iam/home#/policies) on the IAM console\.

1. Choose **Create policy**\.

1. Choose the **JSON** tab\.

1. Paste the JSON policy for `AmazonMWAAAirflowCliAccess`\.

1. Choose the **Review policy**\.

1. Type `AmazonMWAAAirflowCliAccess` in **Name**\.

1. Choose **Create policy**\.

**To create the group**

1. Open the [Groups page](https://console.aws.amazon.com/iam/home#/groups) on the IAM console\.

1. Type a name of `AirflowDevelopmentGroup`\.

1. Choose **Next Step**\.

1. Type `AmazonMWAA` to filter results in **Filter**\.

1. Select the three policies you created\.

1. Choose **Next Step**\.

1. Choose **Create Group**\.

**To associate to a user**

1. Open the [Users page](https://console.aws.amazon.com/iam/home#/users) on the IAM console\.

1. Choose a user\.

1. Choose **Groups**\.

1. Choose **Add user to groups**\.

1. Select the **AirflowDevelopmentGroup**\.

1. Choose **Add to Groups**\.

## What's next?<a name="access-policy-next-up"></a>
+ Learn how to generate a token to access the Apache Airflow UI in [Accessing the Apache Airflow UI](access-airflow-ui.md)\.
+ Learn more about creating IAM policies in [Creating IAM policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html)\.