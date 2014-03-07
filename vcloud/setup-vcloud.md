---
title: Set up vCloud Virtual Data Center Resources
---

If using vCloud Hybrid Service you can access the vCloud Director user interface for the virtual datacenter from the vCHS portal. Once you are in the vCloud Director user interface you can follow the instructions below.

If using vCloud Director, simply log into the user interface.

## <a id="catalog"></a>Add a catalog ##

+ Add a catalog in vCloud Director where stemcells and media (ISOs) for BOSH will be stored.

![vcloud_catalog](/images/vcloud_catalog.png)

## <a id="network"></a>Add a network ##

+ Add a network to the virtual datacenter in vCloud Director. Here are the major steps to create a network from vCloud web portal:
	1. Click the Manage & Monitor tab and click Organization vDCs in the left pane.
	2. Double-click the organization vDC name to open the organization vDC.
	3. Click the Org vDC Networks tab and click Add Network.
	4. Then choose to add an external direct network or an external routed network. For security and better usage of IP resources, the external routed network is recommended. For adding routed network detailed steps, please refer this link: 
 [Create an External Routed Organization vDC Network](http://pubs.vmware.com/vcd-51/index.jsp#com.vmware.vcloud.admin.doc_51/GUID-6E69AF88-31E0-4DD8-A79E-E8E4B6F68878.html).
 	5. To allow machines on the routed network to talk outside the network, e.g. the micro BOSH, [configure a source NAT rule on the network](http://pubs.vmware.com/vcd-51/index.jsp?topic=%2Fcom.vmware.vcloud.admin.doc_51%2FGUID-464E27A8-3238-4553-ABCF-77808D3A510D.html).

	![vcloud_source_nat](/images/vcloud_source_nat.png)
 
 
