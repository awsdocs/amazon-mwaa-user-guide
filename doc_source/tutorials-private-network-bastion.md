# Tutorial: Configuring private network access using a Linux Bastion Host<a name="tutorials-private-network-bastion"></a>

This tutorial walks you through the steps to create an SSH tunnel from your computer to the to the Apache Airflow *Web server* for your Amazon Managed Workflows for Apache Airflow \(MWAA\) environment\. It assumes you've already created an Amazon MWAA environment\. Once set up, a Linux Bastion Host acts as a jump server allowing a secure connection from your computer to the resources in your VPC\. You'll then use a SOCKS proxy management add\-on to control the proxy settings in your browser to access your Apache Airflow UI\. 

**Topics**
+ [Private network](#private-network-lb-onconsole)
+ [Use cases](#private-network-lb-usecases)
+ [Before you begin](#private-network-lb-prereqs)
+ [Objectives](#private-network-lb-objectives)
+ [Step one: Create the bastion instance](#private-network-lb-create-bastion)
+ [Step two: Create the ssh tunnel](#private-network-lb-create-test)
+ [Step three: Configure the bastion security group as an inbound rule](#private-network-lb-create-sgsource)
+ [Step four: Copy the Apache Airflow URL](#private-network-lb-view-env)
+ [Step five: Configure proxy settings](#private-network-lb-browser-extension)
+ [Step six: Open the Apache Airflow UI](#private-network-lb-open)
+ [What's next?](#bastion-next-up)

## Private network<a name="private-network-lb-onconsole"></a>

This tutorial assumes you've chosen the **Private network** access mode for your Apache Airflow *Web server*\.

![\[This image shows the architecture for an Amazon MWAA environment with a private web server.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-private-web-server.png)

The private network access mode limits access of the Apache Airflow UI to users *within your Amazon VPC* that have been granted access to the [IAM policy for your environment](access-policies.md)\.

The following image shows where to find the **Private network** option on the Amazon MWAA console\. 

![\[This image shows where to find the Private network option on the Amazon MWAA console.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-private-network.png)

## Use cases<a name="private-network-lb-usecases"></a>

You can use this tutorial after you've created an Amazon MWAA environment\. You must use the same Amazon VPC, VPC security group\(s\), and public subnets as your environment\. 

## Before you begin<a name="private-network-lb-prereqs"></a>

1. Check for user permissions\. Be sure that your account in AWS Identity and Access Management \(IAM\) has sufficient permissions to create and manage VPC resources\. 

1. Use your Amazon MWAA VPC\. This tutorial assumes that you are associating the bastion host to an existing VPC\. The Amazon VPC must be in the same region as your Amazon MWAA environment and have two private subnets, as defined in [Create the VPC network](vpc-create.md)\.

1. Create an SSH key\. You need to create an Amazon EC2 SSH key \(**\.pem**\) in the same Region as your Amazon MWAA environment to connect to the virtual servers\. If you don't have an SSH key, see [Create or import a key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#prepare-key-pair) in the *Amazon EC2 User Guide for Linux Instances*\.

## Objectives<a name="private-network-lb-objectives"></a>

In this tutorial, you'll do the following:

1. Create a Linux Bastion Host instance using a [AWS CloudFormation template for an existing VPC](https://fwd.aws/vWMxm)\.

1. Authorize inbound traffic to the bastion instance's security group using an ingress rule on port `22`\.

1. Authorize inbound traffic from an Amazon MWAA environment's security group to the bastion instance's security group\.

1. Create an SSH tunnel to the bastion instance\.

1. Install and configure the FoxyProxy add\-on for the Firefox browser to view the Apache Airflow UI\.

## Step one: Create the bastion instance<a name="private-network-lb-create-bastion"></a>

The following section describes the steps to create the linux bastion instance using a [AWS CloudFormation template for an existing VPC](https://fwd.aws/vWMxm) on the AWS CloudFormation console\.

**To create the Linux Bastion Host**

1. Open the [Deploy Quick Start](https://fwd.aws/Jwzqv) page on the AWS CloudFormation console\.

1. Use the region selector in the navigation bar to choose the same AWS Region as your Amazon MWAA environment\.

1. Choose **Next**\.

1. Type a name in the **Stack name** text field, such as `mwaa-linux-bastion`\.

1. On the **Parameters**, **Network configuration** pane, choose the following options:

   1. Choose your Amazon MWAA environment's **VPC ID**\.

   1. Choose your Amazon MWAA environment's **Public subnet 1 ID**\.

   1. Choose your Amazon MWAA environment's **Public subnet 2 ID**\.

   1. Enter the narrowest possible address range \(for example, an internal CIDR range\) in **Allowed bastion external access CIDR**\.
**Note**  
The simplest way to identify a range is to use the same CIDR range as your public subnets\. For example, the public subnets in the AWS CloudFormation template on the [Create the VPC network](vpc-create.md) page are `10.192.10.0/24` and `10.192.11.0/24`\.

1. On the **Amazon EC2 configuration** pane, choose the following:

   1. Choose your SSH key in the dropdown list in **Key pair name**\.

   1. Enter a name in **Bastion Host Name**\.

   1. Choose **true** for **TCP forwarding**\.
**Warning**  
TCP forwarding must be set to **true** in this step\. Otherwise, you won't be able to create an SSH tunnel in the next step\.

1. Choose **Next**, **Next**\.

1. Select the acknowledgement, and then choose **Create stack**\.

To learn more about the architecture of your Linux Bastion Host, see [Linux Bastion Hosts on the AWS Cloud: Architecture](https://docs.aws.amazon.com/quickstart/latest/linux-bastion/architecture.html)\.

## Step two: Create the ssh tunnel<a name="private-network-lb-create-test"></a>

The following steps describe how to create the ssh tunnel to your linux bastion\. An SSH tunnel recieves the request from your local IP address to the linux bastion, which is why TCP forwarding for the linux bastion was set to `true` in previous steps\.

------
#### [ macOS/Linux ]

**To create a tunnel via command line**

1. Open the [Instances](https://console.aws.amazon.com/ec2/v2/home#/Instances:) page on the Amazon EC2 console\.

1. Choose an instance\.

1. Copy the address in **Public IPv4 DNS**\. For example, `ec2-4-82-142-1.compute-1.amazonaws.com`\.

1. In your command prompt, navigate to the directory where your SSH key is stored\.

1. Run the following command to connect to the bastion instance using ssh\. Substitute the sample value with your SSH key name in `mykeypair.pem`\.

   ```
   ssh -i mykeypair.pem -N -D 8157 ec2-user@YOUR_PUBLIC_IPV4_DNS
   ```

------
#### [ Windows \(PuTTY\) ]

**To create a tunnel using PuTTY**

1. Open the [Instances](https://console.aws.amazon.com/ec2/v2/home#/Instances:) page on the Amazon EC2 console\.

1. Choose an instance\.

1. Copy the address in **Public IPv4 DNS**\. For example, `ec2-4-82-142-1.compute-1.amazonaws.com`\.

1. Open [PuTTY](https://www.putty.org/), select **Session**\.

1. Enter the host name in **Host Name** as ec2\-user@*YOUR\_PUBLIC\_IPV4\_DNS* and the **port** as `22`\.

1. Expand the **SSH** tab, select **Auth**\. In **Private Key file for authentication**, choose your local "ppk" file\.

1. Under SSH, choose the **Tunnels** tab, and then select the *Dynamic* and *Auto* options\.

1. In **Source Port**, add the `8157` port \(or any other unused port\), and then leave the **Destination** port blank\. Choose **Add**\.

1. Choose the **Session** tab and enter a session name\. For example `SSH Tunnel`\. 

1. Choose **Save**, **Open**\.
**Note**  
You may need to enter a pass phrase for your public key\.

------

**Note**  
If you receive a `Permission denied (publickey)` error, we recommend using the [AWSSupport\-TroubleshootSSH](https://docs.aws.amazon.com/systems-manager/latest/userguide/automation-awssupport-troubleshootssh.html) tool, and choose **Run this Automation \(console\)** to troubleshoot your SSH setup\.

## Step three: Configure the bastion security group as an inbound rule<a name="private-network-lb-create-sgsource"></a>

Access to the servers and regular internet access from the servers is allowed with a special maintenance security group attached to those servers\. The following steps describe how to configure the bastion security group as an inbound source of traffic to an environment's VPC security group\.

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. On the **Networking** pane, choose **VPC security group**\.

1. Choose **Edit inbound rules**\.

1. Choose **Add rule**\.

1. Choose your VPC security group ID in the **Source** dropdown list\.

1. Leave the remaining options blank, or set to their default values\.

1. Choose **Save rules**\.

## Step four: Copy the Apache Airflow URL<a name="private-network-lb-view-env"></a>

The following steps describe how to open the Amazon MWAA console and copy the URL to the Apache Airflow UI\.

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose an environment\.

1. Copy the URL in **Airflow UI** for subsequent steps\.

## Step five: Configure proxy settings<a name="private-network-lb-browser-extension"></a>

If you use an SSH tunnel with dynamic port forwarding, you must use a SOCKS proxy management add\-on to control the proxy settings in your browser\. For example, you can use the `--proxy-server` feature of Chromium to kick off a browser session, or use the FoxyProxy extension in the Mozilla FireFox browser\. 

### Option one: Setup an SSH Tunnel using local port forwarding<a name="private-network-lb-browser-extension-portforwarding"></a>

If you do not wish to use a SOCKS proxy, you can set up an SSH tunnel using local port forwarding\. The following example command accesses the Amazon EC2 *ResourceManager* web interface by forwarding traffic on local port 8157\.

1. Open a new command prompt window\.

1. Type the following command to open an SSH tunnel\.

   ```
   ssh -i mykeypair.pem -N -L 8157:YOUR_VPC_ENDPOINT_ID-vpce.YOUR_REGION.airflow.amazonaws.com:443 ubuntu@YOUR_PUBLIC_IPV4_DNS.YOUR_REGION.compute.amazonaws.com
   ```

   `-L` signifies the use of local port forwarding which allows you to specify a local port used to forward data to the identified remote port on the node's local web server\.

1. Type `http://localhost:8157/` in your browser\.
**Note**  
You may need to use `https://localhost:8157/`\.

### Option two: Proxies via command line<a name="private-network-lb-browser-extension-foxyp"></a>

Most web browsers allow you to configure proxies via a command line or configuration parameter\. For example, with Chromium you can start the browser with the following command:

```
chromium --proxy-server="socks5://localhost:8157"
```

This starts a browser session which uses the ssh tunnel you created in previous steps to proxy its requests\. You can open your Private Amazon MWAA environment URL \(with *https://*\) as follows:

```
https://YOUR_VPC_ENDPOINT_ID-vpce.YOUR_REGION.airflow.amazonaws.com/home.
```

### Option three: Proxies using FoxyProxy for Mozilla Firefox<a name="private-network-lb-browser-extension-foxyp"></a>

 The following example demonstrates a FoxyProxy Standard \(version 7\.5\.1\) configuration for Mozilla Firefox\. FoxyProxy provides a set of proxy management tools\. It lets you use a proxy server for URLs that match patterns corresponding to domains used by the Apache Airflow UI\.

1. In FireFox, open the [FoxyProxy Standard](https://addons.mozilla.org/en-US/firefox/addon/foxyproxy-standard/) extension page\.

1. Choose **Add to Firefox**\.

1. Choose **Add**\.

1. Choose the FoxyProxy icon in your browser's toolbar, choose **Options**\.

1. Copy the following code and save locally as `mwaa-proxy.json`\. Substitute the sample value in *YOUR\_HOST\_NAME* with your **Apache Airflow URL**\.

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
           "pattern": "YOUR_HOST_NAME",
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

## Step six: Open the Apache Airflow UI<a name="private-network-lb-open"></a>

The following steps describe how to open your Apache Airflow UI\.

1. Open the [Environments page](https://console.aws.amazon.com/mwaa/home#/environments) on the Amazon MWAA console\.

1. Choose **Open Airflow UI**\.

## What's next?<a name="bastion-next-up"></a>
+ Learn how to run Airflow CLI commands on an SSH tunnel to a bastion host in [Apache Airflow CLI command reference](airflow-cli-command-reference.md)\.
+ Learn how to upload DAG code to your Amazon S3 bucket in [Adding or updating DAGs](configuring-dag-folder.md)\.