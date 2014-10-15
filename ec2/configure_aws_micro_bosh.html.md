---
title: Configure AWS for MicroBOSH
---

This page assumes that you have administrator access to an AWS account.

<p class="note"><strong>Note</strong>: The examples below assume that you the US-EAST-1 AWS Region. If you use a different AWS region, ensure that you create Elastic IPs in the same region as your EC2 instances. You cannot assign IP addresses created for one region to an instance in another region.</p>

##<a id="aws-credentials"></a>Obtain AWS credentials ##

If you already know your AWS credentials (access\_key\_id and secret\_access\_key, which are not the same as your AWS login and password), skip this step.

Start by logging into AWS: [https://console.aws.amazon.com](https://console.aws.amazon.com)

Click **Instances** in the left pane. Make sure to select N. Virginia as your region.![image alt text](ec2/image_0.png)

From the dropdown menu next to your login name, and select **Security Credentials**.

![image alt text](ec2/image_1.png)

Next, select **Create New Access Key**. If there are already keys created, you may want to stop and consult someone in your organization to obtain the existing access\_key\_id and secret\_access\_key, since there is a limit of two sets of Access Keys.

![image alt text](ec2/image_2.png)

Download the security file in CSV format when prompted. 

![image alt text](ec2/image_3.png)

Document the access\_key\_id and secret\_access\_key somewhere privately and securely within your organization and keep these safe. These are
the only two pieces of information needed to fraudulently consume AWS resources.

### Create Elastic IPs

The MicroBOSH server needs a public IP address. AWS calls these
Elastic IPs.

Ensure that "N. Virginia" is selected as the AWS Region, then click
**Elastic IPs > Allocate New Address**.

![image alt text](ec2/image_4.png)

To confirm, click **Yes, Allocate** and leave “EC2” selected in the
dropdown menu.

![image alt text](ec2/image_5.png)

Take note of the newly created IP address; you will use it later.
Your address will be different than what is listed here.

![image alt text](ec2/image_6.png)

### Create a key pair

In order to SSH in to your instances, you need to create a key pair.
If you already have a key pair created, you can skip creating a new
key pair. You still need to place a copy of the PEM file, as shown below.

Ensure that "N. Virginia" is selected as the AWS Region, then click
**Key Pairs > Create Key Pair**.

![image alt text](ec2/image_7.png)

Name your key pair "bosh" and click **Yes**.

![image alt text](ec2/image_8.png)

After you click "Yes", a file downloads to your computer, likely
named `bosh.pem.txt`.

Rename this file "bosh" and save it into your ``~/.ssh`` folder. For
example, on OSX you can do this from the terminal:

<pre class="terminal">
  mv ~/Downloads/bosh.pem.txt ~/.ssh/bosh
</pre>

If you receive an error that ~/.ssh does not exist, you can create
this folder:

<pre class="terminal">
  mkdir -p ~/.ssh
</pre>

### Create a security group

Create a security group for the MicroBOSH server.
This security group exposes ports used by the BOSH Agent.

Ensure that "N. Virginia" is selected as the AWS Region, then click
**Security Groups > Create Security Group**.

![image alt text](ec2/image_9.png)

From the popup dialog, assign "bosh" to the Name, “BOSH Security Group” to the Description, and leave “No VPC” selected in the dropdown menu. Then, click **Yes, Create**.

![image alt text](ec2/image_10.png)

### Add ports to the security group

MicroBOSH needs ports 25555 (BOSH Director), 6868 (BOSH Agent) and
22 (ssh to debug) opened for inbound for the security group "bosh"
that we just created.

**Note**: Some or all of these ports may be blocked on your corporate
firewall. A workaround is to attempt this from your personal home
network.

Starting with the BOSH Director, which needs port 25555, select the newly created Security Group and click **Inbound**. In the Port range box, enter “25555” and click **Add Rule**.

![image alt text](ec2/image_11.png)

Now add the second port, 6868, by entering this port number into the Port range box and clicking **Add Rule**.

![image alt text](ec2/image_12.png)

Now add the last port, 22, by entering this port number into the Port
range box and clicking "Add Rule". Since this is the last port you
are adding, click **Apply Rule Changes** to save the changes to this Security Group.

![image alt text](ec2/image_13.png)

###Go on to [Deployment of MicroBOSH on AWS](./deploy_aws_micro_bosh.html) or [Return to Index](./index.html)