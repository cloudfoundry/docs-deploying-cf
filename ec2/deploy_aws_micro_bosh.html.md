---
title: Deploy Micro Bosh on AWS
---

We will leverage the Elastic IP, Security Group and Key Pair file that we created in Configuring AWS for Micro BOSH](./configure_aws_micro_bosh.html) to deploy a Micro BOSH server on AWS in four steps:

1. Create Directory Structure

2. Create Micro BOSH Deployment Manifest

3. Find the correct Micro BOSH Stemcell

4. Deploy Manifest

Much of this is borrowed from the blog post that originally announced support for [deploying BOSH to AWS](http://blog.cloudfoundry.com/2012/09/06/deploying-to-aws-using-cloud-foundry-bosh/).

### Create Directory Structure

Create a standard place to store the Deployment Manifest on your local computer:

<pre class="terminal">
mkdir -p ~/bosh-workspace/deployments/microboshes/deployments/microbosh

cd ~/bosh-workspace/deployments/microboshes/deployments/microbosh

touch microbosh.yml
</pre>

**Create Micro BOSH Deployment Manifest**

Now let’s review what the contents of the `microbosh.yml` deployment
manifest file should include. If you did not use the us-east-1a
availability zone, you will need to adjust that. We are using a small instance type for the Micro BOSH.

**Note**: Replace all instances of "x.x.x.x" with
the IP address you created in [Configuring AWS for Micro BOSH](./configure_aws_micro_bosh.html). Because MicroBOSH deploys to a single VM,
all the IP addresses in the manifest are the same. Also ensure that you replace
the AWS access credentials with your own.

~~~yaml
name: microbosh

logging:
  level: DEBUG

network:
  type: dynamic
  vip: x.x.x.x    #Change this to the MicroBOSH IP address

resources:
  persistent_disk: 20000
  cloud_properties:
    instance_type: m1.small
    availability_zone: us-east-1a

cloud:
  plugin: aws
  properties:
    aws:
      access_key_id: xxxxxxxxxxxx	#Change this to your key
      secret_access_key: xxxxxxxxxxx	#Change this to your key
      default_key_name: bosh
      default_security_groups: ["bosh"]
      ec2_private_key: ~/.ssh/bosh
      ec2_endpoint: ec2.us-east-1.amazonaws.com
      region: us-east-1

apply_spec:
  agent:
    blobstore:
      address: x.x.x.x    #Change this to the MicroBOSH IP address
    nats:
      address: x.x.x.x    #Change this to the MicroBOSH IP address
  properties:
    aws_registry:
      address: x.x.x.x    #Change this to the MicroBOSH IP address
~~~

If you are using the sample manifest provided, confirm the following:

1. Make sure to update all of the instances of x.x.x.x IP with the appropriate elastic IP you created.

2. Add the AWS credentials under the AWS section.

3. Verify the AWS region added in the manifest.

Save your changes to the file.

**Find the correct Micro BOSH Stemcell**

It is important to deploy the correct AWS image that is compatible
with the version of Micro BOSH you install.

To obtain the current AMI, navigate to `http://bosh_artifacts.cfapps.io` and download the "light-bosh (aws xen ubuntu)" tarball file
(.tar.gz). The light stemcell is just a wrapper pointing to a public
AMI. If you are not using us-east-1 as specified previously, there
may not be a public AMI available.

![image alt text](ec2/image_14.png)

This downloads the tar file to you local computer. Extract this
and open the file named "stemcell.MF" in your text editor.

![image alt text](ec2/image_15.png)

This file contains the image name (AMI ID) for this version of the
bosh-cli. You will need this AMI ID for the next section.

![image alt text](ec2/image_16.png)

### Deploy Manifest

Everything is now in place to use the deployment manifest you have
created and deploy Micro BOSH to AWS (the first inception). Let’s now
do this.

Enter the deployments folder you created earlier:

<pre class="terminal">
cd ~/bosh-workspace/deployments/microboshes/deployments
</pre>

Select the deployment you called "aws" by telling bosh which manifest file to use:

<pre class="terminal">
  bosh micro deployment microbosh/microbosh.yml
</pre>

Ignore the warning: it is not an error, but displays to inform you that the director was mapped to port 25555:

  ``WARNING! Your target has been changed to `[http://aws:25555](http://aws:25555)``!

Deploy the Micro BOSH AMI using the AMI we retrieved from the stem cell config:

<pre class="terminal">
bosh micro deploy ami-979dc6fe
</pre>

If the deployment fails, clean it up using the command below before
trying again. Otherwise, do not run the command below. If you get errors regarding your access key signature, double check your keys in
`microbosh.yml` and try again. Many of the errors encountered at this
stage will likely be related to having missed a step in configuring AWS or an error in `microbosh.yml`.

<pre class="terminal">
bosh micro delete
</pre>

Log into the new Micro BOSH server using the commands below. The default username and password are admin/admin.

<pre class="terminal">
$ bosh target 54.204.16.249
Current target is https://54.204.16.249:25555 (Bosh Lite Director)
$ bosh login
Your username: admin
Enter password: *****
Logged in as `admin'
</pre>

If the deployment runs successfully, you have a Micro BOSH VM deployed onto AWS.

![image alt text](ec2/image_17.png)

A few things to note in the screenshot above:

1. **microbosh** - The name of the instance matches the deployment name.

2. **us-east-1a** - The availability zone in the deployment manifest

3. **bosh** - The name of the key pair

4. **bosh** - The name of the Security Group

5. **54.204.16.249**- The Elastic IP address that we created, which
is the external IP address for the Micro BOSH server

###Go on to [Configuring AWS for BOSH](./configure_aws_bosh.html) or [Return to Index](./index.html)