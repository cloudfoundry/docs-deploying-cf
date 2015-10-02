---
title: Deploying Cloud Foundry using BOSH Lite
---

This topic describes the process for deploying Cloud Foundry to a BOSH Lite environment.

##<a id="prerequisites"></a>Prerequisites ##

You must have your BOSH Lite VM up and running, and have targeted the BOSH Lite Director with the BOSH CLI. For instructions, consult the [BOSH Lite repository](https://github.com/cloudfoundry/bosh-lite).

##<a id="stemcell"></a>Upload a Stemcell ##

1. Open [https://bosh.io/stemcells](https://bosh.io/stemcells) in a web browser to view a list of publicly available BOSH stemcells. The list displays the most recent build numbers of BOSH stemcells, organized by operating system, target IaaS, and hypervisor.

1. Choose a BOSH stemcell for "BOSH Lite Warden" and click the build number to download.

1. In a terminal window, run `bosh upload stemcell STEMCELL-NAME` to upload the
stemcell to the BOSH Director.

    <pre class="terminal">
    $ bosh upload stemcell bosh-stemcell-2776-warden-boshlite-ubuntu-trusty-go_agent.tgz
    </pre>

##<a id="create-stub"></a>Create a Deployment Manifest Stub for AWS ##

If you deploy BOSH Lite to AWS, you must specify a system domain in a manifest stub. If you are deploying to a local VM this step is not necessary.

1. By default, BOSH Lite uses the system domain `bosh-lite.com`, which resolves to the local host address `10.244.0.34`. Update the system domain property to one of the following two values:
    - The domain that you configured to point to the public IP address of the BOSH Lite box that you just created.
    - The `xip.io` domain the corresponds to the public IP address given to your BOSH Lite vagrant box: `YOUR_PUBLIC_IP.xip.io`. For more information about xip.io, see [http://xip.io/](http://xip.io/).

1. Create a manifest stub file and add the BOSH Director UUID and system domain as follows:

    ```
    ---
    properties:
      domain: bosh-lite.com # Replace with your system domain
    ```

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

1. Run `bosh target DIRECTOR_ADDRESS_OR_ALIAS` to target the BOSH Director.  The default Director IP when deploying BOSH Lite using the `virtualbox` Vagrant provider is `192.168.50.4`.  If you have overriden the IP, deployed using the `aws` provider, or have already [set an alias for your target](https://bosh.io/docs/sysadmin-commands.html#director), you will use a different value.

    <pre class="terminal">
    $ bosh target 192.168.50.4
    Current target is https://192.168.50.4:25555 (Bosh Lite Director)
    </pre>

1. Use the `scripts/generate-bosh-lite-dev-manifest` command to create a deployment manifest and set it as the current BOSH deployment. 

    <pre class="terminal">
    $ ./scripts/generate-bosh-lite-dev-manifest
    </pre>

    If you have deployed your bosh-lite to AWS you must pass in the stub you [created earlier](#create-stub):

    <pre class="terminal">
    $ ./scripts/generate-bosh-lite-dev-manifest PATH-TO-MANIFEST-STUB
    </pre>

1. Run `bosh create release` to create a Cloud Foundry release.
This command prompts you for a development release name.

    <pre class="terminal">
    $ bosh create release
    </pre>

1. Run `bosh upload release` to upload the generated release to the BOSH
Director.

    <pre class="terminal">
    $ bosh upload release
    </pre>

1. Run `bosh deploy` to deploy the uploaded Cloud Foundry release.

    <pre class="terminal">
    $ bosh deploy
    </pre>

## <a id="verify"></a>Verify the Deployment ##

Install the [Cloud Foundry CLI](https://github.com/cloudfoundry/cli) and run the following commands to verify your deployment:

<pre class="terminal">
$ cf api --skip-ssl-validation https://api.bosh-lite.com  #  If deploying BOSH-Lite to AWS, use 
$                                                         #  https://api.BOSH-LITE-PUBLIC-IP.xip.io instead
$ cf auth admin admin
$ cf create-org test-org
$ cf target -o test-org
$ cf create-space test-space
$ cf target -s test-space
</pre>

If you are behind a proxy, you may need to add `bosh-lite.com` to your `no_proxy` environment variable.

You can now run commands such as `cf push`.

If your Cloud Foundry deployment needs to go through an HTTP proxy to reach the Internet, specify `http_proxy`, `https_proxy` and `no_proxy` environment variables using `cf set-env` or add them to the `env:` section of your application's `manifest.yml`. This ensures the buildpacks can download required libraries, gems, and other files during application staging and execution.

##<a id="update-cf"></a>Update Cloud Foundry##

* If you make change to your manifest, run `bosh deploy` to update your Cloud Foundry deployment with these changes.

* If you make changes to the `cf-release` directory, run `bosh create release && bosh upload release && bosh deploy` to update your Cloud Foundry deployment with these changes.
