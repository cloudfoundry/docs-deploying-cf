---
title: Deploying Cloud Foundry to vCloud Air or vCloud Director
---

## Overview ##

This topic is intended to help you get started with open source Cloud Foundry
on vCloud Air or on a supported vCloud Director (vCD) deployment (5.1+).
Note that these instructions emphasize simplicity, not security, scale, or
other production considerations.

The goal of these instructions is to help you deploy an environment that looks like the following diagram. Specifically:

* We deploy to a private network.
* We deploy a jump box VM in the private network. We carry out all other deployment from the jump box VM.
* We provide outbound connectivity for all VMs from the private network to a public network.
* We provide inbound access to the jump box VM and to the Cloud Foundry instance from the public network.

![vCloud Air / vCloud Director Cloud Foundry deployment](/vcloud_images/vcloud_cf_deployment_vms.png)

Follow the steps below to install open source Cloud Foundry on vCloud Air or vCloud Director.

1. [Set up vCloud Air or vCloud Director](setup_vcloud.html):
Configure a **virtual datacenter** for the installation, then deploy a **jump box** in the virtual datacenter.
The rest of the installation is driven from this jump box.

2. [Deploy MicroBOSH](deploying_micro_bosh.html):
Deploy **MicroBOSH** from the jump box. MicroBOSH is a single VM version of
BOSH.

3. (**Optional**) [Deploy BOSH using MicroBOSH](deploying_bosh_with_micro_bosh.html):
If you need production level resilience and/or large scale, install **BOSH** using MicroBOSH.
Note that this is not illustrated in the above diagram.
Most installations of Cloud Foundry can be done without this step by using MicroBOSH installed in step 3. Since it adds complexity, we suggest avoiding this step if possible.

4. [Deploy Cloud Foundry](deploy_cf.html):
Use either MicroBOSH or BOSH to deploy Cloud Foundry.

Consult the [Troubleshooting](troubleshooting.html) page for assistance.

## Sample Manifest Files ##

* [MicroBOSH Deployment Manifest for vCloud](micro-bosh-example-manifest.html)
* [BOSH example manifest](bosh-example-manifest.html)