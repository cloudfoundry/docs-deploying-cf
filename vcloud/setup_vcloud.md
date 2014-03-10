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


Cloud Foundry installation on vCHS / vCloud Director is simplest if you create a machine within the target vDC (a **jump box** VM) where you drive the rest of the installation process. We typically use Ubuntu 12.04 for this machine, and configure it with 1GB memory, 1GHz vCPU and 16GB storage (You will need to have around 8 GB of free disk space for the bosh_cli if you plan to use it to deploy CF releases. You'll need around 3 GB free disk space in /tmp)

The rest of this documentation assumes you have created this machine in the target vDC.

1. Download the latest Ubuntu 12.04 LTS server ISO image from http://www.ubuntu.com/download/server
2. Upload this ISO to the catalog you created above. Choose the "Media and Other" tab, and then use the upload button.
3. In your virtual datacenter, create a VM connecting the ISO to the VM. Start up the VM and install Ubuntu.
4. Once it starts, assign an IP address to the jump box by logging into the VM and assigning a static IP address using [Ubuntu's instructions](https://help.ubuntu.com/10.04/serverguide/network-configuration.html#static-ip-addressing).


## <a id="network"></a>Set up networking  ##

Please note that the following instructions are optimized for simplicity rather than security. When planning a production deployment of Cloud Foundry you will need to modify these instructions to meet the needs of your service.

### Create a network ###

If you are using vCHS you can reuse the networks provided out of the box. Typically you will use the routed network - for example, `164-935-default-routed`. Note that the default network names are actually all lower case, despite what is shown in the UI.

If you want to add a network in vCHS, or if you need to in vCloud Director, here are the major steps to create a network from the vCloud Director UI:

1. Click the Manage & Monitor tab and click Organization vDCs in the left pane.
2. Double-click the organization vDC name to open the organization vDC.
3. Click the Org vDC Networks tab and click Add Network.
4. Then choose to add an external direct network or an external routed network. For security and better usage of IP resources, the external routed network is recommended. For adding routed network detailed steps, please refer this link: 
 [Create an External Routed Organization vDC Network](http://pubs.vmware.com/vcd-51/index.jsp#com.vmware.vcloud.admin.doc_51/GUID-6E69AF88-31E0-4DD8-A79E-E8E4B6F68878.html).

### Set up connectivity ###

1. Set up networking so that you can **log into the jump box** VM using SSH. In the vCloud Director UI go to the `Administration` button on the top row, then `Cloud Resources`, then `Virtual Datacenters`, then pick your virtual datacenter. Choose the `Edge Gateways` tab, select the gateway being used and click on the gear to select `Edge gateway services`.
![vCloud Edge Gateway](/vcloud_images/vcloud_edge_gateway.png)

* Create a DNAT rule allowing inbound connections from one of your public IP addresses to the jump box IP address on port 22, to allow SSH from your laptop / desktop to the jump box.
* Create a firewall rule allowing inbound traffic from this public IP to the jump box. After this step you should be able to ssh to the jump box.

2. Set up the network so that you can **connect to the internet** from the jump box and other VMs. This is to allow installation of software, connection to the vCloud API, connectivity between VMs, etc.
  * You will need to configure an SNAT rule allowing outbound connections to a public IP. For simplicity you should do this for the entire network to allow all outbound connections. You can reuse the same IP as used for inbound SSH above. More information is available in the [vCloud Director documentation](http://pubs.vmware.com/vcd-51/index.jsp?topic=%2Fcom.vmware.vcloud.admin.doc_51%2FGUID-464E27A8-3238-4553-ABCF-77808D3A510D.html)
  * You will need to set up a firewall rule allowing outbound traffic to external addresses.

![vcloud_source_nat](/vcloud_images/vcloud_source_nat.png)


## <a id="bosh"></a> Install BOSH CLI ##

The following steps will help you to install the BOSH command line interface (CLI) on the jump box VM:

1. Install some core Ubuntu packages that the BOSH deployer depends on.

        sudo apt-get -y install libsqlite3-dev genisoimage libxslt-dev libxml2-dev

2. Install Ruby and RubyGems. Refer to the [Installing Ruby](/docs/common/install_ruby.html) page for help with Ruby installation. We recommend using rbenv to install a recent version of ruby 1.9.3 (e.g. 1.9.3-p545) as we have found the versions available using apt-get on Ubuntu are out of date and don't work well. 

3. Install the BOSH deployer Ruby gem.

        gem install bosh_cli --pre --no-ri --no-rdoc
        gem install bosh_cli_plugin_micro --pre --no-ri --no-rdoc

Once you have installed the deployer, you will be able to use `bosh micro` commands. To see help on these type `bosh help micro`

## <a id="next-step"></a> Next Step ##
When you're finished setting up, move on to the next step - [deploying Micro BOSH](deploying_micro_bosh.html)
