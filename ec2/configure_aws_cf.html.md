---
title: Configuring AWS for Cloud Foundry
---

This topic describes how to configure Amazon Web Services (AWS) for Cloud
Foundry.

## <a id="create-cf-manifest"></a>Step 1: Create a Deployment Manifest

The Cloud Foundry deployment manifest is a YAML file that defines the deployment
components and properties.
The [cloudfoundry/cf-release](https://github.com/cloudfoundry/cf-release) repo
contains template deployment manifests that you can edit with your deployment
information.
The `minimal-aws.yml` file contains the minimum information necessary to deploy
Cloud Foundry to AWS.

Create a manifest for your deployment as follows:

1. Create a deployment directory to store your manifest.

    <pre class='terminal'>
    $ mkdir ~/cf-deployment
    </pre>

1. Change into the cf-deployment directory and clone the cloudfoundry/cf-release
repo.

    <pre class='terminal'>
    $ git clone https://github.com/cloudfoundry/cf-release.git
    </pre>

1. Navigate to the example_manifests subdirectory to retrieve the
`minimal-aws.yml` template.
Copy and paste the template into a text editor and save the edited manifest to
your deployment directory.

    In the template, you must replace the following properties:
    * `REPLACE_WITH_AZ`
    * `REPLACE_WITH_BOSH_SECURITY_GROUP`
    * `REPLACE_WITH_DIRECTOR_ID`
    * `REPLACE_WITH_ELASTIC_IP`
    * `REPLACE_WITH_PRIVATE_SUBNET_ID`
    * `REPLACE_WITH_PUBLIC_SECURITY_GROUP`
    * `REPLACE_WITH_PUBLIC_SUBNET_ID`
    * `REPLACE_WITH_SSL_CERT_AND_KEY`
    * `REPLACE_WITH_SYSTEM_DOMAIN`

    We describe replacing these properties in [Step 2: Configure AWS for Your Cloud Foundry Deployment](#config-aws).

1. Run `bosh status --uuid` to retrieve your BOSH Director ID.
Update `REPLACE_WITH_DIRECTOR_ID` in the example manifest with this value.

##<a id="config-aws"></a>Step 2: Configure AWS for Your Cloud Foundry Deployment

To configure your AWS account for Cloud Foundry:

* [Create a NAT VM](#create-nat-vm)
* [Update the MicroBOSH Security Group](#update-mibo-sec-group)
* [Create a Subnet for Cloud Foundry Deployment](#create-cf-subnet)
* [Configure your Cloud Foundry System Domain](#config-cf-dns)

<p class="note"><strong>Note</strong>: Before configuring these components, ensure that you set "N. Virginia" as the AWS Region.</p>

###<a id="create-nat-vm"></a>Create a NAT VM

1. On the EC2 Dashboard, click **Launch Instance**.
1. Click **Community AMIs**.
1. Search for and select "amzn-ami-vpc-nat-pv-2014.09.1.x86_64-ebs".
1. Select "m1.small".
1. Click **Next: Configure Instance Details** and complete as follows:
    * **Network**: Select your "bosh" VPC.
    * **Subnet**: Select your "Public subnet".
    * **Auto-assign Public IP**: Select "Enable".
1. Click **Next: Add Storage**.
1. Click **Next: Tag Instance**.
1. Enter "NAT" as the **Value** for the "Name" **Key**.
1. Click **Next: Configure Security Group**.
1. Click **Create a new security group** and complete as follows:
    * **Security group name**: "nat"
    * **Description**: "NAT Security Group"
    * **Type**: "All traffic""
    * **Protocol**: "All"
    * **Port Range**: "0 - 65535"
    * **Source**: "Custom IP / 10.0.16.0/24"
1. Click **Review and Launch**.
1. Click **Launch**.
1. Specify "Choose an existing key pair" and "bosh" from the dropdown
menus.
1. Click **Launch Instances**.
1. Click **View Instances**.
1. Select the "NAT" instance in the **Instances** list.
1. Click **Actions**, then **Networking**, then **Change Source/Dest. Check**.
1. Click **Yes, Disable**.

###<a id="update-mibo-sec-group"></a> Update the MicroBOSH Security Group

1. On the VPC Dashboard, click **Security Groups**.
1. Select the "bosh" security group.
1. Click **Inbound Rules** at the bottom.
1. Click **Edit** and add a rule as follows:
    * **Type**: "Custom TCP Rule"
    * **Protocol**: "TCP (6)"
    * **Port Range**: "4443"
    * **Source**: "0.0.0.0/0"
1. Click **Save**.
1. Update `REPLACE_WITH_BOSH_SECURITY_GROUP` in your manifest with the name of the "bosh" security group. Note: You must use add the name of the security group to the manifest, not the security group ID.

###<a id="create-cf-subnet"></a> Create a Subnet for Cloud Foundry Deployment

1. Click **Subnets** from the VPC Dashboard.
1. Click **Create Subnet** and complete as follows:
    * **Name tag**: cf
    * **VPC**: bosh
    * **Availability Zone**: Pick the same Availability Zone as the "bosh" subnet.
    * **CIDR block**: 10.0.16.0/24
    * Click **Yes, Create**.
1. Replace the following in your manifest:
    * `REPLACE_WITH_AZ` with the Availability Zone you chose.
    * `REPLACE_WITH_PRIVATE_SUBNET_ID` with the Subnet ID for the "cf" subnet.
    * `REPLACE_WITH_PUBLIC_SUBNET_ID` with the Subnet ID for the "Public subnet" subnet.
1. Select the "cf Subnet" from the **Subnet** list.
1. Click the **Route table** tab in the bottom window to view the route tables.
1. Click the route table ID link in the **Route Table** field.
1. Select the route to display the route details in the bottom window. Click the **Routes** tab.
1. Click **Edit** and complete as follows:
    * **Destination**: 0.0.0.0/0
    * **Target**: Select the NAT instance from the list.
    * Click **Save**.
1. Click **Elastic IPs** from the VPC Dashboard.
1. Click **Allocate New Address** and click **Yes, Allocate**.
1. Update `REPLACE_WITH_ELASTIC_IP` in your manifest with the new IP address.
1. Click **Security Groups** from the VPC Dashboard.
1. Click **Create Security Group**.
    * **Name tag**: cf-public
    * **Group name**: cf-public
    * **Description**: cf Public Security Group
    * **VPC**: Select the "bosh" VPC.
    * Click **Yes, Create**.
1. In the **Inbound Rules** tab in the bottom window, click **Edit** and add the following inbound rules:
<table border="1" class="nice">
	<tr>
		<th>Type</th>
		<th>Protocol</th>
		<th>Port Range</th>
		<th>Source</th>
	</tr>
	<tr><td>HTTP</td><td>TCP</td><td>80</td><td>0.0.0.0/0</td></tr>
	<tr><td>HTTPS</td><td>TCP</td><td>443</td><td>0.0.0.0/0</td></tr>
	<tr><td>Custom TCP Rule</td><td>TCP</td><td>4443</td><td>0.0.0.0/0</td></tr>
</table>

1. Update `REPLACE_WITH_PUBLIC_SECURITY_GROUP` in your manifest with the name of the "cf-public" security group. Note: You must add the name of the security group to the manifest, not the security group ID.

###<a id="config-cf-dns"></a> Configure your Cloud Foundry System Domain

If you have a domain you plan to use as your Cloud Foundry System Domain, use
the procedure below to set up your DNS.
If you do not have a domain, skip steps 1-5 and use YOUR\_ELASTIC\_IP.xip.io as
your System Domain.

Create a wildcard DNS entry for your root System Domain in the form
`*.ROOT_SYSTEM_DOMAIN.com` to point at the Elastic IP address you created in the
[Create a Subnet for Cloud Foundry Deployment](#create-cf-subnet) section.

1. Click on **Route 53** from the AWS Console.
1. Click on **Hosted Zones**.
1. Create a zone or select an existing zone.
1. Click **Go to Record Sets**.
1. Click **Create Record Set** and complete as follows:
    * **Name**: *
    * **Type**: A - IPv4 address
    * **Value**: Enter the Elastic IP that you created in the previous section.
    * Click **Create**.

1. Update `REPLACE_WITH_SYSTEM_DOMAIN` with your root System Domain, for example, `your-cf-domain.com`.
1. Run the following series of commands to generate an SSL certificate for your system domain:

    <pre class="terminal">
    $ openssl genrsa -out cf.key 1024
    </pre>

    <p class="note"><strong>Note</strong>: The following command displays multiple prompts for certificate request information. For the <code>Common Name</code> prompt, enter <code>*.your-system-domain</code>.</p>

    <pre class="terminal">
    $ openssl req -new -key cf.key -out cf.csr
    </pre>

    <pre class="terminal">
    $ openssl x509 -req -in cf.csr -signkey cf.key -out cf.crt
    </pre>

    <pre class="terminal">
    $ cat cf.crt && cat cf.key
    </pre>

1. Update `REPLACE_WITH_SSL_CERT_AND_KEY` in your manifest with the value from the above command.


