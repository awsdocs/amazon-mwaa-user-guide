# Amazon MWAA network access<a name="configuring-networking"></a>

The Amazon Managed Workflows for Apache Airflow \(MWAA\) console contains built\-in options to configure private or public network access to the Apache Airflow UI\. This guide describes the private and public network options, and the additional resources you'll need to securely access remote instances should you choose the private network option\.

**Topics**
+ [Private or public network](#configuring-networking-onconsole)
+ [How it works](#configuring-networking-how)
+ [Setup for a private or public network](#access-network-choose)
+ [Configuring private network access](#configuring-tips)
+ [\(Optional\) Controlling private network access using a VPC endpoint policy](#configuring-private-policies)

## Private or public network<a name="configuring-networking-onconsole"></a>

The following image shows where to find the **Public network** and **Private network** options on the Amazon MWAA console\.

![\[This image shows where to find the Private network option on the Amazon MWAA console.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-private-public.png)

## How it works<a name="configuring-networking-how"></a>

Amazon MWAA provides private and public networking options for your Apache Airflow web server\.
+ **Public network**\. A public network allows the Apache Airflow UI to be accessed *over the Internet* by users granted access to the [IAM policy for your environment](access-policies.md)\. You can create multiple IAM policies for your users to configure access to your environment and Apache Airflow UI\. When you choose this option, Amazon MWAA attaches an [Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) with an HTTPS endpoint for your Apache Airflow *Web server*\. 

  A public network doesn't require any additional setup to use an environment or the Apache Airflow UI\. We recommend choosing the **Public network** option: if [VPC transit routing](https://docs.aws.amazon.com/vpc/latest/tgw/how-transit-gateways-work.html) is not required for your users in IAM\. 
+ **Private network**\. A private network limits access of the Apache Airflow UI to users *within your Amazon VPC* that have been granted access to the [IAM policy for your environment](access-policies.md)\. You can create multiple IAM policies for your users to configure access to your environment and Apache Airflow UI\. When you choose this option, Amazon MWAA additional Amazon VPC endpoint via [VPC endpoint services \(AWS PrivateLink\)](https://docs.aws.amazon.com/vpc/latest/userguide/endpoint-service-overview.html) to enable network access to the *Web server*\.

  A private network requires additional setup to use an environment and the Apache Airflow UI\. We recommend choosing the **Private network** option: if [VPC transit routing](https://docs.aws.amazon.com/vpc/latest/tgw/how-transit-gateways-work.html) is already configured or required for your users in IAM\.

## Setup for a private or public network<a name="access-network-choose"></a>

The following section describes the additional setup and configurations you should consider for a private or public network for your Apache Airflow *Web server*\.

### Using a public network<a name="access-network-public"></a>

If you choose the **Public network** option for your Apache Airflow *Web server*, you can begin using the Apache Airflow UI after you create your environment\. You'll need to take additional steps to configure access for your users, and permission for your environment to use other AWS services\.

1. **Add permissions**\. Amazon MWAA needs permission to use other AWS services\. When you create an environment, Amazon MWAA creates a [service\-linked role](mwaa-slr.md) that allows it to use certain IAM actions for Amazon Elastic Container Registry \(Amazon ECR\), CloudWatch Logs, and Amazon EC2\. 

   You can add permission to use additional actions for these services, or to use other AWS services by adding permissions to your execution role\. To learn more, see [Amazon MWAA Execution role](mwaa-create-role.md)\.

1. **Create user policies**\. You may need to create multiple IAM policies for your users to configure access to your environment and Apache Airflow UI\. To learn more, see [Accessing an Amazon MWAA environment](access-policies.md)\.

### Using a private network<a name="access-network-private"></a>

If you choose the **Private network** option for your Apache Airflow *Web server*, you'll need to take additional steps to configure access for your users, permission for your environment to use other AWS services, configure network access, and create a mechanism to enable network traffic to your [VPC endpoint \(AWS PrivateLink\)](https://docs.aws.amazon.com/vpc/latest/userguide/endpoint-service-overview.html)\.

1. **Add permissions**\. Amazon MWAA needs permission to use other AWS services\. When you create an environment, Amazon MWAA creates a [service\-linked role](mwaa-slr.md) that allows it to use certain IAM actions for Amazon Elastic Container Registry \(Amazon ECR\), CloudWatch Logs, and Amazon EC2\. 

   You can add permission to use additional actions for these services, or to use other AWS services by adding permissions to your execution role\. To learn more, see [Amazon MWAA Execution role](mwaa-create-role.md)\.

1. **Create user policies**\. You may need to create multiple IAM policies for your users to configure access to your environment and Apache Airflow UI\. To learn more, see [Accessing an Amazon MWAA environment](access-policies.md)\.

1. **Configure network access**\. The [VPC endpoint \(AWS PrivateLink\)](https://docs.aws.amazon.com/vpc/latest/userguide/endpoint-service-overview.html) for your environment needs to either use the same VPC security group as an Amazon MWAA environment, or specify rules in the VPC endpoint security group that allows it to connect to Amazon MWAA's VPC security group\. To configure rules for the VPC endpoint security group, see [Control access to services: Security groups](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-access.html#vpc-endpoints-security-groups)\.

1. **Enable network traffic**\. Amazon MWAA needs a mechanism in your Amazon VPC to connect a user outside the VPC to the [VPC endpoint \(AWS PrivateLink\)](https://docs.aws.amazon.com/vpc/latest/userguide/endpoint-service-overview.html) for your environment\. One such mechanism can be a Linux Bastion Host with an SSH tunnel and a SOCKS proxy management add\-on to control the proxy settings in your browser\. A bastion is an Amazon EC2 instance running in your Amazon VPC that acts as a proxy to perform this routing\. To create a Linux Bastion Host for your Amazon MWAA environment, see [Tutorial: Configuring private network access using a Linux Bastion Host](tutorials-private-network-bastion.md)\.
**Note**  
You can use other mechanisms, including either creating a public [Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/tutorial-application-load-balancer-cli.html), or a [Network Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/network-load-balancer-cli.html)\.

## Configuring private network access<a name="configuring-tips"></a>

The following section describes Amazon MWAA\-specific configurations you can use to configure private network access for an environment\.

### Using a Linux Bastion Host<a name="configuring-private-bastion"></a>

We recommend using the following tutorial to configure a Linux Bastion Host for your Amazon MWAA environment: [Tutorial: Configuring private network access using a Linux Bastion Host](tutorials-private-network-bastion.md)\.

### Using a Network Load Balancer or Application Load Balancer<a name="configuring-private-load-balancer"></a>

If you're using a public [Network Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/network-load-balancer-cli.html), or an [Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/tutorial-application-load-balancer-cli.html), you must use target groups that point to the private IP addresses for your Apache Airflow host name\.

1. Run a *nslookup* or *dig* command on the domain to get the IP addresses\. For example:

   ```
   dig +short 59438366-1087-4346-9f49-a4ccd16267b7-vpce.c9.us-east-1.airflow.amazonaws.com
   vpce-010587637a219be56-85lu8ws1.vpce-svc-0fbf48eec80c6f78f.us-east-1.vpce.amazonaws.com.
   10.192.21.23
   10.192.20.201
   ```

1. If you want to associate an ACM SSL/TLS certificate, you'll need to create a new domain for the HTTPS listener for your Network Load Balancer or Application Load Balancer\. To learn more, see [Create an HTTPS listener for your Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-https-listener.html), or [Create a listener for your Network Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/create-listener.html)\.

## \(Optional\) Controlling private network access using a VPC endpoint policy<a name="configuring-private-policies"></a>

You can optionally use a [VPC endpoint \(AWS PrivateLink\)](https://docs.aws.amazon.com/mwaa/latest/userguide/vpc-create.html#vpc-create-required) policy to control access to AWS services from your private subnet\. A VPC endpoint policy is an IAM resource policy that you attach to the endpoint for your private network\.

If you're using an Amazon S3 endpoint policy, you must configure the policy to:
+ Allow access to the [Amazon S3 bucket](mwaa-s3-bucket.md) where your DAGs and supporting files are stored\.
+ Allow access to the [Amazon Elastic Container Registry \(Amazon ECR\) service bucket](https://docs.aws.amazon.com/AmazonECR/latest/userguide/vpc-endpoints.html#ecr-minimum-s3-perms)\.

**Note**  
If an Amazon S3 endpoint policy is not configured to allow access to these buckets, the Amazon MWAA deployment will fail\.