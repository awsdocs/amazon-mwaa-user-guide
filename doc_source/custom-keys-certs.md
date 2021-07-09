# Customer managed CMKs for Data Encryption<a name="custom-keys-certs"></a>

You can optionally provide a  [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk) for data encryption on your environment\. Charges apply for the storage and use of encryption keys\. For more information, see [AWS KMS Pricing](http://aws.amazon.com/kms/pricing/)\.

A  [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk) must be created in the same Region as your Amazon MWAA environment instance and your Amazon S3 bucket where your customer data is stored\. If the  [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk) that you specify is in a different account from the one that you use to configure an environment, you must specify the key using its ARN\. For more information about creating keys, see [Creating Keys](https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html)

## What's supported<a name="custom-keys-grants-support"></a>


| AWS KMS feature | Supported | 
| --- | --- | 
|  An [AWS KMS key ID or ARN](https://docs.aws.amazon.com/kms/latest/developerguide/find-cmk-id-arn.html)\.  |  Yes  | 
|  An [AWS KMS key alias](https://docs.aws.amazon.com/kms/latest/developerguide/kms-alias.html)\.  |  No  | 
|  An [AWS KMS multi\-region key](https://docs.aws.amazon.com/kms/latest/developerguide/multi-region-keys-overview.html)\.  |  No  | 

## Using Grants for Encryption<a name="custom-keys-grants-provide"></a>

This topic describes the grants Amazon MWAA attaches to a  [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk) on your behalf for data encryption and decryption\.

### How it works<a name="custom-keys-certs-grants"></a>

There are two resource\-based access control mechanisms supported by KMS for  [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk): a [key policy](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html) and [grant](https://docs.aws.amazon.com/kms/latest/developerguide/grants.html)\.

A key policy is used when the permission is mostly static and used in synchronous service mode\. A grant is used when more dynamic and granular permissions are required, such as when a service needs to define different access permissions for itself or other accounts\.

Amazon MWAA uses and attaches four grant policies to your  [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk)\. This is due to the granular permissions required for an environment to encrypt data at rest from CloudWatch Logs, Amazon SQS queue, Aurora PostgreSQL database database, Secrets Manager secrets, Amazon S3 bucket and DynamoDB tables\.

When you create an Amazon MWAA environment and specify a  [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk), Amazon MWAA attaches the grant policies to your [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk)\. These policies allow Amazon MWAA in `airflow.{region}.amazonaws.com` to use your  [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk) to encrypt resources on your behalf that are owned by Amazon MWAA\. 

Additional grants are created and attached to a specified key on your behalf\. This includes policies to retire a grant if you delete your environment, to use your  [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk) for Client\-Side Encryption \(CSE\), and for the ECS Fargate execution role that needs to access secrets protected by your CMK in Secrets Manager\.

## Grant policies<a name="custom-keys-certs-grant-policies"></a>

We add the following [resource based policy](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_identity-vs-resource.html) grants on your behalf to a  [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk)\. These policies allow the grantee and retiring principal \(Amazon MWAA\) to perform actions defined in the policy\.

### Grant 1: For Data Plane Resource Creation<a name="custom-keys-certs-grant-policies-1"></a>

```
{
            "Name": "mwaa-grant-for-env-mgmt-role-{environment name}",
            "GranteePrincipal": "airflow.{region}.amazonaws.com",
            "RetiringPrincipal": "airflow.{region}.amazonaws.com",
            "Operations": [
              "kms:Encrypt",
              "kms:Decrypt",
              "kms:ReEncrypt*",
              "kms:GenerateDataKey*",
              "kms:CreateGrant",
              "kms:DescribeKey",
              "kms:RetireGrant"
            ]
          }
```

### Grant 2: For ControllerLambdaExecutionRole access<a name="custom-keys-certs-grant-policies-2"></a>

```
{
            "Name": "mwaa-grant-for-lambda-exec-{environment name}",
            "GranteePrincipal": "airflow.{region}.amazonaws.com",
            "RetiringPrincipal": "airflow.{region}.amazonaws.com",
            "Operations": [
              "kms:Encrypt",
              "kms:Decrypt",
              "kms:ReEncrypt*",
              "kms:GenerateDataKey*",
              "kms:DescribeKey",
              "kms:RetireGrant"
            ]
          }
```

### Grant 3: For CfnManagementLambdaExecutionRole access<a name="custom-keys-certs-grant-policies-3"></a>

```
{
              "Name": " mwaa-grant-for-cfn-mgmt-{environment name}",
              "GranteePrincipal": "airflow.{region}.amazonaws.com",
              "RetiringPrincipal": "airflow.{region}.amazonaws.com",
              "Operations": [
                "kms:Encrypt",
                "kms:Decrypt",
                "kms:ReEncrypt*",
                "kms:GenerateDataKey*",
                "kms:DescribeKey"
              ]
            }
```

### Grant 4: For ECS Fargate execution role to access secrets<a name="custom-keys-certs-grant-policies-4"></a>

```
{
                "Name": "mwaa-fargate-access-for-{environment name}",
                "GranteePrincipal": "airflow.{region}.amazonaws.com",
                "RetiringPrincipal": "airflow.{region}.amazonaws.com",
                "Operations": [
                  "kms:Encrypt",
                  "kms:Decrypt",
                  "kms:ReEncrypt*",
                  "kms:GenerateDataKey*",
                  "kms:DescribeKey",
                  "kms:RetireGrant"
                ]
              }
```

## Attaching key policies to a [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk)<a name="custom-keys-certs-grant-policies-attach"></a>

If you choose to use your own [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk) with Amazon MWAA you must attach the following policy to the key to allow Amazon MWAA to use it to encrypt your data\. If the [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk) you used for your Amazon MWAA environment is not already configured to work with CloudWatch, you must update the [key policy](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html) to allow for encrypted CloudWatch Logs\. For more information, see the [Encrypt Log Data in CloudWatch Using AWS Key Management Service Service](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/encrypt-log-data-kms.html)\.

The following example represents a key policy for CloudWatch Logs\. Substitute the sample values provided for the region\.

```
{
          "Effect": "Allow",
          "Principal": {
          "Service": "logs.us-west-2.amazonaws.com"
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
            "kms:EncryptionContext:aws:logs:arn": "arn:aws:logs:us-west-2:*:*"
            }
          }
        }
```