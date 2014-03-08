---
title: Deploying Micro BOSH
---

BOSH is a system used to deploy and manage a Cloud Foundry installation. Like Cloud Foundry, BOSH itself is a distributed system. Micro BOSH is a single VM that contains all BOSH components. For most installation purposes, micro BOSH can be used to install Cloud Foundry.
If you are deploying a large, production instance of Cloud Foundry you may prefer to use a multi-VM deployment of BOSH for scale or resiliency purposes. To deploy this, you deploy micro BOSH and then use micro BOSH to deploy BOSH.

A good way to think about this two step process is to consider that BOSH is a
distributed system in itself.
Since BOSH's core purpose is to deploy and manage distributed systems, it makes
sense that we would use it to deploy itself.
On the BOSH team, we gleefully refer to this as [Inception](http://en.wikipedia.org/wiki/Inception).


## <a id="bootstrap"></a>BOSH Bootstrap ##

### <a id="prerequisites"></a>Prerequisites ###

We recommend that you run the BOSH bootstrap from Ubuntu since it is the
distribution used by the BOSH team, and has been thoroughly tested. It's simplest if you use the Ubuntu jump box VM created earlier in the installation process.

Plan to have around 8 GB of free disk space for the bosh_cli if you plan to use it to deploy CF releases.
You'll need around 3 GB free disk space in /tmp

* Install some core packages on Ubuntu  that the BOSH deployer depends on.

<pre class="terminal">
$ sudo apt-get -y install libsqlite3-dev genisoimage libxslt-dev libxml2-dev
</pre>

* Install Ruby and RubyGems. Refer to the [Installing Ruby](/docs/common/install_ruby.html) page for help with Ruby installation. We recommend using rbenv to install a recent version of ruby 1.9.3 as we have found the versions available using apt-get on Ubuntu are out of date and don't work well.

* Install the BOSH deployer Ruby gem.

<pre class="terminal">
$ gem install bosh_cli --pre --no-ri --no-rdoc
$ gem install bosh_cli_plugin_micro --pre --no-ri --no-rdoc
</pre>

Once you have installed the deployer, you will be able to use `bosh micro`
commands.
To see help on these type `bosh help micro`

### <a id="config"></a>Configuration ###

BOSH deploys things from a subdirectory under a deployments directory.
So create both and name it appropriately.
In our example we named it micro01.

<pre class="terminal">
	mkdir deployments
	cd deployments
	mkdir micro01
</pre>

BOSH needs a deployment manifest for MicroBOSH.
It must be named `micro_bosh.yml`.
Create one in your new directory following the example [MICRO_BOSH example manifest](micro-bosh-example-manifest.html)

## <a id="deploy"></a>Deployment ##

Download a BOSH Stemcell for vCloud deployment:

You will need Internet access for the bosh\_cli to download the stemcells.
You may need to temporarily set the http\_proxy and https\_proxy variables if
you are behind a corporate firewall.
If so, remember to unset it before completing the following steps if your proxy
won't allow contacting the newly micro_bosh vm.

<pre class="terminal">
$ bosh public stemcells
+---------------------------------------------+
| Name                                        |
+---------------------------------------------+
| bosh-stemcell-xxxx-aws-xen-ubuntu.tgz       |
| bosh-stemcell-xxxx-aws-xen-centos.tgz       |
| light-bosh-stemcell-xxxx-aws-xen-ubuntu.tgz |
| light-bosh-stemcell-xxxx-aws-xen-centos.tgz |
| bosh-stemcell-xxxx-openstack-kvm-ubuntu.tgz |
| bosh-stemcell-xxxx-vsphere-esxi-ubuntu.tgz  |
| bosh-stemcell-xxxx-vsphere-esxi-centos.tgz  |
| bosh-stemcell-xxxx-vcloud-esxi-ubuntu.tgz   |
| bosh-stemcell-xxxx-vcloud-esxi-centos.tgz   |
+---------------------------------------------+
$ bosh download public stemcell bosh-stemcell-XXXX-vcloud-esxi-ubuntu.tgz
</pre>

CD to the deployments directory and set the deployment.
This assumes you named the directory micro01.

<pre class="terminal">
$ cd deployments
$ bosh micro deployment micro01
Deployment set to '/var/vcap/deployments/micro01/micro_bosh.yml'
</pre>

Deploy a stemcell for MicroBOSH.

<pre class="terminal">
$ bosh micro deploy bosh-stemcell-XXXX-vcloud-esxi-ubuntu.tgz
</pre>


### <a id="verify"></a>Checking Status of a Micro BOSH Deploy ###

Target the Microbosh

<pre class="terminal">
bosh target <ip_address_from_your_micro_bosh_manifest:25555>
</pre>

Login with admin/admin.

The `status` command will show the persisted state for a given micro BOSH
instance.

<pre class="terminal">
$ bosh micro status
Stemcell CID   sc-1744775e-869d-4f72-ace0-6303385ef25a
Stemcell name  bosh-stemcell-1341-vsphere-esxi-ubuntu
VM CID         vm-ef943451-b46d-437f-b5a5-debbe6a305b3
Disk CID       disk-5aefc4b4-1a22-4fb5-bd33-1c3cdb5da16f
Micro BOSH CID bm-fa74d53a-1032-4c40-a262-9c8a437ee6e6
Deployment     /home/user/cloudfoundry/bosh/deployments/micro_bosh/micro_bosh.yml
Target         https://10.146.21.150:25555 #IP Address of the Director
</pre>

### <a id="listing"></a>Listing Deployments ###

The `deployments` command prints a table view of `bosh-deployments.yml`:

<pre class="terminal">
$ bosh micro deployments
</pre>

The files in your current directory need to be saved if you later want to be
able to update your micro BOSH instance.
They are all text files, so you can commit them to a git repository to make
sure they are safe in case your bootstrap VM goes away.


### <a id="send-message"></a>Sending Messages to the Micro BOSH Agent ###

The `bosh` CLI can send messages over HTTP to the agent using the `agent`
command.

<pre class="terminal">
$ bosh micro agent ping
"pong"
</pre>

