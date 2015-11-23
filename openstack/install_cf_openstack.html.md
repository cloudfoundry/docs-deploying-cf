---
title: Deploying Cloud Foundry on OpenStack using BOSH
---

This topic describes the process for deploying Cloud Foundry to an OpenStack
environment using BOSH.

<p class="note"><strong>Note</strong>: Run all the commands in this topic from the <code>~/deployments</code> directory that you created in the <a href="../../bosh/init-openstack.html">Initializing a BOSH environment on OpenStack</a> topic.</p>

##<a id="prerequisites"></a>Prerequisites ##

To deploy Cloud Foundry to OpenStack, you must first complete the following
steps:

* Install the [BOSH Command Line Interface (CLI)](../../bosh/bosh-cli.html).
* Confirm that your OpenStack instance includes four flavors that correspond
with the OpenStack default flavor names `m1.small`, `m1.medium`, `m1.large`, and
`m1.xlarge`.
These flavors must have the following listed specifications, at minimum:
    <table class="nice">
      <tr>
	    <th>OpenStack Flavor Name</th>
	    <th>CPUs</th>
	    <th>RAM (GB)</th>
	    <th>Disk (GB)</th>
	    <th>Ephemeral Disk (GB)</th>
      </tr>
      <tr>
	    <td>m1.small</td>
	    <td>1</td>
	    <td>2</td>
	    <td>10</td>
	    <td>20</td>
	  </tr>
	  <tr>
        <td>m1.medium</td>
        <td>2</td>
        <td>4</td>
        <td>10</td>
        <td>40</td>
      </tr>
      <tr>
        <td>m1.large</td>
	    <td>4</td>
	    <td>8</td>
	    <td>10</td>
	    <td>80</td>
	  </tr>
      <tr>
        <td>m1.xlarge</td>
	    <td>8</td>
	    <td>16</td>
	    <td>10</td>
	    <td>160</td>
	  </tr>
    </table>

* [Validate your OpenStack Instance](validate_openstack.html).
* Provision an IP address and set your DNS to map `*` records to this IP
	address.
	For example, if you use `mycloud.com` domain as the base domain for
	your Cloud Foundry deployment, set a `*` A record for this zone mapping to
	the IP address that you provision.
* Create an OpenStack security group named `cf`.
	Configure the `cf` security group as follows:
	* Open all ports of the VMs in the `cf` security group to all other VMs in
		the `cf` security group. This allows the VMs in the `cf` security group
		to communicate with each other.
    * Open port 22 for administrator SSH access.
	* Open port 80 for HTTP traffic.

##<a id="target"></a>Target the BOSH Director ##

Use the `bosh target` command with the IP address of the BOSH Director to
connect to the BOSH Director.
Log in with the default user name and password, `admin` and `admin`, or use the
user name and password that you set when you installed BOSH.

<pre class="terminal">
$ bosh target https://192.168.109.81:25555
Target set to `bosh'
Your username: admin
Enter password: *****
Logged in as 'admin'
</pre>

##<a id="uuid"></a>Record the BOSH Director UUID ##

Use the `bosh status` command to view information about your BOSH deployment.
Record the UUID of the BOSH Director. You use the UUID when [Customizing the Cloud Foundry Deployment Manifest Stub for OpenStack](cf-stub-openstack.html).

<pre class="terminal">
$ bosh status
Config
             /home/me/.bosh_config

Director
  Name       bosh_openstack
  URL        https://192.168.109.81:25555
  Version    1.2710.0 (00000000)
  User       admin
  UUID       abcdef12-3456-7890-abcd-ef1234567890
  CPI        openstack
  dns        enabled (domain_name: bosh)
  compiled_package_cache disabled
  snapshots  disabled

Deployment
  not set
</pre>

##<a id="stemcell"></a>Upload a Stemcell ##

1. Open [https://bosh.io/stemcells](https://bosh.io/stemcells) in a web browser
to view a list of publicly available BOSH stemcells.
The list displays the most recent build numbers of BOSH stemcells, organized by operating system, target IaaS, and hypervisor.

1. Choose a BOSH stemcell for OpenStack and click the build number to download.

1. In a terminal window, run `bosh upload stemcell STEMCELL-NAME` to upload the
stemcell to the BOSH Director.

    <pre class="terminal">
    $ bosh upload stemcell ./bosh-stemcell-2754-openstack-kvm-ubuntu-trusty-go_agent.tgz
    </pre>

##<a id="create-stub"></a>Create a Deployment Manifest Stub ##

Create a manifest stub file named `cf-stub.yml`.
[Customize the manifest stub](cf-stub-openstack.html) for your environment.

##<a id="deploy-cf"></a>Deploy Cloud Foundry ##

1. Clone the `cf-release` GitHub repository.

    <pre class="terminal">
    $ git clone https://github.com/cloudfoundry/cf-release.git
    </pre>

1. Use the `scripts/update` helper to update the `cf-release` submodules.

    <pre class="terminal">
    $ cd cf-release
    $ ./scripts/update
    </pre>

1. Install [spiff](https://github.com/cloudfoundry-incubator/spiff).

1. Run the following spiff command from the `cf-release` directory to create a deployment manifest named `cf-deployment.yml`:

    `scripts/generate_deployment_manifest INFRASTRUCTURE MANIFEST-STUB > cf-deployment.yml`

    Replace INFRASTRUCTURE with `openstack` and replace MANIFEST-STUB with the name and location of your `cf-stub.yml file`. For example:

    <pre class="terminal">
	$ ./scripts/generate_deployment_manifest openstack cf-stub.yml > cf-deployment.yml
    </pre>

    <p class="note"><strong>Note</strong>: <code>scripts/generate_deployment_manifest</code> can accept a list of stub files. For example: <code>scripts/generate_deployment_manifest openstack cf-stub.yml cf-consul.yml > cf-dm.yml</code></p>

    <p class="note"><strong>Note</strong>: The <code>cf-deployment.yml</code> uses the default OpenStack flavor names of <code>m1.small</code>, <code>m1.medium</code>, <code>m1.large</code>, and <code>m1.xlarge</code>. If your OpenStack instance uses alternate names for these flavors, change the default flavor names in <code>cf-deployment.yml</code> to match the alternate names that your instance uses.</p>

1. Use `bosh target` to target the BOSH Director.

    <pre class="terminal">
    $ bosh target
	Current target is https://192.168.109.81:25555
    </pre>

1. Set your deployment to the generated manifest.

    <pre class="terminal">
    $ bosh deployment cf-deployment.yml
    </pre>

1. Use `bosh create release` to create a Cloud Foundry release.
This command prompts you for a development release name.

    <pre class="terminal">
    $ bosh create release
    </pre>

1. Use `bosh upload release` to upload the generated release to the BOSH
Director.

    <pre class="terminal">
    $ bosh upload release
    </pre>

1. Deploy the uploaded Cloud Foundry release.

    <pre class="terminal">
    $ bosh deploy
    </pre>

    <p class="note"><strong>Note</strong>: <code>bosh deploy</code> can take 2-3 hours to complete.</p>

1. Use `curl` to test the API endpoint of your Cloud Foundry installation.

    <pre class="terminal">
    $ curl api.subdomain.domain/info
    </pre>

    If `curl` succeeds, it should return the JSON-formatted information.
	If `curl` does not succeeds, check your networking and make sure your domain
	has an NS record for your subdomain.

1. You should be able to target your Cloud Foundry installation with the [cf Command Line Interface (CLI)](/devguide/installcf/index.html) and log in as an
administrator.

    The user name, `admin` and the password, `fakepassword`, are specified in
    the deployment manifest under **uaa:scim:users**.

    For more information about managing organizations, spaces, users, and
    applications, refer to the [cf CLI](/devguide/installcf/index.html) topic.

##<a id="update-cf"></a>Update Cloud Foundry##

* If you make change to your manifest, run `bosh deploy` to update your Cloud
Foundry deployment with these changes.

* If you make changes to the `cf-release` directory, run `bosh create release && bosh upload release && bosh deploy` to update your Cloud Foundry deployment with these changes.

## <a id="verify"></a>Verify the Deployment ##

1. Run `bosh vms`. This command provides an overview of the virtual machines that BOSH manages as part of the current deployment. The state of every VM should show as **running**.

<pre class="terminal">
	$ bosh vms

	+-----------------------------+---------+------------------+---------------+
	| Job/index                   | State   | Resource Pool    | IPs           |
	+-----------------------------+---------+------------------+---------------+
	| nfs_server/0                | running | nfs_server       | 10.146.21.174 |
	| ccdb/0                      | running | ccdb             | 10.146.21.175 |
	| cloud_controller/0          | running | cloud_controller | 10.146.21.176 |
	| collector/0                 | running | collector        | 10.146.21.178 |
	| health_manager/0            | running | health_manager   | 10.146.21.173 |
	| nats/0                      | running | nats             | 10.146.21.172 |
	| router/0                    | running | router           | 10.146.21.171 |
	| syslog/0                    | running | syslog           | 10.146.21.177 |
	| uaa/0                       | running | uaa              | 10.146.21.180 |
	| uaadb/0                     | running | uaadb            | 10.146.21.179 |
	| dea/0                       | running | dea              | 10.146.21.181 |
	| saml_login/0                | running | saml_login       | 10.146.21.181 |
	+-----------------------------+---------+------------------+---------------+
</pre>
