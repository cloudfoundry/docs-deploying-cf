To generate a deployment manifest, perform the following steps:


1. Clone the cf-release GitHub repository. Use `git clone` to copy the latest Cloud Foundry configuration files onto your machine.

    <pre class="terminal">
    $ git clone https://github.com/cloudfoundry/cf-release.git
    </pre>

1. From the `cf-release` directory, run the update script to fetch all the submodules.

    <pre class="terminal">
    $ cd cf-release
    $ ./scripts/update
    </pre>
<p class="note"><strong>Note</strong>: Ensure that you have the most up-to-date version of the Cloud Foundry code and all required submodules.</p>

1. Install [spiff](https://github.com/cloudfoundry-incubator/spiff), a command line tool for generating deployment manifests.

1. Run the following command from the `cf-release` directory to create a deployment manifest named `cf-deployment.yml`:
    <pre class="terminal">
    $ ./scripts/generate\_deployment\_manifest IAAS PATH-TO-MANIFEST-STUB > cf-deployment.yml
    </pre>
    * Replace `IAAS` with `aws`, `openstack`, or `vsphere`. Use `vsphere` for vSphere, vCloud Air, and vCloud Director. 
    * Replace `PATH-TO-MANIFEST-STUB` with the location of your `cf-stub.yml` file. 

    <p class="note"><strong>Note</strong>: The <code>generate_deployment_manifest</code> script can accept multiple stub files. For example, the following command passes two stub files to the script: <br/>
    <code>./scripts/generate_deployment_manifest vsphere cf-stub.yml cf-consul.yml > cf-deployment.yml
    </code></p>

1. Use the `bosh deployment` command to set your deployment to the generated manifest:

    <pre class="terminal">
    $ bosh deployment cf-deployment.yml
    </pre>

Now you are ready to deploy Cloud Foundry. See the [Deploying Cloud Foundry](../common/deploy.html) topic for instructions.
