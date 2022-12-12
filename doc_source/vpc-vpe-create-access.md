# Creating the required VPC service endpoints in an Amazon VPC with private routing<a name="vpc-vpe-create-access"></a>

An existing Amazon VPC network *without Internet access* needs additional VPC service endpoints \(AWS PrivateLink\) to use Apache Airflow on Amazon Managed Workflows for Apache Airflow \(MWAA\)\. This page describes the VPC endpoints required for the AWS services used by Amazon MWAA, the VPC endpoints required for Apache Airflow, and how to create and attach the VPC endpoints to an existing Amazon VPC with private routing\.

**Contents**
+ [Pricing](#vpc-vpe-create-pricing)
+ [Private network and private routing](#vpc-vpc-create-onconsole)
+ [\(Required\) VPC endpoints](#vpc-vpe-create-view-endpoints-examples)
+ [Attaching the required VPC endpoints](#vpc-vpe-create-view-endpoints-attach-all)
  + [VPC endpoints required for AWS services](#vpc-vpe-create-view-endpoints-attach-services)
  + [VPC endpoints required for Apache Airflow](#vpc-vpe-create-view-endpoints-attach-aa)
+ [\(Optional\) Enable private IP addresses for your Amazon S3 VPC interface endpoint](#vpc-vpe-create-view-endpoints-s3-exception)
  + [Using Route 53](#vpc-vpe-create-view-endpoints-s3-exception-route53)
  + [VPCs with custom DNS](#vpc-vpe-create-view-endpoints-s3-exception-customdns)

## Pricing<a name="vpc-vpe-create-pricing"></a>
+ [AWS PrivateLink Pricing](http://aws.amazon.com/privatelink/pricing/)

## Private network and private routing<a name="vpc-vpc-create-onconsole"></a>

![\[This image shows the architecture for an Amazon MWAA environment with a private web server.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-private-web-server.png)

The private network access mode limits access of the Apache Airflow UI to users *within your Amazon VPC* that have been granted access to the [IAM policy for your environment](access-policies.md)\.

The following image shows where to find the **Private network** option on the Amazon MWAA console\. 

![\[This image shows where to find the Private network option on the Amazon MWAA console.\]](http://docs.aws.amazon.com/mwaa/latest/userguide/images/mwaa-console-private-network.png)
+ **Private routing**\. An [Amazon VPC *without Internet access*](networking-about.md) limits network traffic within the VPC\. This page assumes your Amazon VPC does not have Internet access and requires VPC endpoints for each AWS service used by your environment, and VPC endpoints for Apache Airflow in the same AWS Region and Amazon VPC as your Amazon MWAA environment\.

## \(Required\) VPC endpoints<a name="vpc-vpe-create-view-endpoints-examples"></a>

The following section shows the required VPC endpoints needed for an Amazon VPC *without Internet access*\. It lists the VPC endpoints for each AWS service used by Amazon MWAA, including the VPC endpoints needed for Apache Airflow\. 

```
com.amazonaws.YOUR_REGION.s3
com.amazonaws.YOUR_REGION.monitoring
com.amazonaws.YOUR_REGION.ecr.dkr
com.amazonaws.YOUR_REGION.ecr.api
com.amazonaws.YOUR_REGION.logs
com.amazonaws.YOUR_REGION.sqs
com.amazonaws.YOUR_REGION.kms
com.amazonaws.YOUR_REGION.airflow.api
com.amazonaws.YOUR_REGION.airflow.env
com.amazonaws.YOUR_REGION.airflow.ops
```

## Attaching the required VPC endpoints<a name="vpc-vpe-create-view-endpoints-attach-all"></a>

This section describes the steps to attach the required VPC endpoints for an Amazon VPC with private routing\.

### VPC endpoints required for AWS services<a name="vpc-vpe-create-view-endpoints-attach-services"></a>

The following section shows the steps to attach the VPC endpoints for the AWS services used by an environment to an existing Amazon VPC\. 

**To attach VPC endpoints to your private subnets**

1. Open the [Endpoints page](https://console.aws.amazon.com/vpc/home#Endpoints:sort=vpcEndpointType) on the Amazon VPC console\.

1. Use the AWS Region selector to select your region\.

1. Create the endpoint for Amazon S3:

   1. Choose **Create Endpoint**\.

   1. In the *Filter by attributes or search by keyword* text field, type: **\.s3**, then press *Enter* on your keyboard\.

   1. We recommend choosing the service endpoint listed for the **Gateway** type\.

      For example, **com\.amazonaws\.us\-west\-2\.s3 amazon Gateway**

   1. Choose your environment's Amazon VPC in **VPC**\.

   1. Ensure that your two private subnets in different Availability Zones are selected, and that that private DNS is enabled by selecting **Enable DNS name**\.

   1. Choose your environment's Amazon VPC security group\(s\)\.

   1. Choose **Full Access** in **Policy**\.

   1. Choose **Create endpoint**\.

1. Create the first endpoint for Amazon ECR:

   1. Choose **Create Endpoint**\.

   1. In the *Filter by attributes or search by keyword* text field, type: **\.ecr\.dkr**, then press *Enter* on your keyboard\.

   1. Select the service endpoint\.

   1. Choose your environment's Amazon VPC in **VPC**\.

   1. Ensure that your two private subnets in different Availability Zones are selected, and that **Enable DNS name** is enabled\.

   1. Choose your environment's Amazon VPC security group\(s\)\.

   1. Choose **Full Access** in **Policy**\.

   1. Choose **Create endpoint**\.

1. Create the second endpoint for Amazon ECR:

   1. Choose **Create Endpoint**\.

   1. In the *Filter by attributes or search by keyword* text field, type: **\.ecr\.api**, then press *Enter* on your keyboard\.

   1. Select the service endpoint\.

   1. Choose your environment's Amazon VPC in **VPC**\.

   1. Ensure that your two private subnets in different Availability Zones are selected, and that **Enable DNS name** is enabled\.

   1. Choose your environment's Amazon VPC security group\(s\)\.

   1. Choose **Full Access** in **Policy**\.

   1. Choose **Create endpoint**\.

1. Create the endpoint for CloudWatch Logs:

   1. Choose **Create Endpoint**\.

   1. In the *Filter by attributes or search by keyword* text field, type: **\.logs**, then press *Enter* on your keyboard\.

   1. Select the service endpoint\.

   1. Choose your environment's Amazon VPC in **VPC**\.

   1. Ensure that your two private subnets in different Availability Zones are selected, and that **Enable DNS name** is enabled\.

   1. Choose your environment's Amazon VPC security group\(s\)\.

   1. Choose **Full Access** in **Policy**\.

   1. Choose **Create endpoint**\.

1. Create the endpoint for CloudWatch Monitoring:

   1. Choose **Create Endpoint**\.

   1. In the *Filter by attributes or search by keyword* text field, type: **\.monitoring**, then press *Enter* on your keyboard\.

   1. Select the service endpoint\.

   1. Choose your environment's Amazon VPC in **VPC**\.

   1. Ensure that your two private subnets in different Availability Zones are selected, and that **Enable DNS name** is enabled\.

   1. Choose your environment's Amazon VPC security group\(s\)\.

   1. Choose **Full Access** in **Policy**\.

   1. Choose **Create endpoint**\.

1. Create the endpoint for Amazon SQS:

   1. Choose **Create Endpoint**\.

   1. In the *Filter by attributes or search by keyword* text field, type: **\.sqs**, then press *Enter* on your keyboard\.

   1. Select the service endpoint\.

   1. Choose your environment's Amazon VPC in **VPC**\.

   1. Ensure that your two private subnets in different Availability Zones are selected, and that **Enable DNS name** is enabled\.

   1. Choose your environment's Amazon VPC security group\(s\)\.

   1. Choose **Full Access** in **Policy**\.

   1. Choose **Create endpoint**\.

1. Create the endpoint for AWS KMS:

   1. Choose **Create Endpoint**\.

   1. In the *Filter by attributes or search by keyword* text field, type: **\.kms**, then press *Enter* on your keyboard\.

   1. Select the service endpoint\.

   1. Choose your environment's Amazon VPC in **VPC**\.

   1. Ensure that your two private subnets in different Availability Zones are selected, and that **Enable DNS name** is enabled\.

   1. Choose your environment's Amazon VPC security group\(s\)\.

   1. Choose **Full Access** in **Policy**\.

   1. Choose **Create endpoint**\.

### VPC endpoints required for Apache Airflow<a name="vpc-vpe-create-view-endpoints-attach-aa"></a>

The following section shows the steps to attach the VPC endpoints for Apache Airflow to an existing Amazon VPC\. 

**To attach VPC endpoints to your private subnets**

1. Open the [Endpoints page](https://console.aws.amazon.com/vpc/home#Endpoints:sort=vpcEndpointType) on the Amazon VPC console\.

1. Use the AWS Region selector to select your region\.

1. Create the endpoint for the Apache Airflow API:

   1. Choose **Create Endpoint**\.

   1. In the *Filter by attributes or search by keyword* text field, type: **\.airflow\.api**, then press *Enter* on your keyboard\.

   1. Select the service endpoint\.

   1. Choose your environment's Amazon VPC in **VPC**\.

   1. Ensure that your two private subnets in different Availability Zones are selected, and that **Enable DNS name** is enabled\.

   1. Choose your environment's Amazon VPC security group\(s\)\.

   1. Choose **Full Access** in **Policy**\.

   1. Choose **Create endpoint**\.

1. Create the first endpoint for the Apache Airflow environment:

   1. Choose **Create Endpoint**\.

   1. In the *Filter by attributes or search by keyword* text field, type: **\.airflow\.env**, then press *Enter* on your keyboard\.

   1. Select the service endpoint\.

   1. Choose your environment's Amazon VPC in **VPC**\.

   1. Ensure that your two private subnets in different Availability Zones are selected, and that **Enable DNS name** is enabled\.

   1. Choose your environment's Amazon VPC security group\(s\)\.

   1. Choose **Full Access** in **Policy**\.

   1. Choose **Create endpoint**\.

1. Create the second endpoint for Apache Airflow operations:

   1. Choose **Create Endpoint**\.

   1. In the *Filter by attributes or search by keyword* text field, type: **\.airflow\.ops**, then press *Enter* on your keyboard\.

   1. Select the service endpoint\.

   1. Choose your environment's Amazon VPC in **VPC**\.

   1. Ensure that your two private subnets in different Availability Zones are selected, and that **Enable DNS name** is enabled\.

   1. Choose your environment's Amazon VPC security group\(s\)\.

   1. Choose **Full Access** in **Policy**\.

   1. Choose **Create endpoint**\.

## \(Optional\) Enable private IP addresses for your Amazon S3 VPC interface endpoint<a name="vpc-vpe-create-view-endpoints-s3-exception"></a>

Amazon S3 **Interface** endpoints don't support private DNS\. The S3 endpoint requests still resolves to a *public* IP address\. To resolve the S3 address to a *private* IP address, you need to add a [private hosted zone in Route 53](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/hosted-zones-private.html) for the S3 regional endpoint\. 

### Using Route 53<a name="vpc-vpe-create-view-endpoints-s3-exception-route53"></a>

This section describes the steps to enable private IP addresses for an S3 **Interface** endpoint using Route 53\.

1. Create a Private Hosted Zone for your Amazon S3 VPC interface endpoint \(such as, s3\.eu\-west\-1\.amazonaws\.com\) and associate it with your Amazon VPC\.

1. Create an ALIAS A record for your Amazon S3 VPC interface endpoint \(such as, s3\.eu\-west\-1\.amazonaws\.com\) that resolves to your VPC Interface Endpoint DNS name\.

1. Create an ALIAS A wildcard record for your Amazon S3 interface endpoint \(such as, \*\.s3\.eu\-west\-1\.amazonaws\.com\) that resolves to the VPC Interface Endpoint DNS name\.

### VPCs with custom DNS<a name="vpc-vpe-create-view-endpoints-s3-exception-customdns"></a>

If your Amazon VPC uses custom DNS routing, you need to make the changes in your DNS resolver \(not Route 53, typically an EC2 instance running a DNS server\) by creating a CNAME record\. For example:

```
Name: s3.us-west-2.amazonaws.com
Type: CNAME
Value:  *.vpce-0f67d23e37648915c-e2q2e2j3.s3.eu-west-1.vpce.amazonaws.com
```