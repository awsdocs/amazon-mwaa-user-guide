# Managing access to VPC endpoints on Amazon MWAA<a name="vpc-vpe-access"></a>

A VPC endpoint \(AWS PrivateLink\) enables you to privately connect your VPC to services hosted on AWS without requiring an Internet gateway, a NAT device, VPN, or firewall proxies\. These endpoints are horizontally scalable and highly available virtual devices that allow communication between instances in your VPC and AWS services\. This page describes the VPC endpoints created by Amazon MWAA, and how to access the VPC endpoint for your Apache Airflow *Web server* if you've chosen the **Private network** access mode on Amazon Managed Workflows for Apache Airflow \(MWAA\)\.

**Contents**
+ [Pricing](#vpc-vpe-pricing)
+ [VPC endpoint overview](#vpc-vpe-about)
  + [Public network access mode](#vpc-vpe-about-public)
  + [Private network access mode](#vpc-vpe-about-private)
+ [Permission to use other AWS services](#vpc-vpe-permission)
+ [Viewing VPC endpoints](#vpc-vpe-view-all)
  + [Viewing VPC endpoints on the Amazon VPC console](#vpc-vpe-view-endpoints)
  + [Identifying the private IP addresses of your Apache Airflow Web server and its VPC endpoint](#vpc-vpe-hosts)
+ [Accessing the VPC endpoint for your Apache Airflow Web server \(private network access\)](#vpc-vpe-access-endpoints)
  + [Using an AWS Client VPN](#vpc-vpe-access-vpn)
  + [Using a Linux Bastion Host](#vpc-vpe-access-bastion)
  + [Using a Load Balancer \(advanced\)](#vpc-vpe-access-load-balancer)

## Pricing<a name="vpc-vpe-pricing"></a>
+ [AWS PrivateLink Pricing](http://aws.amazon.com/privatelink/pricing/)

## VPC endpoint overview<a name="vpc-vpe-about"></a>

When you create an Amazon MWAA environment, Amazon MWAA creates between one to two VPC endpoints for your environment\. These endpoints appear as Elastic Network Interfaces \(ENIs\) with private IPs in your Amazon VPC\. After these endpoints are created, any traffic destined to these IPs is privately or publicly routed to the corresponding AWS services used by your environment\. 

### Public network access mode<a name="vpc-vpe-about-public"></a>

If you chose the **Public network** access mode for your Apache Airflow *Web server*, network traffic is publicly routed *over the Internet*\.
+ Amazon MWAA creates a VPC interface endpoint for your Amazon Aurora PostgreSQL metadata database\. The endpoint is created in the Availability Zones mapped to your private subnets and is independent from other AWS accounts\. 
+ Amazon MWAA then binds an IP address from your private subnets to the interface endpoints\. This is designed to support the best practice of binding a single IP from each Availability Zone of the Amazon VPC\.

### Private network access mode<a name="vpc-vpe-about-private"></a>

 If you chose the **Private network** access mode for your Apache Airflow *Web server*, network traffic is privately routed *within your Amazon VPC*\. 
+ Amazon MWAA creates a VPC interface endpoint for your Apache Airflow *Web server*, and an interface endpoint for your Amazon Aurora PostgreSQL metadata database\. The endpoints are created in the Availability Zones mapped to your private subnets and is independent from other AWS accounts\. 
+ Amazon MWAA then binds an IP address from your private subnets to the interface endpoints\. This is designed to support the best practice of binding a single IP from each Availability Zone of the Amazon VPC\.

## Permission to use other AWS services<a name="vpc-vpe-permission"></a>

The interface endpoints use the execution role for your environment in AWS Identity and Access Management \(IAM\) to manage permission to AWS resources used by your environment\. As more AWS services are enabled for an environment, each service will require you to configure permission using your environment's execution role\. To add permissions, see [Amazon MWAA Execution role](mwaa-create-role.md)\.

If you've chosen the **Private network** access mode for your Apache Airflow *Web server*, you must also allow permission in the VPC endpoint policy for each endpoint\. To learn more, see [VPC endpoint policies \(private routing only\)](vpc-security.md#vpc-external-vpce-policies)\.

## Viewing VPC endpoints<a name="vpc-vpe-view-all"></a>

This section describes how to view the VPC endpoints created by Amazon MWAA, and how to identify the private IP addresses for your Apache Airflow VPC endpoint\.

### Viewing VPC endpoints on the Amazon VPC console<a name="vpc-vpe-view-endpoints"></a>

The following section shows the steps to view the VPC endpoint\(s\) created by Amazon MWAA, and any VPC endpoints you may have created if you're using *private routing* for your Amazon VPC\.

**To view the VPC endpoint\(s\)**

1. Open the [Endpoints page](https://console.aws.amazon.com/vpc/home#Endpoints:) on the Amazon VPC console\.

1. Use the AWS Region selector to select your region\.

1. You should see the VPC interface endpoint\(s\) created by Amazon MWAA, and any VPC endpoints you may have created if you're using *private routing* in your Amazon VPC\.

To learn more about the VPC service endpoints that are required for an Amazon VPC with *private routing*, see [Creating the required VPC service endpoints in an Amazon VPC with private routing](vpc-vpe-create-access.md)\.

### Identifying the private IP addresses of your Apache Airflow Web server and its VPC endpoint<a name="vpc-vpe-hosts"></a>

The following steps describe how to retrieve the host name of your Apache Airflow Web server and its VPC interface endpoint, and their private IP addresses\.

1. Use the following AWS Command Line Interface \(AWS CLI\) command to retrieve the host name for your Apache Airflow *Web server*\.

   ```
   aws mwaa get-environment --name YOUR_ENVIRONMENT_NAME --query 'Environment.WebserverUrl'
   ```

   You should see something similar to the following response:

   ```
   "99aa99aa-55aa-44a1-a91f-f4552cf4e2f5-vpce.c10.us-west-2.airflow.amazonaws.com"
   ```

1. Run a *dig* command on the host name returned in the response of the previous command\. For example:

   ```
   dig CNAME +short 99aa99aa-55aa-44a1-a91f-f4552cf4e2f5-vpce.c10.us-west-2.airflow.amazonaws.com
   ```

   You should see something similar to the following response:

   ```
   vpce-0699aa333a0a0a0-bf90xjtr.vpce-svc-00bb7c2ca2213bc37.us-west-2.vpce.amazonaws.com.
   ```

1. Use the following AWS Command Line Interface \(AWS CLI\) command to retrieve the VPC endpoint DNS name returned in the response of the previous command\. For example:

   ```
   aws ec2 describe-vpc-endpoints | grep vpce-0699aa333a0a0a0-bf90xjtr.vpce-svc-00bb7c2ca2213bc37.us-west-2.vpce.amazonaws.com.
   ```

   You should see something similar to the following response:

   ```
   "DnsName": "vpce-066777a0a0a0-bf90xjtr.vpce-svc-00bb7c2ca2213bc37.us-west-2.vpce.amazonaws.com",
   ```

1. Run either an *nslookup* or *dig* command on your Apache Airflow host name and its VPC endpoint DNS name to retrieve the IP addresses\. For example:

   ```
   dig +short YOUR_AIRFLOW_HOST_NAME YOUR_AIRFLOW_VPC_ENDPOINT_DNS
   ```

   You should see something similar to the following response:

   ```
   10.199.11.111
     10.999.11.33
   ```

## Accessing the VPC endpoint for your Apache Airflow Web server \(private network access\)<a name="vpc-vpe-access-endpoints"></a>

If you've chosen the **Private network** access mode for your Apache Airflow *Web server*, you'll need to create a mechanism to access the VPC interface endpoint for your Apache Airflow *Web server*\. You must use the same Amazon VPC, VPC security group, and private subnets as your Amazon MWAA environment for these resources\.

### Using an AWS Client VPN<a name="vpc-vpe-access-vpn"></a>

AWS Client VPN is a managed client\-based VPN service that enables you to securely access your AWS resources and resources in your on\-premises network\. It provides a secure TLS connection from any location using the OpenVPN client\. 

We recommend following the Amazon MWAA tutorial to configure a Client VPN: [Tutorial: Configuring private network access using an AWS Client VPN](tutorials-private-network-vpn-client.md)\.

### Using a Linux Bastion Host<a name="vpc-vpe-access-bastion"></a>

A bastion host is a server whose purpose is to provide access to a private network from an external network, such as over the Internet from your computer\. Linux instances are in a public subnet, and they are set up with a security group that allows SSH access from the security group attached to the underlying Amazon EC2 instance running the bastion host\.

We recommend following the Amazon MWAA tutorial to configure a Linux Bastion Host: [Tutorial: Configuring private network access using a Linux Bastion Host](tutorials-private-network-bastion.md)\.

### Using a Load Balancer \(advanced\)<a name="vpc-vpe-access-load-balancer"></a>

The following section shows the configurations you'll need to apply to an [Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/tutorial-application-load-balancer-cli.html) or a [Network Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/network-load-balancer-cli.html)\.

1. **Target groups**\. You'll need to use target groups that point to the private IP addresses for your Apache Airflow *Web server*, and its VPC interface endpoint\. We recommend specifying both private IP addresses as your registered targets, as using only one can reduce availability\. To identify the private IP addresses, see [Identifying the private IP addresses of your Apache Airflow Web server and its VPC endpoint](#vpc-vpe-hosts) on this page\.

1. **Status codes**\. We recommend using `200` and `302` status codes in your target group settings\. Otherwise, the targets may be flagged as unhealthy if the VPC endpoint for the Apache Airflow *Web server* responds with a `302 Redirect` error\.

1. **HTTPS Listener**\. You'll need to specify the target port for the Apache Airflow *Web server*\. For example:    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/mwaa/latest/userguide/vpc-vpe-access.html)

1. **ACM new domain**\. If you want to associate an SSL/TLS certificate in AWS Certificate Manager, you'll need to create a new domain for the HTTPS listener for your load balancer\. 

1. **ACM certificate region**\. If you want to associate an SSL/TLS certificate in AWS Certificate Manager, you'll need to upload to the same AWS Region as your environment\. For example:

   1.   
**Example region to upload certificate**  

     ```
     aws acm import-certificate --certificate fileb://Certificate.pem --certificate-chain fileb://CertificateChain.pem --private-key fileb://PrivateKey.pem --region us-west-2
     ```