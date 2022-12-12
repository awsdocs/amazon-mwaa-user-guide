# Apache Airflow access modes<a name="configuring-networking"></a>

The Amazon Managed Workflows for Apache Airflow \(MWAA\) console contains built\-in options to configure private or public routing to the Apache Airflow *web server* on your environment\. This guide describes the access modes available for the Apache Airflow *Web server* on your Amazon Managed Workflows for Apache Airflow \(MWAA\) environment, and the additional resources you'll need to configure in your Amazon VPC if you choose the private network option\.

**Contents**
+ [Apache Airflow access modes](#configuring-networking-onconsole)
  + [Public network](#webserver-options-public-network-onconsole)
  + [Private network](#webserver-options-private-network)
+ [Access modes overview](#configuring-networking-access-overview)
  + [Public network access mode](#access-overview-public)
  + [Private network access mode](#access-overview-private)
+ [Setup for private and public access modes](#access-network-choose)
  + [Setup for public network](#access-network-public)
  + [Setup for private network](#access-network-private)
+ [Accessing the VPC endpoint for your Apache Airflow Web server \(private network access\)](#configuring-access-vpce)

## Apache Airflow access modes<a name="configuring-networking-onconsole"></a>

You can choose private or public routing for your Apache Airflow *Web server*\. To enable private routing, choose **Private network**\. This limits user access to an Apache Airflow *Web server* to within an Amazon VPC\. To enable public routing, choose **Public network**\. This allows users to access the Apache Airflow *Web server* over the Internet\. 

### Public network<a name="webserver-options-public-network-onconsole"></a>

 The following architectural diagram shows an Amazon MWAA environment with a public web server\. 

![\[This image shows the architecture for an Amazon MWAA environment with a private web server.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-public-web-server.png)

The public network access mode allows the Apache Airflow UI to be accessed *over the internet* by users granted access to the [IAM policy for your environment](access-policies.md)\.

The following image shows where to find the **Public network** option on the Amazon MWAA console\. 

![\[This image shows where to find the Public network option on the Amazon MWAA console.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-public-network.png)

### Private network<a name="webserver-options-private-network"></a>

 The following architectural diagram shows an Amazon MWAA environment with a private web server\. 

![\[This image shows the architecture for an Amazon MWAA environment with a private web server.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-private-web-server.png)

The private network access mode limits access of the Apache Airflow UI to users *within your Amazon VPC* that have been granted access to the [IAM policy for your environment](access-policies.md)\.

The following image shows where to find the **Private network** option on the Amazon MWAA console\. 

![\[This image shows where to find the Private network option on the Amazon MWAA console.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-private-network.png)

## Access modes overview<a name="configuring-networking-access-overview"></a>

This section describes the VPC endpoints \(AWS PrivateLink\) created in your Amazon VPC when you choose the **Public network** or **Private network** access mode\. 

### Public network access mode<a name="access-overview-public"></a>

If you chose the **Public network** access mode for your Apache Airflow *Web server*, network traffic is publicly routed *over the Internet*\.
+ Amazon MWAA creates a VPC interface endpoint for your Amazon Aurora PostgreSQL metadata database\. The endpoint is created in the Availability Zones mapped to your private subnets and is independent from other AWS accounts\. 
+ Amazon MWAA then binds an IP address from your private subnets to the interface endpoints\. This is designed to support the best practice of binding a single IP from each Availability Zone of the Amazon VPC\.

### Private network access mode<a name="access-overview-private"></a>

 If you chose the **Private network** access mode for your Apache Airflow *Web server*, network traffic is privately routed *within your Amazon VPC*\. 
+ Amazon MWAA creates a VPC interface endpoint for your Apache Airflow *Web server*, and an interface endpoint for your Amazon Aurora PostgreSQL metadata database\. The endpoints are created in the Availability Zones mapped to your private subnets and is independent from other AWS accounts\. 
+ Amazon MWAA then binds an IP address from your private subnets to the interface endpoints\. This is designed to support the best practice of binding a single IP from each Availability Zone of the Amazon VPC\.

To learn more, see [Example use cases for an Amazon VPC and Apache Airflow access mode](networking-about.md#networking-about-network-usecase)\.

## Setup for private and public access modes<a name="access-network-choose"></a>

The following section describes the additional setup and configurations you'll need based on the Apache Airflow access mode you've chosen for your environment\.

### Setup for public network<a name="access-network-public"></a>

If you choose the **Public network** option for your Apache Airflow *Web server*, you can begin using the Apache Airflow UI after you create your environment\. 

You'll need to take the following steps to configure access for your users, and permission for your environment to use other AWS services\.

1. **Add permissions**\. Amazon MWAA needs permission to use other AWS services\. When you create an environment, Amazon MWAA creates a [service\-linked role](mwaa-slr.md) that allows it to use certain IAM actions for Amazon Elastic Container Registry \(Amazon ECR\), CloudWatch Logs, and Amazon EC2\.

   You can add permission to use additional actions for these services, or to use other AWS services by adding permissions to your execution role\. To learn more, see [Amazon MWAA execution role](mwaa-create-role.md)\.

1. **Create user policies**\. You may need to create multiple IAM policies for your users to configure access to your environment and Apache Airflow UI\. To learn more, see [Accessing an Amazon MWAA environment](access-policies.md)\.

### Setup for private network<a name="access-network-private"></a>

If you choose the **Private network** option for your Apache Airflow *Web server*, you'll need to configure access for your users, permission for your environment to use other AWS services, and create a mechanism to access the resources in your Amazon VPC from your computer\.

1. **Add permissions**\. Amazon MWAA needs permission to use other AWS services\. When you create an environment, Amazon MWAA creates a [service\-linked role](mwaa-slr.md) that allows it to use certain IAM actions for Amazon Elastic Container Registry \(Amazon ECR\), CloudWatch Logs, and Amazon EC2\.

   You can add permission to use additional actions for these services, or to use other AWS services by adding permissions to your execution role\. To learn more, see [Amazon MWAA execution role](mwaa-create-role.md)\.

1. **Create user policies**\. You may need to create multiple IAM policies for your users to configure access to your environment and Apache Airflow UI\. To learn more, see [Accessing an Amazon MWAA environment](access-policies.md)\.

1. **Enable network access**\. You'll need to create a mechanism in your Amazon VPC to connect to the VPC endpoint \(AWS PrivateLink\) for your Apache Airflow *Web server*\. For example, by creating a VPN tunnel from your computer using an AWS Client VPN\. 

## Accessing the VPC endpoint for your Apache Airflow Web server \(private network access\)<a name="configuring-access-vpce"></a>

If you've chosen the **Private network** option, you'll need to create a mechanism in your Amazon VPC to access the VPC endpoint \(AWS PrivateLink\) for your Apache Airflow *Web server*\. We recommend using the same Amazon VPC, VPC security group, and private subnets as your Amazon MWAA environment for these resources\.

To learn more, see [Managing access for VPC endpoints](https://docs.aws.amazon.com/mwaa/latest/userguide/vpc-vpe-access.html)\.