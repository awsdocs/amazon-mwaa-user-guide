# Amazon MWAA Service\-linked role policy<a name="mwaa-slr"></a>

Amazon MWAA creates a [service\-linked role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html) in your account when you create an Amazon MWAA environment\. This page describes the service\-linked role and how to view the role on the IAM console\.

**Topics**
+ [How it works](#mwaa-slr-iam-how)
+ [The permissions policy](#mwaa-slr-iam-policy)
+ [What's next?](#mwaa-slr-next-up)

## How it works<a name="mwaa-slr-iam-how"></a>

Amazon MWAA creates a service\-linked role in your account to allow Amazon MWAA to use other AWS services used by your environment during its provisioning\. The service role enables permission to the following AWS services:
+ Amazon CloudWatch Logs \(CloudWatch Logs\) – to create log groups for Apache Airflow logs\.
+ Amazon Elastic Compute Cloud \(Amazon EC2\) – to create the following resources:
  + An Amazon VPC endpoint in your VPC for an AWS\-managed Amazon Aurora PostgreSQL database cluster to be used by the Apache Airflow *Scheduler* and *Worker*\.
  + An additional Amazon VPC endpoint to enable network access to the *Web server* if you choose the [private network](configuring-networking.md) option for your Apache Airflow *Web server*\.
  + [Elastic Network Interfaces \(ENIs\)](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_ElasticNetworkInterfaces.html) in your Amazon VPC to enable network access to AWS resources hosted in your Amazon VPC\.

The **Private network** option for an Apache Airflow *Web server* requires additional setup to use an environment and the Apache Airflow UI\. To learn more, see [Apache Airflow access modes](configuring-networking.md)\.

## The permissions policy<a name="mwaa-slr-iam-policy"></a>

 The service\-linked role enables permission to certain IAM actions in CloudWatch Logs, and Amazon EC2\. The following is the JSON policy attached to your environment's service\-linked role\. You can also view the JSON policy and the IAM actions for the `AWSServiceRoleForAmazonMWAA` on the IAM console\. 

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "logs:CreateLogGroup",
                "logs:DescribeLogGroups"
            ],
            "Resource": "arn:aws:logs:*:*:log-group:airflow-*:*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:AttachNetworkInterface",
                "ec2:CreateNetworkInterface",
                "ec2:CreateNetworkInterfacePermission",
                "ec2:DeleteNetworkInterface",
                "ec2:DeleteNetworkInterfacePermission",
                "ec2:DescribeDhcpOptions",
                "ec2:DescribeNetworkInterfaces",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeSubnets",
                "ec2:DescribeVpcEndpoints",
                "ec2:DescribeVpcs",
                "ec2:DetachNetworkInterface"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": "ec2:CreateVpcEndpoint",
            "Resource": "arn:aws:ec2:*:*:vpc-endpoint/*",
            "Condition": {
                "ForAnyValue:StringEquals": {
                    "aws:TagKeys": "AmazonMWAAManaged"
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:ModifyVpcEndpoint",
                "ec2:DeleteVpcEndpoints"
            ],
            "Resource": "arn:aws:ec2:*:*:vpc-endpoint/*",
            "Condition": {
                "Null": {
                    "aws:ResourceTag/AmazonMWAAManaged": false
                }
            }
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:CreateVpcEndpoint",
                "ec2:ModifyVpcEndpoint"
            ],
            "Resource": [
                "arn:aws:ec2:*:*:vpc/*",
                "arn:aws:ec2:*:*:security-group/*",
                "arn:aws:ec2:*:*:subnet/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": "ec2:CreateTags",
            "Resource": "arn:aws:ec2:*:*:vpc-endpoint/*",
            "Condition": {
                "StringEquals": {
                    "ec2:CreateAction": "CreateVpcEndpoint"
                },
                "ForAnyValue:StringEquals": {
                    "aws:TagKeys": "AmazonMWAAManaged"
                }
            }
        }
    ]
}
```

**To view the policy using the AWS Management Console**

1. Open the [AWSServiceRoleForAmazonMWAA](https://console.aws.amazon.com/iam/home#/roles/AWSServiceRoleForAmazonMWAA) role on the IAM console\.

1. Choose the **AWSServiceRoleForAmazonMWAA** policy\.

   This shows the permissions that are applied when an AWS service attempts to access the environment\.

1. Choose the **Access Advisor** tab\.

   This shows the service permissions granted to the role\.

## What's next?<a name="mwaa-slr-next-up"></a>
+ Learn how to add permissions to allow Amazon MWAA to use other AWS resources used by your environment in [Amazon MWAA execution role](mwaa-create-role.md)\.
+ Learn more about how IAM actions work in [IAM JSON policy elements: Action](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_action.html)\.