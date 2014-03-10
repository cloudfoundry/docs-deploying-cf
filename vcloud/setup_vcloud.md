---
title: Set up vCHS or vCloud Virtual Data Center Resources
---


## <a id="prerequisites"></a>Prerequisites ##

### vCloud Hybrid Service###
To get started with Cloud Foundry on **vCloud Hybrid Service** you will need a **vCloud Hybrid Service account** and a **virtual datacenter** (vDC) with at least 40GB memory, 120GB disk and 28 vCPU cores. You will need at least two public IP addresses, one to connect to the jump box VM and one for Cloud Foundry.

Your user account will need Virtual Infrastructure Administrator and Network Administrator privileges for this virtual datacenter.

A lot of the actions we take during this installation will be carried out in the vCloud Director UI. When using vCHS, you can open the **vCloud Director UI** for your virtual datacenter as follows:

Click on your virtual datacenter to open the virtual datacenter details.
![vCHS Dashboard](/vcloud_images/vchs_dashboard.png)

Next click on  "Manage Catalogs in vCloud Director" to open the vCloud Director UI.
![vCHS vDC](/vcloud_images/vchs_vdc_vcdui.png)


### vCloud Director###
To get started with Cloud Foundry on **vCloud Director** you will need an **account** in a [vCloud organization](http://pubs.vmware.com/vcd-51/topic/com.vmware.vcloud.users.doc_51/GUID-B2D21D95-B37F-4339-9887-F7788D397FD8.html) and a vCloud **virtual datacenter** with an Internet routable network and a block of assigned IP addresses. The recommended resources for a full CF deployment introduced in this doc is to have 28 CPU cores, 39 GB Memory, 120 GB disk, 23 IPs to assign. 

Your user account will need [organization administrator](http://pubs.vmware.com/vcd-51/topic/com.vmware.vcloud.users.doc_51/GUID-5B60A9C0-612A-4A3A-9ECE-694C40272505.html) privileges for this virtual datacenter.

## <a id="catalog"></a>Add a catalog ##

In the vCloud Director UI, choose "Catalogs" on the top row of buttons, then "My Organization's Catalogs". 

Add a catalog in vCloud Director where stemcells and media (ISOs) for BOSH will be stored.
Next click on  "Manage Catalogs in vCloud Director" to open the vCloud Director UI.

![vCloud Catalog](/vcloud_images/vcloud_catalog.png)


## <a id="jumpbox"></a>Setting up a jump box ##


Cloud Foundry installation on vCHS / vCloud Director is simplest if you create a machine within the target vDC (a **jump box** VM) where you drive the rest of the installation process. We typically use Ubuntu 12.04 for this machine, and configure it with 1GB memory, 1GHz vCPU and 16GB storage. The rest of this documentation will assume you have created this machine in the target vDC.

To create this machine:

1. Download the latest Ubuntu 12.04 LTS server ISO image from http://www.ubuntu.com/download/server
2. Upload this ISO to the catalog you created above. Choose the "Media and Other" tab, and then use the upload button.
3. In your virtual datacenter, create a VM connecting the ISO to the VM. Start up the VM and install Ubuntu.
4. Once it starts, assign an IP address to the jump box by logging into the VM using [Ubuntu's instructions](https://help.ubuntu.com/10.04/serverguide/network-configuration.html#static-ip-addressing).
5. Set up networking so that you can log into the VM. On vCHS, this will require you to set up a SNAT rule and firewall rule appropriately. After this step you should be able to ssh to the jump box.
6. Finally, install the [BOSH CLI](../../bosh/setup/index.html) on the jump box.


## <a id="network"></a>Create a network for Cloud Foundry ##

Add a network to the virtual datacenter in vCloud Director. Here are the major steps to create a network from the vCloud Director UI:
	1. Click the Manage & Monitor tab and click Organization vDCs in the left pane.
	2. Double-click the organization vDC name to open the organization vDC.
	3. Click the Org vDC Networks tab and click Add Network.
	4. Then choose to add an external direct network or an external routed network. For security and better usage of IP resources, the external routed network is recommended. For adding routed network detailed steps, please refer this link: 
 [Create an External Routed Organization vDC Network](http://pubs.vmware.com/vcd-51/index.jsp#com.vmware.vcloud.admin.doc_51/GUID-6E69AF88-31E0-4DD8-A79E-E8E4B6F68878.html).
 	5. To allow machines on the routed network to talk outside the network, e.g. the micro BOSH, [configure a source NAT rule on the network](http://pubs.vmware.com/vcd-51/index.jsp?topic=%2Fcom.vmware.vcloud.admin.doc_51%2FGUID-464E27A8-3238-4553-ABCF-77808D3A510D.html).

![vcloud_source_nat](/vcloud_images/vcloud_source_nat.png)

