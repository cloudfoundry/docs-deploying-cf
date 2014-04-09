---
title: Set up vCHS or vCloud Virtual Data Center Resources
---


## <a id="prerequisites"></a>Prerequisites ##

### vCloud Hybrid Service ###
To get started with Cloud Foundry on **vCloud Hybrid Service** you will need a **vCloud Hybrid Service account** and a **virtual datacenter** (vDC) with at least 40GB memory, 120GB disk and 28 vCPU cores. You will need at least two public IP addresses, one to connect to the jump box VM and one for Cloud Foundry.

Your user account will need Virtual Infrastructure Administrator and Network Administrator privileges for this virtual datacenter.

A lot of the actions we take during this installation will be carried out in the vCloud Director UI. When using vCHS, you can open the **vCloud Director UI** for your virtual datacenter as follows:

Click on your virtual datacenter to open the virtual datacenter details.
![vCHS Dashboard](/vcloud_images/vchs_dashboard.png)

Next click on  "Manage Catalogs in vCloud Director" to open the vCloud Director UI.
![vCHS vDC](/vcloud_images/vchs_vdc_vcdui.png)


### vCloud Director ###
To get started with Cloud Foundry on **vCloud Director** you will need an **account** in a [vCloud organization](http://pubs.vmware.com/vcd-51/topic/com.vmware.vcloud.users.doc_51/GUID-B2D21D95-B37F-4339-9887-F7788D397FD8.html) and a vCloud **virtual datacenter** with an Internet routable network and a block of assigned IP addresses. The recommended resources for a full CF deployment introduced in this doc is to have 28 CPU cores, 39 GB Memory, 120 GB disk, 23 IPs to assign.

Your user account will need [organization administrator](http://pubs.vmware.com/vcd-51/topic/com.vmware.vcloud.users.doc_51/GUID-5B60A9C0-612A-4A3A-9ECE-694C40272505.html) privileges for this virtual datacenter.

## <a id="catalog"></a>Add a catalog ##

In the vCloud Director UI, choose "Catalogs" on the top row of buttons, then "My Organization's Catalogs".

Add a catalog in vCloud Director where stemcells and media (ISOs) for BOSH will be stored.
Next click on  "Manage Catalogs in vCloud Director" to open the vCloud Director UI.

![vCloud Catalog](/vcloud_images/vcloud_catalog.png)

## <a id="network"></a>Set up networking ##

Please note that the following instructions are optimized for simplicity rather than security. When planning a production deployment of Cloud Foundry you will need to modify these instructions to meet the needs of your service.

### Create a network ###

If you are using vCHS you can reuse the networks provided out of the box. Typically you will use the routed network - for example, `164-935-default-routed`. Note that the default network names are actually all lower case, despite what is shown in the UI.

If you want to add a network in vCHS, or if you need to in vCloud Director, here are the major steps to create a network from the vCloud Director UI:

1. Click the Manage & Monitor tab and click Organization vDCs in the left pane.
2. Double-click the organization vDC name to open the organization vDC.
3. Click the Org vDC Networks tab and click Add Network.
4. Then choose to add an external direct network or an external routed network. For security and better usage of IP resources, the external routed network is recommended. For adding routed network detailed steps, please refer this link:
 [Create an External Routed Organization vDC Network](http://pubs.vmware.com/vcd-51/index.jsp#com.vmware.vcloud.admin.doc_51/GUID-6E69AF88-31E0-4DD8-A79E-E8E4B6F68878.html).


### Plan your IP address allocation ###

You will be using the private network created above for your deployment. You should plan the use of your **private IP addresses** as follows:
* Assign a private IP for the jump box
* Assign a second IP for Micro BOSH
* Assign a range of at least 20 IPs use for static IP assignments to VMs in the Cloud Foundry deployment
* Assign one of the IPs in this static range for HA Proxy
* Allocate the same number again to be used dynamically during deployment.

The exact number of addresses needed for the static and dynamic ranges will depend on the size of your deployment. The basic deployment described in this documentation will use 1 jump box VM, 1 micro BOSH VM and ~16 Cloud Foundry VMs.

You will need two **public IP addresses** for this deployment. These are used as follows:
* One for connectivity to the jump box
* One for all user connectivity to the Cloud Foundry instance


### Set up SSH in to the jump box ###

In the vCloud Director UI go to the `Administration` button on the top row, then `Cloud Resources`, then `Virtual Datacenters`, then pick your virtual datacenter. Choose the `Edge Gateways` tab, select the gateway being used and click on the gear to select `Edge gateway services`.
![vCloud Edge Gateway](/vcloud_images/vcloud_edge_gateway.png)

* Create a DNAT rule allowing inbound connections from your jump box public IP address to the jump box private IP address on port 22, to allow SSH from your laptop / desktop to the jump box.
* Create a firewall rule allowing inbound traffic from this public IP to the jump box private IP.

### Set up connectivity in to the Cloud Foundry instance ###

Using very similar steps:

* Create a DNAT rule allowing inbound connections from your public Cloud Foundry instance IP to the HA Proxy private IP address. You can allow all ports, or restrict it to ports 80 (HTTP) and 443 (HTTPS).
* Create a firewall rule allowing inbound traffic from this public IP to the HA Proxy private IP


### Connect outbound to the internet ###

We allow all private network VMs access to the internet to allow installation of software, connection to the vCloud API, connectivity between VMs, etc.

  * You will need to configure an SNAT rule allowing outbound connections to a public IP. For simplicity you should do this for the entire network to allow all outbound connections. You can reuse the same IP as used for inbound SSH above. More information is available in the [vCloud Director documentation](http://pubs.vmware.com/vcd-51/index.jsp?topic=%2Fcom.vmware.vcloud.admin.doc_51%2FGUID-464E27A8-3238-4553-ABCF-77808D3A510D.html)
  * You will need to set up a firewall rule allowing outbound traffic from all private / internal addresses to all external addresses.

![vcloud_source_nat](/vcloud_images/vcloud_source_nat.png)


## <a id="jumpbox"></a>Set up a jump box

Cloud Foundry installation on vCHS / vCloud Director is simplest if you create a machine within the target vDC (a **jump box** VM) where you drive the rest of the installation process. We typically use Ubuntu 12.04 for this machine, and configure it with 1GB memory, 1GHz vCPU and 16GB storage (You will need to have around 8 GB of free disk space for the bosh_cli if you plan to use it to deploy CF releases. You'll need around 3 GB free disk space in /tmp)

The rest of this documentation assumes you have created this machine in the target vDC.

1. Download the latest Ubuntu 12.04 LTS server ISO image from http://www.ubuntu.com/download/server
2. Upload this ISO to the catalog you created above. Choose the "Media and Other" tab, and then use the upload button.
3. In your virtual datacenter, create a VM connecting the ISO to the VM. Start up the VM and install Ubuntu.
4. Once it starts, assign an IP address to the jump box by logging into the VM and assigning a static IP address using [Ubuntu's instructions](https://help.ubuntu.com/10.04/serverguide/network-configuration.html#static-ip-addressing).
5. Install a set of packages we will need for various purposes during this installation

		sudo apt-get -y install libsqlite3-dev genisoimage libxslt-dev libxml2-dev whois openssl bind9


## <a id="bosh"></a> Install BOSH CLI

The following steps will help you to install the BOSH command line interface (CLI) on the jump box VM:

1. Make sure you have installed the core Ubuntu packages that the BOSH deployer depends on - see the [section above](#jumpbox)

2. Install Ruby and RubyGems. We recommend using rbenv to install a recent version of ruby 1.9.3 (e.g. 1.9.3-p545) as we have found the versions available using apt-get on Ubuntu are out of date and don't work well.

3. Install the BOSH deployer Ruby gem.

        gem install bosh_cli --pre --no-ri --no-rdoc
        gem install bosh_cli_plugin_micro --pre --no-ri --no-rdoc

Once you have installed the deployer, you will be able to use `bosh micro` commands. To see help on these type `bosh help micro`

## <a id="dns"></a> DNS and public IPs

### Public IPs

This deployment makes use of two public IP addresses, used as follows:

* One to access the jump box
* One for all access to the Cloud Foundry instance

### Wildcard domains

In general we use a wildcard domain to map all apps & services running on a Cloud Foundry instance to it's public IP address (i.e. the public IP address of HA Proxy in the deployment described in this documentation, or the IP address of a load balancer if that's being used). For example, if a Cloud Foundry instance is using the wildcard domain `*.mycf.com`, and the application foo is deployed to it, the URL `foo.mycf.com` can be used to access the foo application.

The system services are made available in the same way - `api.mycf.com`, `console.mycf.com` etc may all be used. This can lead to some phishing opportunities, however. For example, someone could pretend to be a legitimate system service and create `change_your_password.mycf.com` or `sign_up_with_credit_card.mycf.com`. To avoid this, Cloud Foundry supports two domains, one for system applications and one for general applications. Only administrators of the system can deploy applications to the system domain.

These instructions explain a deployment using a single wildcard domain for both applications and system services. It's is straightforward to use separate domains if you wish, by modifying the Cloud Foundry manifest file used in [Deploying Cloud Foundry](deploy_cf.html).

### Private network deployments and DNS

When the internal components of Cloud Foundry talk to each other over http (e.g. login server to the UAA) they use the same wildcard DNS name to resolve host names to IPs as external applications, as described above. As a result, when `login.mycf.com` wants to connect to `uaa.mycf.com` and does a DNS lookup, it will receive the public IP address of the deployed HA Proxy in this deployment. Since the public IP is simply NAT'd to a private IP in this deployment, this requires the network setup to allow for NATing from the private network IP to a public IP and then back to a private IP. In carrying out this deployment we have found this both tricky and inefficient.

To simplify this we recommend that, when deploying a CF instance to a private network, VMs on this network resolve host names for other internal services to private IP addresses rather than public IPs. In the example above, when the login server tries to connect to the UAA service on `uaa.mycf.com` it will get the private network IP address for HA Proxy rather than the public IP.

Doing this requires setting up a **private DNS server** within the private network for VMs on that network.

For this deployment, we suggest using the jump box to support this DNS Server. We installed the `bind2` package on the jump box earlier to provide internal DNS support.

You can configure the package as follows:

In /etc/bind, modify the file `named.conf.options` to include `forwarders`. For example, the version of the file used in developing these docs looks as follows:

	options {
	        directory "/var/cache/bind";

	        forwarders {
	        8.8.8.8;
	        8.8.4.4;
	        };

	        dnssec-validation auto;

	        auth-nxdomain no;    # conform to RFC1035
	        listen-on-v6 { any; };
	};

Modify the file `named.conf.local` to pull in a zone configuration file. Replace <public IP> below with the public IP address being used to access your deployment.

	zone "<public IP>.xip.io" IN {
	 type master;
	 file "/etc/bind/zones/<public IP>.xip.io.db";
	};

Finally, create a file in `/etc/bind/zones/<public IP>.xip.io"` as follows

	<public IP>.xip.io. 3600 IN SOA ns.<public IP>.xip.io. hostmaster.<public IP>.xip.io. (
	 2014032701 ; serial
	 8H ; refresh
	 4H ; retry
	 4W ; expire
	 1D ; minimum
	)

	$ORIGIN <public IP>.xip.io.  ; Set the $ORIGIN domain

	@ IN NS ns.<public IP>.xip.io. ; Set the nameserver to this machine
	ns IN A 192.168.109.15

	* IN A 192.168.109.119           ; Set the wildcard domain to the HA Proxy internal IP

Once you create / change these files you will need to restart the DNS server. This can be done with the command `service bind9 restart`.

*Thanks to [Bobby Allen's blog](http://blog.bobbyallen.me/2013/09/19/setting-up-internal-dns-on-ubuntu-server-12-04-lts/) for the help in figuring this out.*


## <a id="next-step"></a> Next Step ##
When you're finished setting up, move on to the next step - [deploying Micro BOSH](deploying_micro_bosh.html)
