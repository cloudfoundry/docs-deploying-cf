---
title: Deploying BOSH on AWS
owner: Release Integration
---

In the [previous step](./setup_aws.html) you prepared an AWS environment for BOSH. This document shows how to deploy BOSH into your AWS environment by creating a deployment manifest and passing it to `bosh-init`.

## <a id="copy-keypair"></a>Copy the RSA Private Key
When `bosh aws create` prepared your AWS environment, it generated an RSA private key file and saved it into your `~/.ssh` directory. To deploy BOSH, you need this file in your deployment directory.

Copy the RSA private key file, `id_rsa_bosh`, to the `cf-example` directory you created in the [previous step](./setup_aws.html).

<pre class='terminal'>
$ cp ~/.ssh/id_rsa_bosh ~/deployments/cf-example
</pre>

## <a id="create-manifest"></a>Create a Deployment Manifest

1. Log into the AWS Console.

1. Create a deployment manifest file named `bosh.yml` in the deployment directory. For its contents, copy the YAML code from the [AWS BOSH manifest template](http://bosh.io/docs/init-aws.html#create-manifest) in the BOSH documentation.

1. Replace properties listed in the file as follows:
    * `ELASTIC-IP`:In the **EC2 Dashboard** under **Elastic IPs**, use the Elastic IP with the Instance associated with **bosh**. Example: "203.0.113.126".
    * `SUBNET-ID`: In the **VPC Dashboard** tab **Subnets**, use the Subnet ID for the subnet named "bosh1." Example: "subnet-2f58134a".
    * `AVAILABILITY-ZONE` and `REGION`: Use the values from the `bosh_environment` file you created in the [previous step](./setup_aws.html).
    * `ACCESS-KEY-ID` and `SECRET-ACCESS-KEY`: Use the values from your `bosh_environment` file.
    * Replace all predefined passwords with passwords of your choice.</p>

## <a id="deploy"></a>Deploy BOSH

1. Confirm that [bosh-init](http://bosh.cloudfoundry.org/docs/install-bosh-init.html) and the [BOSH CLI](https://bosh.io/docs/bosh-cli.html) are installed on your machine.

1. Run `bosh-init deploy ./bosh.yml` to start the deployment process.

    <pre class='terminal'>
    $ bosh-init deploy ./bosh.yml
    ...
    </pre>

    See [AWS CPI errors](http://bosh.io/docs/aws-cpi.html#errors) for list of common errors and resolutions.

1. Use `bosh target ELASTIC-IP` to log into your new BOSH Director. The default username and password are `admin` and `admin`.

    <pre class="terminal">
    $ bosh target 198.51.100.129

    Target set to 'bosh'
    Your username: admin
    Enter password: *****
    Logged in as 'admin'
    </pre>

For information about deploying BOSH on AWS without running `bosh aws create`, see the [Initializing BOSH environment on AWS](http://bosh.io/docs/init-aws.html) topic.

