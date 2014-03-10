---
title: Deploying Cloud Foundry on vCloud Hybrid Service or vCloud Director
---

## Overview ##

Installation of Cloud Foundry on vCloud Hybrid Service (vCHS) or vCloud Director goes through the following steps:
1. Create and configure a virtual datacenter for the installation
2. Deploy a jump box in the virtual datacenter. The rest of the installation is driven from this jump box.
3. Deploy micro BOSH from the jump box. Micro BOSH is a single VM version of BOSH.
4. If you need production level resilience and/or large scale, install BOSH using micro BOSH. Most installations of Cloud Foundry can be done simply using the micro BOSH installed in stage 3.
5. Install Cloud Foundry using micro BOSH or, if you installed it, BOSH.

In general it is not necessary to install the distributed version of BOSH when doing a Cloud Foundry deployment. It has advantages in scale and resilience for production deployments, but adds complexity if you are doing a smaller dev, test or sandbox environment.

## Details ##

The following steps and sections provide more detail on installing Cloud Foundry on vCloud Director.

[Prerequisites](prerequisites.html)

[Set up vCloud Director](setup-vcloud.html)

[Deploy Micro BOSH](deploying_micro_bosh.html)

[Deploy BOSH using Micro BOSH](deploying_bosh_with_micro_bosh.html)

[Deploy Cloud Foundry using BOSH](deploy_cf.html)

The following pages show example manifests for deploying MICRO_BOSH, BOSH and Cloud Foundry.

[MICRO_BOSH example manifest](micro-bosh-example-manifest.html)

[BOSH example manifest](bosh-example-manifest.html)

[Cloud Foundry example manifest](cloud-foundry-example-manifest.html)