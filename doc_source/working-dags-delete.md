# Deleting files on Amazon S3<a name="working-dags-delete"></a>

This page describes how versioning works in an Amazon S3 bucket for an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment, and the steps to delete a DAG, `plugins.zip`, or `requirements.txt` file\.

**Contents**
+ [Prerequisites](#working-dags-delete-prereqs)
+ [Versioning overview](#working-dags-delete-overview)
+ [How it works](#working-dags-delete-how)
+ [Deleting a DAG on Amazon S3](#working-dags-s3-dag-delete)
+ [Removing a "current" requirements\.txt or plugins\.zip from an environment](#working-dags-s3-delete-version-c)
+ [Deleting a "non\-current" \(previous\) requirements\.txt or plugins\.zip version](#working-dags-s3-delete-version-p)
+ [Using lifecycles to delete "non\-current" \(previous\) versions and delete markers automatically](#working-dags-s3-delete-lifecycle)
+ [Example lifecycle policy to delete requirements\.txt "non\-current" versions and delete markers automatically](#working-dags-s3-delete-lifecycle-ex)
+ [What's next?](#working-dags-s3-delete-next-up)

## Prerequisites<a name="working-dags-delete-prereqs"></a>

You'll need the following before you can complete the steps on this page\.

1. **Access**\. Your AWS account must have been granted access by your administrator to the [AmazonMWAAFullConsoleAccess](access-policies.md#console-full-access) access control policy for your environment\.

1. **Amazon S3 configurations**\. The [Amazon S3 bucket](mwaa-s3-bucket.md) used to store your DAGs, custom plugins in `plugins.zip`, and Python dependencies in `requirements.txt` must be configured with *Public Access Blocked* and *Versioning Enabled*\.

1. **Permissions**\. Your Amazon MWAA environment must be permitted by your [execution role](mwaa-create-role.md) to access the AWS resources used by your environment\.

## Versioning overview<a name="working-dags-delete-overview"></a>

The `requirements.txt` and `plugins.zip` in your Amazon S3 bucket are versioned\. When Amazon S3 bucket versioning is enabled for an object, and an artifact \(for example, plugins\.zip\) is deleted from an Amazon S3 bucket, the file doesn't get deleted entirely\. Anytime an artifact is deleted on Amazon S3, a new copy of the file is created that is a 404 \(Object not found\) error/0k file that says "I'm not here\." Amazon S3 calls this a *delete marker*\. A delete marker is a "null" version of the file with a key name \(or key\) and version ID like any other object\.

We recommend deleting file versions and delete markers periodically to reduce storage costs for your Amazon S3 bucket\. To delete "non\-current" \(previous\) file versions entirely, you must delete the versions of the file\(s\), and then the *delete marker* for the version\.

## How it works<a name="working-dags-delete-how"></a>

Amazon MWAA runs a [sync](https://docs.aws.amazon.com/v2/documentation/api/latest/reference/s3/sync.html) operation on your Amazon S3 bucket every thirty seconds\. This causes any DAG deletions in an Amazon S3 bucket to be synced to the Airflow image of your Fargate container\.

For `plugins.zip` and `requirements.txt` files, changes occur only after an environment update when Amazon MWAA builds a new Airflow image of your Fargate container with the custom plugins and Python dependencies\. If you delete the *current* version of any of a `requirements.txt` or `plugins.zip` file, and then update your environment without providing a new version for the deleted file, then the update will fail with an error message, such as, "Unable to read version `{version}` of file `{file}`"\.

## Deleting a DAG on Amazon S3<a name="working-dags-s3-dag-delete"></a>

A DAG file \(`.py`\) is not versioned and can be deleted directly on the Amazon S3 console\. The following steps describe how to delete a DAG on your Amazon S3 bucket\.

**To delete a DAG**

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Select the **S3 bucket** link in the **DAG code in S3** pane to open your storage bucket on the Amazon S3 console\.

1. Choose the `dags` folder\.

1. Select the DAG, **Delete**\.

1. Under **Delete objects?**, type `delete`\.

1. Choose **Delete objects**\.

**Note**  
Apache Airflow preserves historical DAG runs\. After a DAG has been run in Apache Airflow, it remains in the Airflow DAGs list regardless of the file status, until you delete it in Apache Airflow\. To delete a DAG in Apache Airflow, choose the red "delete" button under the **Links** column\.

## Removing a "current" requirements\.txt or plugins\.zip from an environment<a name="working-dags-s3-delete-version-c"></a>

Currently, there isn't a way to remove a plugins\.zip or requirements\.txt from an environment after they’ve been added, but we're working on the issue\. In the interim, a workaround is to point to an empty text or zip file, respectively\.

## Deleting a "non\-current" \(previous\) requirements\.txt or plugins\.zip version<a name="working-dags-s3-delete-version-p"></a>

The `requirements.txt` and `plugins.zip` files in your Amazon S3 bucket are versioned on Amazon MWAA\. If you want to delete these files on your Amazon S3 bucket entirely, you must retrieve the current version \(121212\) of the object \(for example, plugins\.zip\), delete the version, and then remove the *delete marker* for the file version\(s\)\.

You can also delete "non\-current" \(previous\) file versions on the Amazon S3 console; however, you'll still need to delete the *delete marker* using one of the following options\.
+ To retrieve the object version, see [Retrieving object versions from a versioning\-enabled bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/RetrievingObjectVersions.html) *in the Amazon S3 guide*\.
+ To delete the object version, see [Deleting object versions from a versioning\-enabled bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/DeletingObjectVersions.html) *in the Amazon S3 guide*\.
+ To remove a delete marker, see [Removing delete markers](https://docs.aws.amazon.com/AmazonS3/latest/userguide/RemDelMarker.html) *in the Amazon S3 guide*\.

## Using lifecycles to delete "non\-current" \(previous\) versions and delete markers automatically<a name="working-dags-s3-delete-lifecycle"></a>

You can configure a lifecycle policy for your Amazon S3 bucket to delete "non\-current" \(previous\) versions of the plugins\.zip and requirements\.txt files in your Amazon S3 bucket after a certain number of days, or to remove an expired object's delete marker\.

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Under **DAG code in Amazon S3**, choose your Amazon S3 bucket\.

1. Choose **Create lifecycle rule**\.

## Example lifecycle policy to delete requirements\.txt "non\-current" versions and delete markers automatically<a name="working-dags-s3-delete-lifecycle-ex"></a>

The following example shows how to create a lifecycle rule that permanently deletes "non\-current" versions of a requirements\.txt file and their delete markers after thirty days\.

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Under **DAG code in Amazon S3**, choose your Amazon S3 bucket\.

1. Choose **Create lifecycle rule**\.

1. In **Lifecycle rule name**, type `Delete previous requirements.txt versions and delete markers after thirty days`\.

1. In **Prefix**, **requirements**\.

1. In **Lifecycle rule actions**, choose **Permanently delete previous versions of objects** and **Delete expired delete markers or incomplete multipart uploads**\.

1. In **Number of days after objects become previous versions**, type `30`\.

1. In **Expired object delete markers**, choose **Delete expired object delete markers, objects are permanently deleted after 30 days**\.

## What's next?<a name="working-dags-s3-delete-next-up"></a>
+ Learn more about Amazon S3 delete markers in [Managing delete markers](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/create-lifecycle.html)\.
+ Learn more about Amazon S3 lifecycles in [Expiring objects](https://docs.aws.amazon.com/AmazonS3/latest/userguide/lifecycle-expire-general-considerations.html)\.