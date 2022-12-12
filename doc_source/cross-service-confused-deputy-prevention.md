# Cross\-service confused deputy prevention<a name="cross-service-confused-deputy-prevention"></a>

The confused deputy problem is a security issue where an entity that doesn't have permission to perform an action can coerce a more\-privileged entity to perform the action\. In AWS, cross\-service impersonation can result in the confused deputy problem\. Cross\-service impersonation can occur when one service \(the *calling service*\) calls another service \(the *called service*\)\. The calling service can be manipulated to use its permissions to act on another customer's resources in a way it should not otherwise have permission to access\. To prevent this, AWS provides tools that help you protect your data for all services with service principals that have been given access to resources in your account\. 

We recommend using the [https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourcearn](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourcearn) and [https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourceaccount](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourceaccount) global condition context keys in your environment' execution role to limit the permissions that Amazon MWAA gives another service to access the resource\. Use `aws:SourceArn` if you want only one resource to be associated with the cross\-service access\. Use `aws:SourceAccount` if you want to allow any resource in that account to be associated with the cross\-service use\.

The most effective way to protect against the confused deputy problem is to use the `aws:SourceArn` global condition context key with the full ARN of the resource\. If you don't know the full ARN of the resource or if you are specifying multiple resources, use the `aws:SourceArn` global context condition key with wildcard characters \(`*`\) for the unknown portions of the ARN\. For example, `arn:aws:airflow:*:123456789012:environment/*`\. 

The value of `aws:SourceArn` must be your Amazon MWAA environment ARN, for which you are creating an execution role\.

The following example shows how you can use the `aws:SourceArn` and `aws:SourceAccount` global condition context keys in your environment's execution role trust policy to prevent the confused deputy problem\. You can use the following trust policy when you create a new execution role\.

```
{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": {
            "Service": ["airflow.amazonaws.com","airflow-env.amazonaws.com"]
        },
        "Action": "sts:AssumeRole",
        "Condition":{
            "ArnLike":{
               "aws:SourceArn":"arn:aws:airflow:your-region:123456789012:environment/your-environment-name"
            },
            "StringEquals":{
               "aws:SourceAccount":"123456789012"
            }
         }
      }
   ]
}
```