# Create an Amazon S3 bucket for Amazon MWAA<a name="mwaa-s3-bucket"></a>

Amazon MWAA uses an Amazon S3 bucket to store your DAGs and associated support files\. You must create an S3 bucket or an environment before you can create the environment\. The name of the bucket must be globally unique\. To use a bucket with a Amazon MWAA environment, you must create the bucket in the same Region where you create the environment\.

Buckets have configuration properties, including geographical Region, access settings for the objects in the bucket, and other metadata\.

**To create a bucket**

1. Sign in to the AWS Management Console and open the Amazon S3 console at [https://console\.aws\.amazon\.com/s3/](https://console.aws.amazon.com/s3/)\.

1. Choose **Create bucket**\.

1. In **Bucket name**, enter a DNS\-compliant name for your bucket\.

   The bucket name must:
   + Be unique across all of Amazon S3\.
   + Be between 3 and 63 characters long\.
   + Not contain uppercase characters\.
   + Start with a lowercase letter or number\.

   After you create the bucket, you can't change its name\. For information about naming buckets, see [Rules for bucket naming](https://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html#bucketnamingrules) in the *Amazon Simple Storage Service Developer Guide*\.
**Important**  
Avoid including sensitive information, such as account numbers, in the bucket name\. The bucket name is visible in the URLs that point to the objects in the bucket\.

1. In **Region**, choose the AWS Region where you want the bucket to reside\. This must be the same Region where you create your Amazon MWAA environment\. You must create a separate bucket for each Region where you plan to use Amazon MWAA\.

   Choose a Region close to you to minimize latency and costs and address regulatory requirements\. Objects stored in a Region never leave that Region unless you explicitly transfer them to another Region\. For a list of Amazon S3 AWS Regions, see [AWS service endpoints](https://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region) in the *Amazon Web Services General Reference*\.

1. In **Bucket settings for Block Public Access**, Amazon MWAA requires that the bucket does not allow public access\. You should leave all settings enabled\. For more information about blocking public access, see [Using Amazon S3 Block Public Access](https://docs.aws.amazon.com/AmazonS3/latest/dev/access-control-block-public-access.html) in the *Amazon Simple Storage Service Developer Guide*\.

1. For **Bucket Versioning**, choose **Enable**\.

1. Optional\. For **Tags**, add any tags as appropriate for your environment\. You can also add Tags to your Amazon MWAA environment\.

1. For **Default encryption**, choose whether to enable server\-side encryption for the bucket\. If you choose to enable server\-side encryption, you must use the same key for your Amazon S3 bucket and Amazon MWAA environment\.

1. \(Optional\) If you want to enable S3 Object lock:

   1. Choose **Advanced settings**, and read the message that appears\.
**Important**  
You can only enable S3 Object lock for a bucket when you create it\. If you enable Object lock for the bucket, you can't disable it later\. Enabling Object lock also enables versioning for the bucket\. After you enable Object lock for the bucket, you must configure the Object lock settings before any objects in the bucket are protected\. For more information about configuring protection for objects, see [How do I lock an Amazon S3 object?](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/object-lock.html)\.

   1. If you want to enable S3 Object lock, enter *enable* in the text box and choose **Confirm**\.

   For more information about the S3 Object lock feature, see [Locking Objects Using Amazon S3 Object Lock](https://docs.aws.amazon.com/AmazonS3/latest/dev/object-lock.html) in the *Amazon Simple Storage Service Developer Guide*\.

1. Choose **Create bucket**\.

## What's next?<a name="mwaa-s3-bucket-next-up"></a>
+ Learn about the VPC networking components needed for a Amazon MWAA environment in [Create the VPC network](vpc-create.md)\.
+ Learn how to how to manage access permissions in [How do I set ACL bucket permissions?](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/set-bucket-permissions.html)\.
+ Learn how to delete a storage bucket in [How do I delete an S3 Bucket?](https://docs.aws.amazon.com/AmazonS3/latest/user-guide/delete-bucket.html)\.