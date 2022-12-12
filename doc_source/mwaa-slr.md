# Service\-linked role for Amazon MWAA<a name="mwaa-slr"></a>

Amazon Managed Workflows for Apache Airflow \(MWAA\) uses AWS Identity and Access Management \(IAM\)[ service\-linked roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_terms-and-concepts.html#iam-term-service-linked-role)\. A service\-linked role is a unique type of IAM role that is linked directly to Amazon MWAA\. Service\-linked roles are predefined by Amazon MWAA and include all the permissions that the service requires to call other AWS services on your behalf\. 

A service\-linked role makes setting up Amazon MWAA easier because you don’t have to manually add the necessary permissions\. Amazon MWAA defines the permissions of its service\-linked roles, and unless defined otherwise, only Amazon MWAA can assume its roles\. The defined permissions include the trust policy and the permissions policy, and that permissions policy cannot be attached to any other IAM entity\.

You can delete a service\-linked role only after first deleting their related resources\. This protects your Amazon MWAA resources because you can't inadvertently remove permission to access the resources\.

For information about other services that support service\-linked roles, see [AWS Services That Work with IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_aws-services-that-work-with-iam.html) and look for the services that have **Yes **in the **Service\-linked roles** column\. Choose a **Yes** with a link to view the service\-linked role documentation for that service\.

## Service\-linked role permissions for Amazon MWAA<a name="mwaa-slr-iam-policy"></a>

Amazon MWAA uses the service\-linked role named `AWSServiceRoleForAmazonMWAA` – The service\-linked role created in your account grants Amazon MWAA access to the following AWS services:
+ Amazon CloudWatch Logs \(CloudWatch Logs\) – To create log groups for Apache Airflow logs\.
+ Amazon CloudWatch \(CloudWatch\) – To publish metrics related to your environment and its underlying components to your account\.
+ Amazon Elastic Compute Cloud \(Amazon EC2\) – To create the following resources:
  + An Amazon VPC endpoint in your VPC for an AWS\-managed Amazon Aurora PostgreSQL database cluster to be used by the Apache Airflow *Scheduler* and *Worker*\.
  + An additional Amazon VPC endpoint to enable network access to the *Web server* if you choose the [private network](configuring-networking.md) option for your Apache Airflow *Web server*\.
  + [Elastic Network Interfaces \(ENIs\)](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_ElasticNetworkInterfaces.html) in your Amazon VPC to enable network access to AWS resources hosted in your Amazon VPC\.

 The following trust policy allows the service principal to assume the service\-linked role\. The service principal for Amazon MWAA is `airflow.amazonaws.com` as demonstrated by the policy\. 

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "airflow.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

The role permissions policy named `AmazonMWAAServiceRolePolicy` allows Amazon MWAA to complete the following actions on the specified resources:

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
        },
        {
            "Effect": "Allow",
            "Action": "cloudwatch:PutMetricData",
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "cloudwatch:namespace": [
                        "AWS/MWAA"
                    ]
                }
            }
        }
    ]
}
```

You must configure permissions to allow an IAM entity \(such as a user, group, or role\) to create, edit, or delete a service\-linked role\. For more information, see [Service\-linked role permissions](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#service-linked-role-permissions) in the *IAM User Guide*\.

## Creating a service\-linked role for Amazon MWAA<a name="mwaa-slr-create-slr"></a>

You don't need to manually create a service\-linked role\. When you create a new Amazon MWAA environment using the AWS Management Console, the AWS CLI, or the AWS API, Amazon MWAA creates the service\-linked role for you\. 

If you delete this service\-linked role, and then need to create it again, you can use the same process to recreate the role in your account\. When you create another environment, Amazon MWAA creates the service\-linked role for you again\. 

## Editing a service\-linked role for Amazon MWAA<a name="mwaa-slr-edit-slr"></a>

Amazon MWAA does not allow you to edit the AWSServiceRoleForAmazonMWAA service\-linked role\. After you create a service\-linked role, you cannot change the name of the role because various entities might reference the role\. However, you can edit the description of the role using IAM\. For more information, see [Editing a service\-linked role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#edit-service-linked-role) in the *IAM User Guide*\.

## Deleting a service\-linked role for Amazon MWAA<a name="mwaa-slr-delete-slr"></a>

 If you no longer need to use a feature or service that requires a service\-linked role, we recommend that you delete that role\. That way you don’t have an unused entity that is not actively monitored or maintained\. 

 When you delete an Amazon MWAA environment, Amazon MWAA deletes all the associated resources it uses as a part of the service\. However, you must wait before Amazon MWAA completes deleting your environment, before attempting to delete the service\-linked role\. If you delete the service\-linked role before Amazon MWAA deletes the environment, Amazon MWAA might be unable to delete all of the environment's associated resources\. 

**To manually delete the service\-linked role using IAM**

Use the IAM console, the AWS CLI, or the AWS API to delete the AWSServiceRoleForAmazonMWAA service\-linked role\. For more information, see [Deleting a service\-linked role](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html#delete-service-linked-role) in the *IAM User Guide*\.

## Supported regions for Amazon MWAA service\-linked roles<a name="mwaa-slr-regions"></a>

Amazon MWAA supports using service\-linked roles in all of the regions where the service is available\. For more information, see [Amazon Managed Workflows for Apache Airflow \(MWAA\) endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/mwaa.html)\.

## Policy updates<a name="mwaa-slr-policies-updates"></a>


| Change | Description | Date | 
| --- | --- | --- | 
|  Amazon MWAA update its service\-linked role permission policy  |   [`AmazonMWAAServiceRolePolicy`](#mwaa-slr-iam-policy) – Amazon MWAA updates the permission policy for its service\-linked role to grant Amazon MWAA permission to publish additional metrics related to the service's underlying resources to customer accounts\. These new metrics are published under the `AWS/MWAA`   |  November 18, 2022  | 
|  Amazon MWAA started tracking changes  |  Amazon MWAA started tracking changes for its AWS managed service\-linked role permission policy\.  |  November 18, 2022  | 