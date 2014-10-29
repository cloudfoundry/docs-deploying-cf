---
title: Deploying Cloud Foundry on AWS
---

These steps leverage the additional security group and ports opened in the previous step to deploy Cloud Foundry from the BOSH server:

* Obtain and Upload Release

* Upload a Stemcell

* Modify a Deployment Manifest

* Deploy

### Obtain and Upload Release

The original documentation was borrowed heavily from [here](http://www.google.com/url?q=http%3A%2F%2Fdocs.cloudfoundry.com%2Fdocs%2Frunning%2Fdeploying-cf%2Fcommon%2Fcf-release.html&sa=D&sntz=1&usg=AFQjCNEnuPr0Fy05RET8NliV2OUS-sjwhw); however, you can follow the instructions below.

Create a folder on the local computer to store the CF Release:

<pre class="terminal">
  mkdir -p ~/bosh-workspace/releases/
  cd ~/bosh-workspace/releases/
  git clone -b release-candidate git://github.com/cloudfoundry/cf-release.git
  cd ~/bosh-workspace/releases/cf-release
</pre>

Navigate to the releases folder and pick a recent release, then upload:

<pre class="terminal">
  bosh upload release ~/bosh-workspace/releases/cf-release/releases/cf-146.yml
</pre>

**Note**: If you get a blobstore error "No space left of device…",
the local computer ran out of space in the `/tmp` folder. To fix
this, find a larger local partition and execute the following commands to point `/tmp` to a larger device:

<pre class="terminal">
  sudo su -
  mkdir /tmp2
  mount --bind /tmp2 /tmp
  sudo chown root.root /tmp
  sudo chmod 1777 /tmp
</pre>

To confirm that the release was successful:

<pre class="terminal">
  bosh releases
</pre>

### Obtain and Upload Stemcell

These are the same instructions that we used to upload the latest stemcell of BOSH onto the MicroBOSH server:

<pre class="terminal">
  bosh upload stemcell https://s3.amazonaws.com/bosh-jenkins-artifacts/bosh-stemcell/aws/bosh-stemcell-1274-aws-xen-ubuntu.tgz
</pre>


After the upload is complete, view the list of stemcells by calling:

<pre class="terminal">
  bosh stemcells
</pre>

### Generate a Deployment Manifest

You need a deployment manifest to deploy Cloud Foundry.
We recommend using [Spiff](https://github.com/cloudfoundry-incubator/spiff) to
generate Cloud Foundry deployment manifests.

Spiff produces a deployment manifest by merging environment-specific information
that you provide in a stub with the most current Cloud Foundry release
templates.

Follow the instructions in the [Generating a Cloud Foundry Deployment Manifest Using Spiff](../cf-manifest-spiff.html) topic.

### Deploy Cloud Foundry on AWS

Everything is now in place to use the deployment manifest you have
created and deploy Cloud Foundry from the BOSH server to AWS. Let’s now do this.

Select the deployment file:

<pre class="terminal">
  bosh deployment ~/bosh-workspace/deployments/cf/cf.yml</td>
</pre>


Deploy the BOSH:

<pre class="terminal">
  bosh deploy
</pre>


If you receive the following error, try running "bosh deploy" again:

**Error 400007: `api/0' is not running after update**

###Go on to [Configuring AWS for CF](./configure_aws_cf.html) or [Return to Index](./index.html)