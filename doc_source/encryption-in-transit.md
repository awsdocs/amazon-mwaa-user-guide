# Encryption in Transit<a name="encryption-in-transit"></a>

Data in transit is referred to as data that may be intercepted as it travels the network\. This page describes how encryption and decryption works for data in transit on an Amazon MWAAenvironment\.

Transport Layer Security \(TLS\) encrypts the Amazon MWAA objects in transit between Fargate containers and Amazon S3\. For in\-depth information about Amazon S3 encryption, see [Protecting Data Using Encryption](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingEncryption.html)\.

You specify the encryption artifacts used for in\-transit encryption in one of two ways: either by using an  [AWS owned CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#aws-owned-cmk) at the time you create an environment\. In which case, Amazon MWAA takes care of data encryption in\-transit with other AWS services\. Or by using a  [Customer managed CMK](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#customer-cmk), in which case, Amazon MWAA attaches grant policies to the key you provided\.