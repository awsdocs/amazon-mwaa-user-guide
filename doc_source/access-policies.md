# Accessing an Amazon MWAA environment<a name="access-policies"></a>

To use Amazon Managed Workflows for Apache Airflow \(MWAA\), you must use an account, user, or role with the necessary permissions\. This page describes the access policies you can attach to your team and Apache Airflow users to access your Amazon Managed Workflows for Apache Airflow \(MWAA\) environment and its services\.

**Topics**
+ [How it works](#access-policies-how)
+ [Full console access policy: AmazonMWAAFullConsoleAccess](#console-full-access)
+ [Full API and console access policy: AmazonMWAAFullApiAccess](#full-access-policy)
+ [Read\-only console access policy: AmazonMWAAReadOnlyAccess](#mwaa-read-only)
+ [Apache Airflow UI access policy: AmazonMWAAWebServerAccess](#web-ui-access)
+ [Creating a JSON policy](#access-policy-iam-console-create)
+ [What's next?](#access-policy-next-up)

## How it works<a name="access-policies-how"></a>

The resources and services used in an Amazon MWAA environment are not accessible to all IAM entities \(users, roles, or groups\)\. You must create a policy that grants your Apache Airflow users permission to access these resources\. For example, you need to grant access to your Apache Airflow development team\.

Amazon MWAA uses these policies to validate whether a user has the permissions needed to perform an action on the AWS console or via the APIs used by an environment\.

You can use the JSON policies in this topic to create a policy for your Apache Airflow users in IAM, and then attach the policy to a user, group, or role in IAM\. Here are the policies available:
+ [AmazonMWAAFullConsoleAccess](#console-full-access) – A user may need access to this permissions policy if they need to configure an environment on the Amazon MWAA console\.
+ [AmazonMWAAFullApiAccess](#full-access-policy) – A user may need access to this permissions policy if they need access to all Amazon MWAA APIs used to manage an environment\.
+ [AmazonMWAAReadOnlyAccess](#mwaa-read-only) – A user may need access to this permissions policy if they need to view the resources used by an environment on the Amazon MWAA console\.
+ [AmazonMWAAWebServerAccess](#web-ui-access) – A user may need access to this permissions policy if they need to access the Apache Airflow UI\.

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

A user may need access to the `AmazonMWAAFullApiAccess` permissions policy if they need access to all Amazon MWAA APIs used to manage an environment\. This policy does not grant permissions to access the Apache Airflow UI\. 

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
                    "iam:CreateServiceLinkedRole"
                ],
                "Resource": "arn:aws:iam::*:role/aws-service-role/airflow.amazonaws.com/AWSServiceRoleForAmazonMWAA"
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

## Read\-only console access policy: AmazonMWAAReadOnlyAccess<a name="mwaa-read-only"></a>

A user may need access to the `AmazonMWAAReadOnlyAccess` permissions policy if they need to view the resources used by an environment on the Amazon MWAA console environment details page\. This policy does not allow a user to create new environments, edit existing environments, or allow read access to the Apache Airflow UI\.

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

A user may need access to the `AmazonMWAAWebServerAccess` permissions policy if they need to access the Apache Airflow UI\. This policy does not allow the user to view environments on the Amazon MWAA console or use the Amazon MWAA APIs to perform any actions\. Specify the `Admin`, `Op`, `User`, `Viewer` or the `Public` role in `{airflow-role}`\.

**Note**  
Amazon MWAA does not support custom Apache Airflow role\-based access control \(RBAC\) roles as of yet\. 

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

## Creating a JSON policy<a name="access-policy-iam-console-create"></a>

You can create the JSON policy, and attach the policy to your user, role, or group on the IAM console\. The following procedure demonstrates how to attach a JSON policy to a role\.

**To create the JSON policy**

1. Open the [Policies page](https://console.aws.amazon.com/iam/home#/policies) on the IAM console\.

1. Choose **Create policy**\.

1. Choose the **JSON** tab\.

1. Add your JSON policy\.

1. Choose the **Review policy**\.

1. Enter a value in the text field for **Name** and ****Description \(optional\)\.

   For example, you could name the policy `AmazonMWAAReadOnlyAccess`\.

1. Choose the **Create policy**\.

## What's next?<a name="access-policy-next-up"></a>
+ Learn how to attach the policy you created to a user in [IAM Tutorial: Create and attach your first customer managed policy](https://docs.aws.amazon.com/https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_managed-policies.html)\.
+ Learn how to generate a token to access the Apache Airflow UI in [Accessing the Apache Airflow UI](access-airflow-ui.md)\.
+ Learn more about creating IAM policies in [Creating IAM policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_create.html)\.