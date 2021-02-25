# Tutorial: Configuring private network access using a Linux Bastion Host<a name="tutorials-private-network-bastion"></a>

To reduce exposure of your Apache Airflow UI within a VPC, you need to create and use a Linux Bastion Host\.

A Linux Bastion Host is an instance that is provisioned with a public IP address and can be accessed via SSH\. Administrative tasks on the individual servers are performed using SSH, proxied through the bastion\. Amazon Managed Workflows for Apache Airflow \(MWAA\) connects \(SSH\) securely to the instance's private IP address via the bastion host to install or update any required software, e\.g\. for your Apache Airflow web server\. Once set up, the bastion acts as a jump server allowing secure connection to the Amazon EC2 instance used by an environment\. This tutorial walks you through the deployment of a Linux Bastion Host to securely access remote instances within a Amazon VPC for an Amazon MWAA environment\.

**Topics**
+ [Before you begin](#private-network-prereqs)
+ [Objectives](#private-network-objectives)
+ [Private network](#private-network-onconsole)
+ [Create the bastion instance](#private-network-create-bastion)
+ [Create the ssh tunnel](#private-network-create-test)
+ [Configure the bastion security group as an inbound rule](#private-network-create-sgsource)
+ [Copy the Apache Airflow URL](#private-network-view-env)
+ [Configure proxy settings](#private-network-browser-extension)
+ [Open the Apache Airflow UI](#private-network-open)
+ [What's next?](#create-vpc-next-up)

## Before you begin<a name="private-network-prereqs"></a>

1. Check for user permissions\. Be sure that your account in AWS Identity and Access Management \(IAM\) has sufficient permissions to create and manage VPC resources\. 

1. This tutorial assumes that you are adding the bastion host in an existing Amazon VPC, as defined in [Create the VPC network](vpc-create.md)\.

1. You need an Amazon EC2 SSH key in your target region to connect to the virtual servers\. If you don't have an SSH key, see [Create or import a key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#prepare-key-pair) in the *Amazon EC2 User Guide for Linux Instances*\.

## Objectives<a name="private-network-objectives"></a>

In this tutorial, you'll do the following:

1. Create a Linux Bastion Host instance\.

1. Authorize inbound traffic to the bastion instance's security group using an ingress rule on port `22`\.

1. Authorize inbound traffic from an Amazon MWAA environment's security group to the bastion instance's security group\.

1. Create an SSH tunnel to the bastion instance\.

1. Install and configure the FoxyProxy add\-on for the Firefox browser to view the Apache Airflow UI\.

## Private network<a name="private-network-onconsole"></a>

You can use the Amazon MWAA console to enable private access to your Apache Airflow UI\. The following image shows where to find the **Private network** option on the Amazon MWAA console\. The steps on this page assume you have chosen this option to access your Apache Airflow UI\.

![\[This image shows where to find the Private network option on the Amazon MWAA console.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-private-public.png)

## Create the bastion instance<a name="private-network-create-bastion"></a>

You can use the AWS CloudFormation console to launch the AWS CloudFormation stack needed for a Linux Bastion Host\. To learn more about each of the fields in this tutorial, see [Launch the Stack](https://docs.aws.amazon.com/quickstart/latest/linux-bastion/step2.html) in the *Linux Bastion Hosts on the AWS Cloud Quick Start Guide*\. 

**To create the Linux Bastion Host**

1. Open the [Deploy Quick Start](https://fwd.aws/Jwzqv) page on the AWS CloudFormation console\.

1. Choose **Next**\.

1. On the **Parameters**, **Network configuration** pane, choose the following options:

   1. Choose your Amazon MWAA environment's **VPC ID**\.

   1. Choose your Amazon MWAA environment's **Public subnet 1 ID**\.

   1. Choose your Amazon MWAA environment's **Public subnet 2 ID**\.

   1. Enter the narrowest possible address range \(for example, an internal CIDR range\) in **Allowed bastion external access CIDR**\.

   1. Leave the remaining options blank, or set to their default values\.

1. On the **Amazon EC2 configuration** pane, choose the following:

   1. Choose your Amazon MWAA environment's **Key pair name**\.

   1. Enter a name in **Bastion Host Name**\.

   1. Choose **true** for **TCP forwarding**\.

   1. Leave the remaining options blank, or set to their default values\.

1. Choose **Next**, **Next**\.

1. Select the acknowledgement, and then choose **Create stack**\.

## Create the ssh tunnel<a name="private-network-create-test"></a>

The following steps describe how to create the ssh tunnel to your bastion\.

1. Open the [Instances](https://console.aws.amazon.com/ec2/v2/home#/Instances:) page on the Amazon EC2 console\.

1. Choose an instance\.

1. Copy the address in **Public IPv4 DNS**\. For example, `ip-10-192-11-219.ec2.internal`\.

1. In your command prompt, navigate to the directory where your SSH key is stored\.

1. Run the following command to connect to the bastion instance using ssh\. Substitute the sample value with your SSH key name in `mykeypair.pem`\.

   ```
   ssh -i mykeypair.pem -N -D 8157 ec2-user@YOUR_PUBLIC_IPV4_DNS
   ```

**Note**  
If you receive a `Permission denied (publickey)` error, see the following Amazon support topic in [I'm receiving "Permission denied \(publickey\)"](http://aws.amazon.com/premiumsupport/knowledge-center/ec2-linux-fix-permission-denied-errors/)\.

## Configure the bastion security group as an inbound rule<a name="private-network-create-sgsource"></a>

Access to the servers and regular internet access from the servers is allowed with a special maintenance security group attached to those servers\. The following steps describe how to configure the bastion security group as an inbound source of traffic to an environment's VPC security group\.

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. On the **Networking** pane, choose **VPC security group**\.

1. Choose **Edit inbound rules**\.

1. Choose **Add rule**\.

1. Choose your VPC security group ID in the **Source** dropdown list\.

1. Leave the remaining options blank, or set to their default values\.

1. Choose **Save rules**\.

## Copy the Apache Airflow URL<a name="private-network-view-env"></a>

The following steps describe how to open the Amazon MWAA console and copy the URL to the Apache Airflow UI\.

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Copy the URL in **Airflow UI** for subsequent steps\.

## Configure proxy settings<a name="private-network-browser-extension"></a>

If you use an SSH tunnel with dynamic port forwarding, you must use a SOCKS proxy management add\-on to control the proxy settings in your browser\. For example, you can use the `--proxy-server` feature of Chromium to kick off a browser session, or use the FoxyProxy extension in the Mozilla FireFox browser\.

### Proxies via command line<a name="private-network-browser-extension-foxyp"></a>

Most web browsers allow you to configure proxies via a command line or configuration parameter\. For example, with Chromium you can start the browser with the following command:

```
chromium --proxy-server="socks5://localhost:8157"
```

This starts a browser session which uses the ssh tunnel you created in previous steps to proxy its requests\. You can open your Private Amazon MWAA environment URL \(with *https://*\) as follows:

```
https://{unique-id}-vpce.{region}.airflow.amazonaws.com/home.
```

### Proxies using FoxyProxy for Mozilla Firefox<a name="private-network-browser-extension-foxyp"></a>

 The following example demonstrates a FoxyProxy Standard \(version 7\.5\.1\) configuration for Mozilla Firefox\. FoxyProxy provides a set of proxy management tools\. It lets you use a proxy server for URLs that match patterns corresponding to domains used by the Apache Airflow UI\.

1. In FireFox, open the [FoxyProxy Standard](https://addons.mozilla.org/en-US/firefox/addon/foxyproxy-standard/) extension page\.

1. Choose **Add to Firefox**\.

1. Choose **Add**\.

1. Choose the FoxyProxy icon in your browser's toolbar, choose **Options**\.

1. Copy the following code and save locally as `mwaa-proxy.json`\. Substitute the sample value with your **Airflow UI** URL in the `pattern` field\.

   ```
   {
     "e0b7kh1606694837384": {
       "type": 3,
       "color": "#66cc66",
       "title": "airflow",
       "active": true,
       "address": "localhost",
       "port": 8157,
       "proxyDNS": false,
       "username": "",
       "password": "",
       "whitePatterns": [
         {
           "title": "airflow-ui",
           "pattern": "xxxxx-vpce.c5.us-east-1.airflow.amazonaws.com",
           "type": 1,
           "protocols": 1,
           "active": true
         }
       ],
       "blackPatterns": [],
       "pacURL": "",
       "index": -1
     },
     "k20d21508277536715": {
       "active": true,
       "title": "Default",
       "notes": "These are the settings that are used when no patterns match a URL.",
       "color": "#0055E5",
       "type": 5,
       "whitePatterns": [
         {
           "title": "all URLs",
           "active": true,
           "pattern": "*",
           "type": 1,
           "protocols": 1
         }
       ],
       "blackPatterns": [],
       "index": 9007199254740991
     },
     "logging": {
       "active": true,
       "maxSize": 500
     },
     "mode": "patterns",
     "browserVersion": "82.0.3",
     "foxyProxyVersion": "7.5.1",
     "foxyProxyEdition": "standard"
   }
   ```

1. On the **Import Settings from FoxyProxy 6\.0\+** pane, choose **Import Settings** and select the `mwaa-proxy.json` file\.

1. Choose **OK**\.

## Open the Apache Airflow UI<a name="private-network-open"></a>

The following steps describe how to open your Apache Airflow UI\.

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose **Open Airflow UI**\.

## What's next?<a name="create-vpc-next-up"></a>
+ Learn how to upload DAG code to your Amazon S3 bucket in [Adding or updating DAGs](configuring-dag-folder.md)\.