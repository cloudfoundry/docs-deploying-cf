---
title: Configuring AWS for Cloud Foundry
---

### Additional Elastic IPs

Ensure that "N. Virginia" is selected as the AWS Region, then click **Elastic IPs > Allocate New Address**.

![image alt text](ec2/image_27.png)

Leave "EC2" selected in the dropdown menu, then click **Yes, Allocate** to confirm.

![image alt text](ec2/image_28.png)

Take note of the newly created IP address; you will use it later in the CF Deployment Manifest.
Your address will be different than what is listed here.
Also, notice that the two IP addresses we created are currently allocated to the Micro Bosh and BOSH instances.

![image alt text](ec2/image_29.png)

### Create a new Security Group

Ensure that "N. Virginia" is selected as the AWS Region, then click **Security Groups > Create Security Group**.

![image alt text](ec2/image_30.png)

On the pop-up dialog, assign "cf" to the Name tag, "cf" to Group tag, "cf Security Group" to the Description, and select VPC for the BOSH in the dropdown menu.
 Click **Yes, Create** to confirm.

![image alt text](ec2/image_31.png)

### Open Ports for the new Security Group

Open the [AWS Console for Security Groups](https://www.google.com/url?q=https%3A%2F%2Fconsole.aws.amazon.com%2Fec2%2Fhome%3Fregion%3Dus-east-1%23s%3DSecurityGroups&sa=D&sntz=1&usg=AFQjCNGEowcsPVCqMAhuqS27xnaVuvKiIg) and click **Create Security Group**.

Click **Create Security Group** and create the group with the name "cf".

Add TCP ports:

* 22 (ssh)

* 80 (http)

* 443 (https)

* 4222

* 1-65535 (for security group VMs only)

Add UDP ports:

* 1-65535 (for security group VMs only)

Select the **Inbound** tab, enter "22" into the "Port range:" box and click **Add Rule**.  Repeat this for ports 80 and 443.  To add TCP 1 -
65535, enter "1-66535" into the "Port range:" box, enter the name
of the cf Security Group into the "Source:" box, and click **Add Rule**.
 To add UDP 1 - 65535, select "Custom UDP rule" from the "Create a new
rule:" dropdown box, enter "1-66535" into the "Port range:" box,
enter the name of the cf Security Group into the "Source:" box, and
click **Add Rule**. When you are done, click **Apply Rule Changes**.
 The screen should look similar to the image below:

![image alt text](ec2/image_32.png)

###Go on to [Deploying Cloud Foundry on AWS](./deploy_aws_cf.html) or [Return to Index](./index.html)