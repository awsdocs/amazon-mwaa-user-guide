# About networking on Amazon MWAA<a name="networking-about"></a>

An Amazon VPC is a virtual network that is linked to your AWS account\. It gives you cloud security and the ability to scale dynamically by providing fine\-grained control over your virtual infrastructure and network traffic segmentation\. This page describes the Amazon VPC infrastructure with *public routing* or *private routing* that's needed to support an Amazon Managed Workflows for Apache Airflow \(MWAA\) environment\.

**Contents**
+ [Terms](#networking-about-defs)
+ [What's supported](#networking-about-supported)
+ [VPC infrastructure overview](#networking-about-overview)
  + [Public routing over the Internet](#networking-about-overview-public)
  + [Private routing without Internet access](#networking-about-overview-private)
+ [Example use cases for an Amazon VPC and Apache Airflow access mode](#networking-about-network-usecase)
  + [Internet access is allowed \- new Amazon VPC network](#networking-about-network-usecase-internet)
  + [Internet access is not allowed \- new Amazon VPC network](#networking-about-network-usecase-nointernet)
  + [Internet access is not allowed \- existing Amazon VPC network](#networking-about-network-usecase-nointernet-existing-vpc)

## Terms<a name="networking-about-defs"></a>

**Public routing**  
An Amazon VPC network that has access to the Internet\. 

**Private routing**  
An Amazon VPC network **without** access to the Internet\.

## What's supported<a name="networking-about-supported"></a>

The following table describes the types of Amazon VPCs Amazon MWAA supports\.


| Amazon VPC types | Supported | 
| --- | --- | 
|  An Amazon VPC owned by the account that is attempting to create the environment\.  |  Yes  | 
|  A shared Amazon VPC where multiple AWS accounts create their AWS resources\.  |  No  | 

## VPC infrastructure overview<a name="networking-about-overview"></a>

When you create an Amazon MWAA environment, Amazon MWAA creates between one to two VPC endpoints for your environment based on the Apache Airflow access mode you chose for your environment\. These endpoints appear as Elastic Network Interfaces \(ENIs\) with private IPs in your Amazon VPC\. After these endpoints are created, any traffic destined to these IPs is privately or publicly routed to the corresponding AWS services used by your environment\.

The following section describes the Amazon VPC infrastructure required to route traffic publicly *over the Internet*, or privately *within your Amazon VPC*\.

### Public routing over the Internet<a name="networking-about-overview-public"></a>

This section describes the Amazon VPC infrastructure of an environment with public routing\. You'll need the following VPC infrastructure:
+ **One VPC security group**\. A VPC security group acts as a virtual firewall to control ingress \(inbound\) and egress \(outbound\) network traffic on an instance\.
  + Up to 5 security groups can be specified\.
  + The security group must specify a self\-referencing inbound rule to itself\.
  + The security group must specify an outbound rule for all traffic \(`0.0.0.0/0`\)\.
  + The security group must allow all traffic in the self\-referencing rule\. For example, [\(Recommended\) Example all access self\-referencing security group ](vpc-security.md#vpc-security-sg-example)\. 
  + The security group can *optionally* restrict traffic further by specifying the port range for HTTPS port range `443` and a TCP port range `5432`\. For example, [\(Optional\) Example security group that restricts inbound access to port 5432](vpc-security.md#vpc-security-sg-example-port5432) and [\(Optional\) Example security group that restricts inbound access to port 443](vpc-security.md#vpc-security-sg-example-port443)\.
+ **Two public subnets**\. A public subnet is a subnet that's associated with a route table that has a route to an Internet gateway\.
  + Two public subnets are required\. This allows Amazon MWAA to build a new container image for your environment in your other availability zone, if one container fails\. 
  + The subnets must be in different Availability Zones\. For example, `us-east-1a`, `us-east-1b`\.
  + The subnets must route to a NAT gateway \(or NAT instance\) with an Elastic IP Address \(EIP\)\.
  + The subnets must have a route table that directs internet\-bound traffic to an Internet gateway\.
+ **Two private subnets**\. A private subnet is a subnet that's **not** associated with a route table that has a route to an Internet gateway\.
  + Two private subnets are required\. This allows Amazon MWAA to build a new container image for your environment in your other availability zone, if one container fails\. 
  + The subnets must be in different Availability Zones\. For example, `us-east-1a`, `us-east-1b`\. 
  + The subnets *must* have a route table to a NAT device \(gateway or instance\)\.
  + The subnets **must not** route to an Internet gateway\. 
+ **A network access control list \(ACL\)**\. An NACL manages \(by allow or deny rules\) inbound and outbound traffic at the subnet level\.
  + The NACL must have an inbound rule that allows all traffic \(`0.0.0.0/0`\)\.
  + The NACL must have an outbound rule that denies all traffic \(`0.0.0.0/0`\)\.
  + For example, [\(Recommended\) Example ACLs](vpc-security.md#vpc-security-acl-example)\.
+ **Two NAT gateways \(or NAT instances\)**\. A NAT device forwards traffic from the instances in the private subnet to the Internet or other AWS services, and then routes the response back to the instances\.
  + The NAT device must be attached to a public subnet\. \(One NAT device per public subnet\.\)
  + The NAT device must have an Elastic IPv4 Address \(EIP\) attached to each public subnet\.
+ **An Internet gateway**\. An Internet gateway connects an Amazon VPC to the Internet and other AWS services\.
  + An Internet gateway must be attached to the Amazon VPC\.

### Private routing without Internet access<a name="networking-about-overview-private"></a>

This section describes the Amazon VPC infrastructure of an environment with *private routing*\. You'll need the following VPC infrastructure:
+ **One VPC security group**\. A VPC security group acts as a virtual firewall to control ingress \(inbound\) and egress \(outbound\) network traffic on an instance\.
  + Up to 5 security groups can be specified\.
  + The security group must specify a self\-referencing inbound rule to itself\.
  + The security group must specify an outbound rule for all traffic \(`0.0.0.0/0`\)\.
  + The security group must allow all traffic in the self\-referencing rule\. For example, [\(Recommended\) Example all access self\-referencing security group ](vpc-security.md#vpc-security-sg-example)\. 
  + The security group can *optionally* restrict traffic further by specifying the port range for HTTPS port range `443` and a TCP port range `5432`\. For example, [\(Optional\) Example security group that restricts inbound access to port 5432](vpc-security.md#vpc-security-sg-example-port5432) and [\(Optional\) Example security group that restricts inbound access to port 443](vpc-security.md#vpc-security-sg-example-port443)\.
+ **Two private subnets**\. A private subnet is a subnet that's **not** associated with a route table that has a route to an Internet gateway\.
  + Two private subnets are required\. This allows Amazon MWAA to build a new container image for your environment in your other availability zone, if one container fails\. 
  + The subnets must be in different Availability Zones\. For example, `us-east-1a`, `us-east-1b`\.
  + The subnets must have a route table to your VPC endpoints\. 
  + The subnets **must not** have a route table to a NAT device \(gateway or instance\), **nor** an Internet gateway\.
+ **A network access control list \(ACL\)**\. An NACL manages \(by allow or deny rules\) inbound and outbound traffic at the subnet level\.
  + The NACL must have an inbound rule that allows all traffic \(`0.0.0.0/0`\)\.
  + The NACL must have an outbound rule that denies all traffic \(`0.0.0.0/0`\)\.
  + For example, [\(Recommended\) Example ACLs](vpc-security.md#vpc-security-acl-example)\.
+ **A local route table**\. A local route table is a default route for communication within the VPC\.
  + The local route table must be associated to your private subnets\.
  + The local route table must enable instances in your VPC to communicate with your own network\. For example, if you're using an AWS Client VPN to access the VPC interface endpoint for your Apache Airflow *Web server*, the route table must route to the VPC endpoint\.
+ **VPC endpoints** for each AWS service used by your environment, and Apache Airflow VPC endpoints in the same AWS Region and Amazon VPC as your Amazon MWAA environment\.
  + A VPC endpoint for each AWS service used by the environment and VPC endpoints for Apache Airflow\. For example, [\(Required\) VPC endpoints](vpc-vpe-create-access.md#vpc-vpe-create-view-endpoints-examples)\.
  + The VPC endpoints must have private DNS enabled\.
  + The VPC endpoints must be associated to your environment's two private subnets\.
  + The VPC endpoints must be associated to your environment's security group\.
  + The VPC endpoint policy for each endpoint should be configured to allow access to AWS services used by the environment\. For example, [\(Recommended\) Example VPC endpoint policy to allow all access](vpc-security.md#vpc-external-vpce-policies-all)\.
  + A VPC endpoint policy for Amazon S3 should be configured to allow bucket access\. For example, [\(Recommended\) Example Amazon S3 gateway endpoint policy to allow bucket access](vpc-security.md#vpc-external-vpce-policies-s3)\.

## Example use cases for an Amazon VPC and Apache Airflow access mode<a name="networking-about-network-usecase"></a>

This section descibes the different use cases for network access in your Amazon VPC and the Apache Airflow *Web server* access mode you should choose on the Amazon MWAA console\.

### Internet access is allowed \- new Amazon VPC network<a name="networking-about-network-usecase-internet"></a>

If Internet access in your VPC is allowed by your organization, *and* you would like users to access your Apache Airflow *Web server* over the Internet:

1. Create an Amazon VPC network *with Internet access*\. 

1. Create an environment with the **Public network** access mode for your Apache Airflow *Web server*\.

1. **What we recommend**: We recommend using the AWS CloudFormation quick\-start template that creates the Amazon VPC infrastructure, an Amazon S3 bucket, and an Amazon MWAA environment at the same time\. To learn more, see [Quick start tutorial for Amazon Managed Workflows for Apache Airflow \(MWAA\)](quick-start.md)\.

If Internet access in your VPC is allowed by your organization, *and* you would like to limit Apache Airflow *Web server* access to users within your VPC:

1. Create an Amazon VPC network *with Internet access*\.

1. Create a mechanism to access the VPC interface endpoint for your Apache Airflow *Web server* from your computer\.

1. Create an environment with the **Private network** access mode for your Apache Airflow *Web server*\. 

1. **What we recommend**:

   1. We recommend using the Amazon MWAA console in [Option one: Creating the VPC network on the Amazon MWAA console](vpc-create.md#vpc-create-mwaa-console), or the AWS CloudFormation template in [Option two: Creating an Amazon VPC network *with* Internet access](vpc-create.md#vpc-create-template-private-or-public)\.

   1. We recommend configuring access using an AWS Client VPN to your Apache Airflow *Web server* in [Tutorial: Configuring private network access using an AWS Client VPN](tutorials-private-network-vpn-client.md)\.

### Internet access is not allowed \- new Amazon VPC network<a name="networking-about-network-usecase-nointernet"></a>

If Internet access in your VPC is **not allowed** by your organization: 

1. Create an Amazon VPC network *without Internet access*\.

1. Create a mechanism to access the VPC interface endpoint for your Apache Airflow *Web server* from your computer\.

1. Create VPC endpoints for each AWS service used by your environment\.

1. Create an environment with the **Private network** access mode for your Apache Airflow *Web server*\. 

1. **What we recommend**:

   1. We recommend using the AWS CloudFormation template to create an Amazon VPC without Internet access and the VPC endpoints for each AWS service used by Amazon MWAA in [Option three: Creating an Amazon VPC network *without* Internet access](vpc-create.md#vpc-create-template-private-only)\.

   1. We recommend configuring access using an AWS Client VPN to your Apache Airflow *Web server* in [Tutorial: Configuring private network access using an AWS Client VPN](tutorials-private-network-vpn-client.md)\.

### Internet access is not allowed \- existing Amazon VPC network<a name="networking-about-network-usecase-nointernet-existing-vpc"></a>

If Internet access in your VPC is **not allowed** by your organization, *and* you already have the required Amazon VPC network *without Internet access*:

1. Create VPC endpoints for each AWS service used by your environment\.

1. Create VPC endpoints for Apache Airflow\.

1. Create a mechanism to access the VPC interface endpoint for your Apache Airflow *Web server* from your computer\.

1. Create an environment with the **Private network** access mode for your Apache Airflow *Web server*\. 

1. **What we recommend**:

   1. We recommend creating and attaching the VPC endpoints needed for each AWS service used by Amazon MWAA, and the VPC endpoints needed for Apache Airflow in [Creating the required VPC service endpoints in an Amazon VPC with private routing](vpc-vpe-create-access.md)\.

   1. We recommend configuring access using an AWS Client VPN to your Apache Airflow *Web server* in [Tutorial: Configuring private network access using an AWS Client VPN](tutorials-private-network-vpn-client.md)\.