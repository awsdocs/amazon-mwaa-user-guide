# Security in your VPC on Amazon MWAA<a name="vpc-security"></a>

This page describes the Amazon VPC components used to secure your Amazon Managed Workflows for Apache Airflow \(MWAA\) environment and the configurations needed for these components\.

**Contents**
+ [Terms](#networking-security-defs)
+ [Security overview](#vpc-security-about)
+ [Network access control lists \(ACLs\)](#vpc-security-acl)
  + [\(Recommended\) Example ACLs](#vpc-security-acl-example)
+ [VPC security groups](#vpc-security-sg)
  + [\(Recommended\) Example security group all access](#vpc-security-sg-example)
  + [\(Optional\) Example security group that restricts inbound access to port 5432](#vpc-security-sg-example-port5432)
  + [\(Optional\) Example security group that restricts inbound access to port 443](#vpc-security-sg-example-port443)
+ [VPC endpoint policies \(private routing only\)](#vpc-external-vpce-policies)
  + [\(Recommended\) Example VPC endpoint policy to allow all access](#vpc-external-vpce-policies-all)
  + [\(Recommended\) Example Amazon S3 gateway endpoint policy to allow bucket access](#vpc-external-vpce-policies-s3)

## Terms<a name="networking-security-defs"></a>

**Public routing**  
An Amazon VPC network that has access to the Internet\. 

**Private routing**  
An Amazon VPC network **without** access to the Internet\.

## Security overview<a name="vpc-security-about"></a>

Security groups and access control lists \(ACLs\) provide ways to control the network traffic across the subnets and instances in your Amazon VPC using rules you specify\.
+ Network traffic to and from a subnet can be controlled by Access Control Lists \(ACLs\)\. You only need one ACL, and the same ACL can be used on multiple environments\.
+ Network traffic to and from an instance can be controlled by an Amazon VPC security group\. You can use between one to five security groups per environment\.
+ Network traffic to and from an instance can also be controlled by VPC endpoint policies\. If Internet access within your Amazon VPC is not allowed by your organization and you're using an Amazon VPC network with private routing, a VPC endpoint policy is required for each of the VPC endpoints you created for each AWS service used by your environment\.

## Network access control lists \(ACLs\)<a name="vpc-security-acl"></a>

A [network access control list \(ACL\)](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html) can manage \(by allow or deny rules\) inbound and outbound traffic at the *subnet* level\. An ACL is stateless, which means that inbound and outbound rules must be specified separately and explicitly\. It is used to specify the types of network traffic that are allowed in or out from the instances in a VPC network\. 

Every Amazon VPC has a default ACL that allows all inbound and outbound traffic\. You can edit the default ACL rules, or create a custom ACL and attach it to your subnets\. A subnet can only have one ACL attached to it at any time, but one ACL can be attached to multiple subnets\.

### \(Recommended\) Example ACLs<a name="vpc-security-acl-example"></a>

The following example shows the *inbound* and *outbound* ACL rules that can be used for an Amazon VPC for an Amazon VPC with *public routing* or *private routing*\.


| Rule number | Type | Protocol | Port range | Source | Allow/Deny | 
| --- | --- | --- | --- | --- | --- | 
|  100  |  All IPv4 traffic  |  All  |  All  |  0\.0\.0\.0/0  |  Allow  | 
|  \*  |  All IPv4 traffic  |  All  |  All  |  0\.0\.0\.0/0  |  Deny  | 

## VPC security groups<a name="vpc-security-sg"></a>

A [VPC security group](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html) acts as a virtual firewall that controls the network traffic at the *instance* level\. A security group is stateful, which means that when an inbound connection is permitted, it is allowed to reply\. It is used to specify the types of network traffic that are allowed in from the instances in a VPC network\. 

Every Amazon VPC has a default security group\. By default, it has no inbound rules\. It has an outbound rule that allows all outbound traffic\. You can edit the default security group rules, or create a custom security group and attach it to your Amazon VPC\. On Amazon MWAA, you need to configure inbound and outbound rules to direct traffic on your NAT gateways\.

### \(Recommended\) Example security group all access<a name="vpc-security-sg-example"></a>

The following example shows the *inbound* security group rules that allows all traffic for an Amazon VPC for an Amazon VPC with *public routing* or *private routing*\.


| Type | Protocol | Source Type | Source | 
| --- | --- | --- | --- | 
|  All traffic  |  All  |  All  |  sg\-0909e8e81919 / my\-mwaa\-vpc\-security\-group  | 

The following example shows the *outbound* security group rules\.


| Type | Protocol | Source Type | Source | 
| --- | --- | --- | --- | 
|  All traffic  |  All  |  All  |  0\.0\.0\.0/0  | 

### \(Optional\) Example security group that restricts inbound access to port 5432<a name="vpc-security-sg-example-port5432"></a>

The following example shows the *inbound* security group rules that allow all HTTPS traffic on port 5432 for the Amazon Aurora PostgreSQL metadata database\. 

**Note**  
If you choose to restrict traffic using this rule, you'll need to add another rule to allow TCP traffic on port 443\.


| Type | Protocol | Port range | Source type | Source | 
| --- | --- | --- | --- | --- | 
|  HTTPS  |  TCP  |  5432  |  Custom  |  sg\-0909e8e81919 / my\-mwaa\-vpc\-security\-group  | 

### \(Optional\) Example security group that restricts inbound access to port 443<a name="vpc-security-sg-example-port443"></a>

The following example shows the *inbound* security group rules that allow all TCP traffic on port 443 for the Apache Airflow *Web server*\. 


| Type | Protocol | Port range | Source type | Source | 
| --- | --- | --- | --- | --- | 
|  Custom TCP  |  TCP  |  443  |  Custom  |  sg\-0909e8e81919 / my\-mwaa\-vpc\-security\-group  | 

## VPC endpoint policies \(private routing only\)<a name="vpc-external-vpce-policies"></a>

A [VPC endpoint \(AWS PrivateLink\)](https://docs.aws.amazon.com/mwaa/latest/userguide/vpc-create.html#vpc-create-required) policy controls access to AWS services from your private subnet\. A VPC endpoint policy is an IAM resource policy that you attach to your VPC gateway or interface endpoint\. This section describes the permissions needed for the VPC endpoint policies for each VPC endpoint\. 

We recommend using a VPC interface endpoint policy for each of the VPC endpoints you created that allows full access to all AWS services, and using your execution role exclusively for AWS permissions\.

### \(Recommended\) Example VPC endpoint policy to allow all access<a name="vpc-external-vpce-policies-all"></a>

The following example shows a VPC interface endpoint policy for an Amazon VPC with *private routing*\.

```
{
    "Statement": [
        {
            "Action": "*",
            "Effect": "Allow",
            "Resource": "*",
            "Principal": "*"
        }
    ]
}
```

### \(Recommended\) Example Amazon S3 gateway endpoint policy to allow bucket access<a name="vpc-external-vpce-policies-s3"></a>

The following example shows a VPC gateway endpoint policy that provides access to the Amazon S3 buckets required for Amazon ECR operations for an Amazon VPC with *private routing*\. This is required for your Amazon ECR image to be retrieved, in addition to the bucket where your DAGs and supporting files are stored\.

```
{
  "Statement": [
    {
      "Sid": "Access-to-specific-bucket-only",
      "Principal": "*",
      "Action": [
        "s3:GetObject"
      ],
      "Effect": "Allow",
      "Resource": ["arn:aws:s3:::prod-region-starport-layer-bucket/*"]
    }
  ]
}
```