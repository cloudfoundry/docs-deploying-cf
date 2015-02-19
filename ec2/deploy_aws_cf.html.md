---
title: Deploying Cloud Foundry on AWS
---

This topic leverages the AWS and Cloud Foundry components defined and uploaded in the [Deploying MicroBOSH to AWS](../../bosh/deploy-microbosh-to-aws.html) and [Configuring AWS for Cloud Foundry](./configure_aws_cf.html) topics.

Run `bosh stemcells` to view stemcells that are active in your MicroBOSH deployment.

##<a id="release"></a>Select and Upload Release ##

1. Navigate to the releases subdirectory:

    <pre class="terminal">
    cd ~/cf-deployment/cf-release/releases/
    </pre>

1. Select and upload a recent release:

    <pre class="terminal">
    bosh upload release ~/cf-deployment/cf-release/releases/cf-193.yml
    </pre>

1. To confirm that the release was successful:

    <pre class="terminal">
    bosh releases
    </pre>

<p class="note"><strong>Note</strong>: If you get a blobstore error that indicates that the device has no space left, your machine ran out of space in the <code>/tmp</code> directory. To fix this, find a larger local partition and execute the following commands to point <code>/tmp</code> to a larger device:</p>

<pre class="terminal">
sudo su -
mkdir /tmp2
mount --bind /tmp2 /tmp
sudo chown root.root /tmp
sudo chmod 1777 /tmp
</pre>

##<a id="deploy"></a>Deploy Cloud Foundry on AWS ##

To deploy Cloud Foundry on AWS:

1. Deploy the manifest you created in the [Configuring AWS for Cloud Foundry](./configure_aws_cf.html) topic.

    <pre class="terminal">
    bosh deployment ~/PATH_TO_YOUR_MANIFEST/MANIFEST_NAME.yml
    </pre>

1. Deploy BOSH.

    <pre class="terminal">
    bosh deploy
    </pre>


Re-run `bosh deploy` if you receive the message: `Error 400007: 'api/0' is not running after update`

##<a id="access-cf-vms"></a>Access Cloud Foundry VMs ##

To manage your Cloud Foundry deployment, you need to access the VMs using SSH. 

Run the `bosh vms` command to view a list of deployed VMs. 

Access the VMs with the following command:

<pre class="terminal">
bosh ssh VM_FROM_BOSH_VMS_COMMAND/INSTANCE_NUMBER --gateway_host YOUR_PUBLIC_MICROBOSH_ADDRESS --gateway_user vcap
</pre>

