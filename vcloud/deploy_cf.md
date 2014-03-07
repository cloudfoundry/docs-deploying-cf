---
title: Deploying Cloud Foundry with BOSH
---

This guide describes the process for deploying Cloud Foundry to a vcloud environment using BOSH.

## <a id="prerequisites"></a>Prerequisites ##

* BOSH should be deployed. See the steps in the [previous section](deploying_bosh_with_micro_bosh.html).

## <a id="target"></a>Target New BOSH Director ##

Target the Director of the deployed BOSH using `bosh target` and the IP address of the Director.

<pre class='terminal'>
$ bosh target 10.146.21.153
$ bosh status
Updating director data... done

Director
  Name      bosh director
  URL       http://10.146.21.153:25555
  Version   0.7 (release:fb1aebb0 bosh:20f2ca20)
  User      admin
  UUID      2250612f-f0e4-41b3-b1b2-3d730e9011a7
  CPI       vcloud
  dns       disabled

Deployment
  not set
</pre>

## <a id="upload-stemcell"></a>Upload a Stemcell ##

The Director needs a stemcell in order to deploy Cloud Foundry. If you follow the doc, you should already have vcloud stemcell downloaded in the step of deploying micro_bosh/bosh. Please upload that stemcell.

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

## <a id="get-release"></a>Get a Cloud Release ##

For this exercise, we'll use a release from the public repository:

<pre class="terminal">
$ git clone git@github.com:cloudfoundry/cf-release.git
$ cd cf-release
$ bosh upload release releases/appcloud-xxx.yml # use the highest number available - inspecting the files in this directory
</pre>

You'll see a flurry of output as BOSH configures and uploads release components.

<!---  Gabi and Matt stopped here until Monday :) -->

## <a id="create-manifest"></a>Create a Cloud Deployment Manifest ##

For the purpose of this tutorial, we'll use a sample [deployment manifest](http://docs.cloudfoundry.com/docs/running/deploying-cf/vcloud/cloud-foundry-example-manifest.html). 

Use the BOSH CLI to set the deployment manifest file. 

<pre class="terminal">
$ bosh deployment ~/deployments/cloudfoundry.yml
Deployment set to '/home/user/deployments/cloudfoundry.yml'
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

## <a id="verfy"></a>Verify the Deployment ##

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

The Cloud Foundry deployment should now be ready to use. You can now follow the instructions in the [Using](/docs/using/index.html) section of these docs to install the [cf](/docs/using/managing-apps/cf/index.html) command-line tool and push an application.
