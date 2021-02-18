# How Amazon MWAA works with IAM<a name="security_iam_service-with-iam"></a>

Amazon MWAA uses IAM identity\-based policies to grant permissions to Amazon MWAA actions and resources\.

To get a high\-level view of how Amazon MWAA and other AWS services work with IAM, see [AWS Services That Work with IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_aws-services-that-work-with-iam.html) in the *IAM User Guide*\.

## Amazon MWAA identity\-based policies<a name="security_iam_service-with-iam-id-based-policies"></a>

With IAM identity\-based policies, you can specify allowed or denied actions and resources, as well as the conditions under which actions are allowed or denied\. Amazon MWAA supports specific actions, resources, and condition keys\.

To learn about all of the elements that you use in a JSON policy, see [IAM JSON Policy Elements Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html) in the *IAM User Guide*\.

### Actions<a name="security_iam_service-with-iam-id-based-policies-actions"></a>

Administrators can use AWS JSON policies to specify who has access to what\. That is, which **principal** can perform **actions** on what **resources**, and under what **conditions**\.

The `Action` element of a JSON policy describes the actions that you can use to allow or deny access in a policy\. Policy actions usually have the same name as the associated AWS API operation\. There are some exceptions, such as *permission\-only actions* that don't have a matching API operation\. There are also some operations that require multiple actions in a policy\. These additional actions are called *dependent actions*\.

Include actions in a policy to grant permissions to perform the associated operation\.

Policy statements must include either an `Action` element or a `NotAction` element\. The `Action` element lists the actions allowed by the policy\. The `NotAction` element lists the actions that are not allowed\.

The actions defined for Amazon MWAA reflect tasks that you can perform using Amazon MWAA\. Policy actions in Detective have the following prefix: `airflow:`\.

You can also use wildcards \(\*\) to specify multiple actions\. Instead of listing these actions separately, you can grant access to all actions that end with the word, for example, `environment`\.

To see a list of Amazon MWAA actions, see [Actions Defined by Amazon Managed Workflows for Apache Airflow](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_mwaa.html#mwaa-actions-as-permissions) in the *IAM User Guide*\.