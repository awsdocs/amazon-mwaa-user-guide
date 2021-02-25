# Amazon MWAA network access<a name="configuring-networking"></a>

The Amazon Managed Workflows for Apache Airflow \(MWAA\) console contains built\-in options to configure private or public network access to the Apache Airflow UI\. This guide describes the private and public network options, and the additional resources you'll need to securely access remote instances should you choose the private network option\.

**Topics**
+ [Private or public network](#configuring-networking-onconsole)
+ [How it works](#configuring-networking-how)
+ [Using a private or public network](#access-network-choose)
+ [Configuring private network access](#configuring-tips)

## Private or public network<a name="configuring-networking-onconsole"></a>

The following image shows where to find the **Public network** and **Private network** options on the Amazon MWAA console\.

![\[This image shows where to find the Private network option on the Amazon MWAA console.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-private-public.png)

## How it works<a name="configuring-networking-how"></a>

Amazon MWAA provides private and public networking options for your Apache Airflow web server\.
+ A public network allows the Apache Airflow UI to be accessed over the Internet by users granted access to your IAM policy\. When you choose this option, Amazon MWAA attaches an [Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html) with an HTTPS endpoint for your web server\. We recommend using the **Public network** option when VPC transit routing is not required, as this method doesn't require any additional setup to use an environment\.
+ A private network limits access to the Apache Airflow UI to users within your VPC\. In addition, you must grant users access to your IAM policy\. When you choose this option, Amazon MWAA attaches a [VPC endpoint](https://docs.aws.amazon.com/vpc/latest/userguide/endpoint-services-overview.html) to your web server\. Enabling access to this endpoint requires additional configuration, such as a proxy or [Linux Bastion Host on the AWS Cloud](https://docs.aws.amazon.com/quickstart/latest/linux-bastion/welcome.html)\. We recommend using the **Private network** option if you already have [VPC transit routing](https://docs.aws.amazon.com/vpc/latest/tgw/how-transit-gateways-work.html) configured for your users in IAM\.

## Using a private or public network<a name="access-network-choose"></a>

The following section describes the additional resources you may need to create before you can use your environment\.

### Using a public network<a name="access-network-public"></a>

If you choose the **Public network** option on the Amazon MWAA console, you don't need to create additional AWS resources to access your Apache Airflow UI\. To grant users access to your Apache Airflow UI, see [Accessing an Amazon MWAA environment](access-policies.md)\.

### Using a private network<a name="access-network-private"></a>

When you create an Amazon MWAA environment with the **Private network** option, Amazon MWAA creates and attaches an *interface endpoint* to your Apache Airflow web server via [VPC endpoint services \(AWS PrivateLink\)](https://docs.aws.amazon.com/vpc/latest/userguide/endpoint-service-overview.html)\. If you choose this option, you'll need to configure access and create additional AWS resources to resolve the domain for your environment and access your Apache Airflow UI\. Here are a few options:
+ **Option 1: Linux Bastion Host** – to forward requests to a domain name through an Amazon EC2 instance in the same VPC\. To learn more, see [Tutorial: Configuring private network access using a Linux Bastion Host](tutorials-private-network-bastion.md)\.
+ **Option 2: Load Balancer** – to forward requests to the IP addresses assigned to a domain name\.
  + An Application Load Balancer in the same Amazon VPC as your environment to forward requests to the IP addresses assigned to a domain name\. To learn more, see [Tutorial: Create an Application Load Balancer using the AWS CLI](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/tutorial-application-load-balancer-cli.html)\.
  + A Network Load Balancer in the same Amazon VPC as your environment to forward requests to the IP addresses assigned to a domain name\. To learn more, see [Tutorial: Create a Network Load Balancer using the AWS CLI](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/network-load-balancer-cli.html)\.
+ **Option 3: VPN client or Direct Connect** – to resolve the domain name for an environment \(provided that the environment's security group allows the local IP range\)\.
  + [AWS Client VPN](https://docs.aws.amazon.com/vpn/latest/clientvpn-admin/what-is.html) to securely access your AWS resources and any resources in your on\-premises network\. To learn more, see [Getting started with Client VPN](https://docs.aws.amazon.com/vpn/latest/clientvpn-admin/cvpn-getting-started.html)\.
  + [AWS Direct Connect](https://docs.aws.amazon.com/directconnect/latest/UserGuide/Welcome.html) to link your internal network to an AWS Direct Connect location over a standard Ethernet fiber\-optic cable\. To learn more, see [Create a connection using the AWS CLI](https://docs.aws.amazon.com/directconnect/latest/UserGuide/using-cli.html)\.

## Configuring private network access<a name="configuring-tips"></a>

The following section describes Amazon MWAA\-specific configurations you can use to configure private network access for an environment\.

### Using a Linux Bastion Host<a name="configuring-private-bastion"></a>

We recommend using the following tutorial to configure a Linux Bastion Host for your Amazon MWAA environment: [Tutorial: Configuring private network access using a Linux Bastion Host](tutorials-private-network-bastion.md)\.

### Using a Network Load Balancer or Application Load Balancer<a name="configuring-private-load-balancer"></a>

A public Network Load Balancer, or an Application Load Balancer must use target groups that point to the private IP addresses for your Apache Airflow host name\.

1. Run a *nslookup* or *dig* command on the domain to get the IP addresses\. For example:

   ```
   dig +short 59438366-1087-4346-9f49-a4ccd16267b7-vpce.c9.us-east-1.airflow.amazonaws.com
   vpce-010587637a219be56-85lu8ws1.vpce-svc-0fbf48eec80c6f78f.us-east-1.vpce.amazonaws.com.
   10.192.21.23
   10.192.20.201
   ```

1. If you want to associate an [ACM SSL/TLS certificate](https://docs.aws.amazon.com/premiumsupport/knowledge-center/associate-acm-certificate-alb-nlb/), you'll need to create a new domain for the HTTPS listener for your Network Load Balancer or Application Load Balancer\. To learn more, see [Create an HTTPS listener for your Application Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-https-listener.html), or [Create a listener for your Network Load Balancer](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/create-listener.html)\.