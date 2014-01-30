---
title: Configuring AWS for BOSH
---
You must alter the AWS configuration in order to deploy the BOSH
server(s) to AWS EC2:

* Add additional ports to the "bosh" Security Group

* Add an additional Elastic IP (two additional for a deployment with multiple BOSH servers)

### Additional Ports

BOSH needs TCP ports (53, 4222, 25250 and 25777) and a UDP port (53)
opened for inbound to the previously created security group "bosh".
We will also open ports 1 through 65535 so that all servers within
that security group can talk to each other.

**Note**: During the process below, your changes are not complete until you click **Apply Rule Changes**.
If you click on another security group during this process, you will lose any unapplied changes.

Starting with the BOSH Director, which needs port 4222, select the
newly created Security Group and click **Inbound**.
In the Port range box, enter “4222” and click **Add Rule**.

![image alt text](ec2/image_18.png)

Now add the second port, 53, by entering this port number into the Port range box and clicking **Add Rule**.

![image alt text](ec2/image_19.png)

Repeat this for the remaining ports 25250.

Now open TCP ports 1-65535 to all servers within the Security Group by
entering this port number, 1-65535, into the Port range box, entering
the Group ID associated with this "bosh" Security Group (yes, this at
first sounds circular, but it will eventually make sense) and clicking
**Add Rule**. ![image alt text](ec2/image_20.png)

Now open UDP port 53 (you’ve already opened it for TCP).
In the Create a new rule dropdown, select Custom UPD rule, enter 53 into the Port range and click "Add Rule."
Since this is the last port you are adding, click “Apply Rule Changes”
to save the changes to this Security Group.

![image alt text](ec2/image_21.png)

### Additional Elastic IPs

Ensure that "N. Virginia" is selected as the AWS Region, then click
 **Elastic IPs > Allocate New Address**.

![image alt text](ec2/image_22.png)

Leave "EC2" selected in the dropdown menu and click **Yes, Allocate** to confirm.

![image alt text](ec2/image_23.png)

Take note of the newly created IP address; you will use it later.
Your address will be different than what is listed here.
Also, notice that the first IP address we created is currently allocated to the Micro Bosh instance. ![image alt text](ec2/image_24.png)

###Go on to [Deployment of BOSH on AWS](./deploy_aws_bosh.html) or [Return to Index](./index.html)