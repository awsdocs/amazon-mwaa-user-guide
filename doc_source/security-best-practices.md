# Best practices<a name="security-best-practices"></a>

Amazon MWAA provides a number of security features to consider as you develop and implement your own security policies\. The following best practices are general guidelines and don’t represent a complete security solution\. Because these best practices might not be appropriate or sufficient for your environment, treat them as helpful considerations rather than prescriptions\.
+ Use least\-permissive permission policies\. Grant permissions to only the resources or actions that users need to perform tasks\.
+ Use AWS CloudTrail to monitor user activity in your account\.
+ Ensure that the Amazon S3 bucket policy and object ACLs grant permissions to the users from the associated Amazon MWAA environment to put objects into the bucket\. This ensures that users with permissions to add workflows to the bucket also have permissions to run the workflows in Airflow\.
+ Use the Amazon S3 buckets associated with Amazon MWAA environments\. Your Amazon S3 bucket can be any name\. Do not store other objects in the bucket, or use the bucket with another service\. 
