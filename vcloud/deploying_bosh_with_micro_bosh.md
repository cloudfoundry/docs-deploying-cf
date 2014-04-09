---
title: Deploying BOSH with Micro BOSH
---

This guide describes the process for deploying BOSH as an application using micro BOSH.

## <a id="prerequisites"></a>Prerequisites ##

* Micro BOSH should be deployed. See the steps in the [previous section](deploying_micro_bosh.html)

## <a id="target"></a>Target Micro BOSH ##

Once your micro BOSH instance is deployed, you can target its Director:

<pre class="terminal">
$ bosh micro status
...
Target         micro (http://10.146.21.150:25555)

$ bosh target 10.146.21.150
Target set to 'your-micro-BOSH'
Your username: admin
Enter password: admin
Logged in as 'admin'
</pre>

*Note*: Create a new user using `bosh create user` which overrides the default username and password.

## <a id="download-stemcell"></a>Download a BOSH Stemcell ##

List public stemcells with the `bosh public stemcells` command:

<pre class="terminal">
$ mkdir -p ~/stemcells
$ cd ~/stemcells
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
To download use `bosh download public stemcell <stemcell_name>'. For full url use --full.
</pre>

Download a public stemcell. *Note*: if you have already downloaded a stemcell to deploy your micro BOSH, you can use that one.

<pre class="terminal">
$ bosh download public stemcell bosh-stemcell-XXXX-vcloud-esxi-ubuntu.tgz
</pre>

Upload the downloaded stemcell to micro BOSH.

<pre class="terminal">
$ bosh upload stemcell bosh-stemcell-XXXX-vcloud-esxi-ubuntu.tgz
</pre>

You can see the uploaded stemcells (on your Micro BOSH) by using `bosh stemcells`:

<pre class="terminal">
$ bosh stemcells

+-------------------------------+---------+-----------------------------------------+
| Name                          | Version | CID                                     |
+-------------------------------+---------+-----------------------------------------+
| bosh-vcloud-esxi-ubuntu       | 1861    | sc-c43d6d06-53b9-47dc-8830-b4e280684a9a |
+-------------------------------+---------+-----------------------------------------+
</pre>

## <a id="upload-release"></a>Upload a BOSH release ##

You can find and download the latest bosh release file from [here](https://bosh-artifacts.cfapps.io/file_collections?type=releases)

<pre class="terminal">
$ bosh upload release bosh-XXXX.tgz
</pre>

## <a id="deploy"></a>Setup a BOSH Deployment Manifest and Deploy ##

Create and setup a BOSH deployment manifest. Look at the [BOSH example manifest](./bosh-example-manifest.html).

Deploy BOSH. (the following assumes your manifest is named `bosh.yml` in `/home/bosh_user`).

<pre class="terminal">
$ cd /home/bosh_user
$ bosh deployment bosh.yml

Deployment set to `/home/bosh_user/bosh.yml'

$ bosh deploy

Getting deployment properties from director...
Unable to get properties list from director, trying without it...
Compiling deployment manifest...
Cannot get current deployment information from director, possibly a new deployment
Please review all changes carefully
Deploying `bosh.yml' to `your-micro-BOSH' (type 'yes' to continue): yes

Director task 5

Preparing deployment
  binding deployment (00:00:00)
  binding releases (00:00:00)
  binding existing deployment (00:00:00)
  binding resource pools (00:00:00)
  binding stemcells (00:00:00)
  binding templates (00:00:00)
  binding unallocated VMs (00:00:00)
  binding instance networks (00:00:00)
Done                    8/8 00:00:00

Preparing package compilation
  finding packages to compile (00:00:00)
Done                    1/1 00:00:00

Preparing DNS
  binding DNS (00:00:00)
Done                    1/1 00:00:00

Creating bound missing VMs
  small/0 (00:00:50)
  small/4 (00:01:02)
  small/1 (00:01:19)
  small/2 (00:01:22)
  small/3 (00:01:24)
  director/0 (00:01:26)
Done                    6/6 00:01:26
</pre>

*Note*: There will be a lot of status information in addition to the above output.

## <a id="verify"></a>Verification ##

The example BOSH director has the IP address `10.146.21.153`. Targeting this director with `bosh target 10.146.21.153` will verify a successful installation.

<pre class="terminal">
$ bosh target 10.146.21.153
Target set to 'your-director'
Your username: admin
Enter password: admin
Logged in as `admin'
</pre>

