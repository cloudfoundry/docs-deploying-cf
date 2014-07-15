---
title: Deploying Cloud Foundry with Micro BOSH or BOSH
---

This guide describes the process for deploying Cloud Foundry to a vCloud environment using Micro BOSH or distributed BOSH. To keep it simple, the rest of this document will simply call it BOSH.

All of the commands below should be run from the jump box, in the `~/deployments` directory we created earlier.

## <a id="prerequisites"></a>Prerequisites ##

BOSH should be deployed. See the steps in the [previous](deploying_micro_bosh.html) [sections](deploying_bosh_with_micro_bosh.html).

## <a id="target"></a>Target New BOSH Director ##

Target the Director of the deployed BOSH using `bosh target` and the IP address of the Director. Login with admin/admin or the userid and password you set after installation.

<pre class='terminal'>
$ bosh target https://192.168.109.81:25555
Target already set to `bosh_micro_vchs'
$ bosh status
Config
             /home/killian/.bosh_config

Director
  Name       bosh_micro_vchs
  URL        https://192.168.109.81:25555
  Version    1.0000.0 (release:125d9104 bosh:125d9104)
  User       admin
  UUID       566c47b3-ca85-47e2-95f6-32d095747acf
  CPI        vcloud
  dns        enabled (domain_name: microbosh)
  compiled_package_cache disabled
  snapshots  disabled

Deployment
  not set
</pre>

You will need the UUID of the BOSH Director in your Cloud Foundry deployment file later. It is provided in the above list of information also.

## <a id="upload-stemcell"></a>Upload a Stemcell ##

The Director needs a stemcell in order to deploy Cloud Foundry. If you followed the installation documentation, you should already have vcloud stemcell downloaded previously when you deployed Micro BOSH or BOSH. Please upload the same stemcell.

<pre class="terminal">
$ bosh upload stemcell ~/stemcells/bosh-stemcell-xxxx-vcloud-esxi-ubuntu.tgz
Verifying stemcell...
File exists and readable                                     OK
Using cached manifest...
Stemcell properties                                          OK

Stemcell info
-------------
Name:    bosh-vcloud-esxi-ubuntu
Version: 1868

Checking if stemcell already exists...
No

Uploading stemcell...
bosh-vcloud-esxi-ubuntu: 100% |ooooooooooooooooooooooooooooooooooooooo| 277.1MB  78.8MB/s 		Time: 00:00:03

Director task 1

Update stemcell
extracting stemcell archive (00:00:06)
verifying stemcell manifest (00:00:00)
checking if this stemcell already exists (00:00:00)
uploading stemcell bosh-vcloud-esxi-ubuntu/1868 to the cloud (00:01:08)
save stemcell: bosh-vcloud-esxi-ubuntu/1868 (sc-a85ab3dc-8d3d-4228-83d0-5be2436a1886) (00:00:00)
Done                    5/5 00:01:14
Task 1 done
Started		2014-01-26 10:14:26 UTC
Finished	2012-01-26 10:15:40 UTC
Duration	00:01:14

Stemcell uploaded and created
</pre>

After uploading you should be able to see the stem cell when you run `bosh stemcells`

<pre class="terminal">
$ bosh stemcells

+-------------------------+---------+-------------------------------------------------------------+
| Name                    | Version | CID                                                         |
+-------------------------+---------+-------------------------------------------------------------+
| bosh-vcloud-esxi-ubuntu | 0000    | urn:vcloud:catalogitem:dd470e53-dbe4-4ad8-9257-4ac76bce273f |
+-------------------------+---------+-------------------------------------------------------------+

(*) Currently in-use

Stemcells total: 1
</pre>


## <a id="get-release"></a>Get a Cloud Release ##

For this exercise, we'll use a release from the public repository. Clone this repo in the `deployments` directory.

<pre class="terminal">
$ git clone https://github.com/cloudfoundry/cf-release.git
$ cd cf-release
$ bosh upload release releases/cf-xxx.yml # These docs have been tested with cf-147.yml ; they do NOT work with cf-173.yml
</pre>

You'll see a flurry of output as BOSH configures and uploads release components. Here's a shortened version:

<pre class="terminal">
$ bosh upload release releases/cf-147.yml

Copying packages
----------------
rootfs_lucid64 (2)            FOUND LOCAL
saml_login (14)               FOUND LOCAL
haproxy (1)                   FOUND LOCAL
loggregator_trafficcontroller (3) FOUND LOCAL

[… Lots of output …]


Release has been created
  cf/147 (00:00:00)
Done                    1/1 00:00:00

Task 11 done

Started		2014-03-22 00:40:06 UTC
Finished	2014-03-22 00:40:38 UTC
Duration	00:00:32

Release uploaded
</pre>

After finishing you should be able to see the release when you run `bosh releases`

<pre class="terminal">
$ bosh releases

+------+----------+-------------+
| Name | Versions | Commit Hash |
+------+----------+-------------+
| cf   | 147      | 07e0aa91+   |
+------+----------+-------------+
(+) Uncommitted changes

Releases total: 1
</pre>


## <a id="create-manifest"></a>Create a Cloud Foundry deployment manifest file ##

For the purpose of this tutorial, we'll use a sample [deployment manifest template](cloud-foundry-example-manifest.html). Copy this file to the `~/deployments` directory on the jump box, with the name `cf.yml.erb` indicating that it's a YAML file written using erb, the ruby templating engine. It allows us to fill in a set of properties at the beginning of the file and then run `erb cf.yml.erb > cf.yml` to generate a suitable manifest file in `cf.yml`. There are significant comments within this template file, in the first section with all the user defined variables. You should fill out all of the variables provided.

Please note that the goal of this example is to get an instance running **as simply as possible**. A production deployment of Cloud Foundry will likely make **significant changes** to network topology, security settings / certs, scale, etc. etc.

To address some of the properties in the manifest template:

* `director_uuid` - this should be set to the UUID of the Micro BOSH / BOSH being used. You can get this using bosh status.
* `cf_release_name` / `cf_release_version` - these instructions have been tested with the Cloud Foundry release cf-147.
* `vapp_name` - if you include a name all VMs will be created within a single vApp, which allows much simpler cleanup.
* `IP addresses` - in general, you provide a complete range in `ip_range`, you exclude some of these (`reserved_ip_range`), assign some statically to particular VMs and leave some dynamic IPs for assignment to other VMs (in this case the DEAs are dynamic. Typically compilation VMs created by BOSH during deployments are also dynamic).
* `network_name` - If using vCHS, this is lower case despite how it's presented in the UI
* `uaa_login_http_s` - a secure deployment requires the UAA and login server to communicate over https. However, this requires trust between the two endpoints. Since we're using self-signed certs this is a little tricky to set up, so this deployment simply uses http. A production deployment should do this using https.
* `wildcard_domain_name` - this deployment assumes a single (wildcard) domain name for both apps and system. Production deployments would typically use different domains.
* **Cert / private key and public / private key pairs** - there are instructions to generate these in the template file. You can run these instructions on the jump box VM (that's why we installed the `openssl` package when setting up the jump box)

Once you have filled in all the properties at the top of the templated manifest file run `erb` to generate the final manifest file `cf.yml`. Then use the BOSH CLI to set the deployment manifest file:

<pre class="terminal">
$ erb ~/deployments/cf.yml.erb > ~/deployments/cf.yml
$ bosh deployment ~/deployments/cf.yml
Deployment set to '/home/user/deployments/cf.yml'
</pre>

## <a id="deploy"></a>Deploy Cloud Foundry ##

Now deploy Cloud Foundry via using the bosh command`bosh deploy`. This example shows only part of the expected output:

<pre class="terminal">
$ bosh deploy
    Getting deployment properties from director...
	Unable to get properties list from director, trying without it...
	Compiling deployment manifest...
	Cannot get current deployment information from director, possibly a new deployment
    Please review all changes carefully
      Deploying <filename>.yml' to dev148'(type 'yes' to continue): yes
    Director task 31
    Preparing deployment
        binding deployment (00:00:00)
        binding releases (00:00:00)
        binding existing deployment (00:00:00)
        binding resource pools (00:00:00)
    binding stemcells (00:00:00)
    binding templates (00:00:00)
    binding unallocated VMs (00:00:01)
    binding instance networks (00:00:00)
    Done                    8/8 00:00:01        Preparing package compilation
    finding packages to compile (00:00:00)
    Done                    1/1 00:00:00
</pre>

If you run into issues, the Troubleshooting section may be of help.

## <a id="verify"></a>Verify the Deployment ##

Execute the `bosh vms` command to see all the vas deployed.

Output of this command will similar to the listing below. Make sure the "State" of all the jobs is "running".

<pre class="terminal">
$ bosh vms
Deployment `cloudfoundry'

Director task 30


Task 30 done

+-----------------------------+---------+------------------+---------------+
| Job/index                   | State   | Resource Pool    | IPs           |
+-----------------------------+---------+------------------+---------------+
| nfs_server/0                | running | nfs_server       | 10.146.21.174 |
| ccdb/0                      | running | ccdb             | 10.146.21.175 |
| cloud_controller/0          | running | cloud_controller | 10.146.21.176 |
| collector/0                 | running | collector        | 10.146.21.178 |
| health_manager/0            | running | health_manager   | 10.146.21.173 |
| nats/0                      | running | nats             | 10.146.21.172 |
| router/0                    | running | router           | 10.146.21.171 |
| syslog/0                    | running | syslog           | 10.146.21.177 |
| uaa/0                       | running | uaa              | 10.146.21.180 |
| uaadb/0                     | running | uaadb            | 10.146.21.179 |
| dea/0                       | running | dea              | 10.146.21.181 |
| saml_login/0                | running | saml_login       | 10.146.21.181 |
+-----------------------------+---------+------------------+---------------+
</pre>


## <a id="login"></a>Log In ##

From your laptop try to target and log into your Cloud Foundry instance. You can now follow the instructions in the [cf](/devguide/index.html#cf) section of the Developer Guide to [install the cf](/devguide/installcf/install-go-cli.html) command-line tool and push an application.

To log in, the manifest file includes a pre-defined account, user id `admin` and password `cdd0549963e8749b4a04` . You can modify or add to the list of pre-provisioned accounts in the generated manifest. In the jobs section look for uaa -> properties -> uaa ->

## <a id="fin"></a>Finishing up ##

If you run into issues the Troubleshooting section may be of some help.

We hope these instructions have been helpful. If you find errors, want to make updates or have suggestions for further enhancement we would greatly appreciate it. Please follow the standard processes on github (i.e. send pull requests and file issues), along with the bosh-users email list. You can learn more at [cloudfoundry.org](http://www.cloudfoundry.org)

Good luck with your Cloud Foundry deployments!

