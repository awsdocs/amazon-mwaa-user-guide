# Amazon MWAA Service\-linked role policy<a name="mwaa-slr"></a>

Amazon MWAA creates a [service\-linked role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html) in your account when you create an Amazon MWAA environment\. This page describes the service\-linked role and how to view the role on the IAM console\.

**Topics**
+ [How it works](#mwaa-slr-iam-how)
+ [Viewing the permissions policy](#mwaa-slr-iam-policy)
+ [What's next?](#mwaa-slr-next-up)

## How it works<a name="mwaa-slr-iam-how"></a>

Amazon MWAA creates and attaches a JSON policy to your account's service\-linked role to allow Amazon MWAA to use other AWS services used by your Amazon MWAA environment\. For example, permission to CloudWatch logs and the VPC network for your environment\.

## Viewing the permissions policy<a name="mwaa-slr-iam-policy"></a>

You can view the JSON policy for the `AWSServiceRoleForAmazonMWAA` on the IAM console\. The following procedure demonstrates how to attach a JSON policy to a role\.

**To view the JSON policy**

1. Open the [AWSServiceRoleForAmazonMWAA](https://console.aws.amazon.com/iam/home#/roles/AWSServiceRoleForAmazonMWAA) role on the IAM console\.

1. Choose the **AWSServiceRoleForAmazonMWAA** policy\.

   This shows the permissions that are applied when an AWS service attempts to access the environment\.

1. Choose the **Access Advisor** tab\.

   This shows the service permissions granted to the role\.

## What's next?<a name="mwaa-slr-next-up"></a>
+ Learn how to add permissions to allow Amazon MWAA to use other AWS resources used by your environment in [Amazon MWAA Execution role](mwaa-create-role.md)\.