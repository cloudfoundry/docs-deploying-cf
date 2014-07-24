---
title: Install and Prepare vSphere Cluster
---

Before starting a Cloud Foundry deployment, a vSphere cluster must be set up. This guide will use a minimal configuration to setup the cluster:

1. Two servers to install ESXi with `X` core processors and `Y` GB RAM, where `X` and `Y` depend on the hardware configuration selected.
2. One server to install vCenter. This can be a VM in any of the ESXi servers.
3. A storage server. SAN is recommended, but you can also use other storages servers
4. A Switch
5. Network: IP ranges at least 100 IPs

## <a id="install"></a>Install ESXi and vCenter ##

Follow the standard ESXi and vCenter installation process. After installation your ESXi will look like the image below:

![esxi](/images/esxi5.png)

## <a id="prepare"></a>Prepare vCenter for Cloud Foundry Deployment ##

### <a id="prepare-datacenter"></a>Create a Datacenter ###

In vCenter, go to `Hosts and Clusters` then click on `Create a Datacenter`. A new datacenter will be created in the left panel. Name your new datacenter.

![datacenter](/images/datacenter.png)

### <a id="prepare-cluster"></a>Create a Cluster ###

Create a cluster and add ESXi hosts to the cluster.

1. Select the datacenter created in the step above.
2. Click on the `Create a cluster` link.
3. The `New Cluster Wizard` will open. Name your cluster, then click **Next** and follow the instructions in the wizard.

Once finished, the new cluster can be seen in the left panel.

![cluster1](/images/cluster1.png)

### <a id="prepare-pool"></a>Create a Resource Pool ###

Create a resource pool.

### <a id="prepare-hosts"></a>Add the ESXi hosts to the cluster ###

1. Select the cluster created in the step above.
2. Click on the `Add a Host` link.
3. The `Add Host Wizard` will open. Provide the IP address/hostname and login credentials for the ESXi host. Click **Next** and follow the instructions in the wizard.

Once finished, the newly added host can be seen in the left panel.

![host1](/images/add_host.png)

## <a id="folders"></a>Create the Folders for VMs, Templates, and Disk Path ##

MicroBOSH and BOSH use predefined locations for VMs, templates, and disk path. These locations are defined in the deployment manifest.

### <a id="folders-vm"></a>Create the VM and Templates Folder ###

1. Click **Inventory**. Click **Select VMs and Templates**.
2. Select the datacenter created in the step above.
3. Click **New Folder** on the top of the left panel to create new folders. Create the following four folders:
    * `MicroBOSH_VMs` for MicroBOSH
    * `MicroBOSH_Templates` for MicroBOSH
    * `CF_VMs` for BOSH
    * `CF_Templates` for BOSH

![vms_and_folders](/images/vms_templates.png)

### <a id="folders-disk"></a>Create the Disk Path ###

1. Click **Inventory**. Click **Datastore and Datastore Clusters**.
2. Right-click the datastore where you want to store VM disks. Select `Browse Datastore`. The datastore opens in a new window.
3. Click **New Folder** on the top of the left panel to create new folders. Create the following two folders:
    * `MicroBOSH_Disks` for MicroBOSH
    * `CF_Disks` for BOSH

![datastore1](/images/datastore.png)

## <a id="summary"></a>Summary ##

The vSphere 5.x cluster is ready for Cloud Foundry deployment.