---
title: Prerequisites
---

To get started with BOSH on vCloud Hybrid Service you will need:

1. A **vCloud Hybrid Service account**
2. A **virtual datacenter** (vDC) with at least 40GB memory, 120GB disk and 28 vCPU cores. To connect to the service, or to be able to set up and configure the jump box VM, you will need at least two public IP addresses (one for the jump box, one for Cloud Foundry).
3. Installation is simplest if you create a machine within the target vDC (a **jump box** VM) where you drive the rest of the installation process. We typically use Ubuntu 12.04 for this machine, and configure it with 1GB memory, 1GHz vCPU and 16GB storage. The rest of this documentation will assume you have created this machine in the target vDC.
4. The [BOSH CLI](../../bosh/setup/index.html) installed on the jump box.

To get started with BOSH on vCloud you need:

1. An **account** in a [vCloud organization](http://pubs.vmware.com/vcd-51/topic/com.vmware.vcloud.users.doc_51/GUID-B2D21D95-B37F-4339-9887-F7788D397FD8.html) with [organization administrator](http://pubs.vmware.com/vcd-51/topic/com.vmware.vcloud.users.doc_51/GUID-5B60A9C0-612A-4A3A-9ECE-694C40272505.html) privileges. 
2. A vCloud **virtual datacenter** with an Internet routable network and a block of assigned IP addresses. The recommended resources for a full CF deployment introduced in this doc is to have 28 CPU cores, 39 GB Memory, 120 GB disk, 23 IPs to assign. 
3. Installation is simplest if you create a machine within the target vDC (a **jump box** VM) where you drive the rest of the installation process. We typically use Ubuntu 12.04 for this machine, and configure it with 1GB memory, 1GHz vCPU and 16GB storage. The rest of this documentation will assume you have created this machine in the target vDC.
4. The [BOSH CLI](../../bosh/setup/index.html) installed on the jump box.
