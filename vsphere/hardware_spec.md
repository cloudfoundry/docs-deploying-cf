---
title: Hardware Requirements
---

This page shows the hardware requirements for installing BOSH and Cloud Foundry on vSphere.

Cloud Foundry requires an ESXi host on which to run; the hardware requirements for running ESXi 5.5 can be found [here](http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2052329).

Cloud Foundry also requires a vCenter to manage the ESXi host; the hardware requirements for running vCenter Server can be found [here](http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2005086).

## Minimum Requirements

The ESXi host on which Cloud Foundry will be installed must have the following resources available:

* Processor: **2 Physical CPU Cores** (hyperthreading does not count)
* RAM: **48 GiB RAM**
* Hard Disk Space: **500 GB**
* 1 NIC (Network Interface Controller)

## Recommended

For internal small-scale testing and proof-of-concept, we recommend the following:

* Processor: **8 Physical CPU Cores** (hyperthreading does not count)
* RAM: **128 GiB RAM**
* Hard Disk Space: **1 TB**
* 2 NICs (Network Interface Controller)

