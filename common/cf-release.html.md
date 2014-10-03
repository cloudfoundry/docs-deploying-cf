---
title: Using the Latest CF-Release
---

[CF-Release](https://github.com/cloudfoundry/cf-release) is the BOSH [release](/bosh/reference/index.html#bosh-release) repository for Cloud Foundry. Use this with a custom manifest for your environment to deploy Cloud Foundry.

This topic describes how you can create a Cloud Foundry release ready to deploy to your environment after [bootstrapping BOSH](/deploying/).

<p class="note"><strong>Note</strong>: These instructions are for v170 release of Cloud Foundry. We strongly recommend using the <a href="https://github.com/cloudfoundry/cf-release/releases">highest final version tag</a> of cf-release, though this may require that you modify the deployment manifest.</p>

## <a id='clone'></a> Clone CF-Release ##

Create a folder for your clone of the CF-Release repository and clone the repo from Github:

<pre class="terminal">
$ mkdir -p ~/bosh-workspace/releases
$ cd ~/bosh-workspace/releases
$ git clone https://github.com/cloudfoundry/cf-release.git
$ cd cf-release
</pre>

## <a id='upload-the-release'></a> Upload the Release ##

Releases of Cloud Foundry are published regularly.
Use `bosh upload release RELEASE-NAME` to upload a release to your BOSH
deployment. Replace RELEASE-NAME with the version of the cf-release YAML file of
your choice.
We recommend you use the [highest final version tag of cf-release](https://github.com/cloudfoundry/cf-release/releases).

Run the following command to upload a release `.yml` file:

<pre class="terminal">
$ bosh upload release releases/cf-170.yml

Copying packages
----------------
rootfs_lucid64 (1)            FOUND LOCAL
...
Copying jobs
------------
saml_login (4)                FOUND LOCAL
...
Building tarball
----------------
Generated /tmp/d20130829-912-fq3kkd/d20130829-912-1vco0hv/release.tgz
Release size: 1.0G
...
Release uploaded
</pre>

Run the following command to check that you have successfully added the release:

<pre class="terminal">
$ bosh releases

+------+----------+-------------+
| Name | Versions | Commit Hash |
+------+----------+-------------+
| cf   | 170      | 121623ca    |
+------+----------+-------------+
</pre>

You can now deploy a release using a deployment manifest file.