##Deploying Wordpress Using BOSH##

We have created a sample three-tier application (Nginx, Apache + PHP with WordPress, and MySQL) to demonstrate how you can use BOSH, and the next step is to deploy it using your newly created micro BOSH instance.

##Uploading the Sample Release##

The sample release is on Github for your cloning convenience:

+ First make a git clone of the sample application release repository:

		cd ~
		git clone git://github.com/cloudfoundry/bosh-sample-release.git
		cd bosh-sample-release

+ Upload the release to micro BOSH:

		bosh upload releases/wordpress-1.yml

##Uploading the Latest Stem Cell##

Now we download the latest stem cellto upload to our micro BOSH instance.

   * Download the latest BOSH stem cell for vCloud:

		bosh download public stemcell bosh-stemcell-vsphere-0.6.7.tgz

   * Upload it to your micro BOSH instance:

		bosh upload stemcell bosh-stemcell-vsphere-0.6.7.tgz

##Create a Private Network##

   1. [Add private networks](http://pubs.vmware.com/vcd-51/index.jsp?topic=%2Fcom.vmware.vcloud.admin.doc_51%2FGUID-6E69AF88-31E0-4DD8-A79E-E8E4B6F68878.html) to separate application components from each other and from direct access by users. Here, “cf-net” is a direct network added earlier and “cf-routed” is a private network.
	![vcloud_private_network](../images/vcloud_private_network.png)

   1. To allow machines on the private network to talk outside the network, e.g. the micro BOSH, [configure a source NAT rule on the network](http://www.google.com/url?q=http%3A%2F%2Fpubs.vmware.com%2Fvcd-51%2Findex.jsp%3Ftopic%3D%252Fcom.vmware.vcloud.admin.doc_51%252FGUID-464E27A8-3238-4553-ABCF-77808D3A510D.html&sa=D&sntz=1&usg=AFQjCNGXS8KPBo_PsbMblK3bh835u_FFmg).

	![vcloud_source_nat](../images/vcloud_source_nat.png)

##Create a Deployment Manifest##

   1. Get the director UUID using the following command:

		bosh status

   2. Copy the file [wordpress-vcloud.yml](wordpress-vcloud.yml) in the bosh-sample-release directory and update it to suit your network.


##Deploy##

   1. Select the deployment manifest you just created:

		bosh deployment ~/wordpress-vcloud.yml

   1. Initiate the deployment:

		bosh deploy

   1. Sit back and enjoy the show!

##Connect to the deployed sample application##

Once your deployment is complete point your browser to the IP of the vm where nginx job is running `http://<nginx-vm-staticip>`.

Congratulations. You just used BOSH to deploy an application to vCloud!
