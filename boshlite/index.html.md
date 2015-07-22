---
title: Deploying Locally with BOSH Lite
---

If you want to run an instance of Cloud Foundry on a single
machine or VM, the only supported mechanism is with
[BOSH Lite](https://github.com/cloudfoundry/bosh-lite).

BOSH Lite uses Linux containers instead of VMs, allowing you
to deploy an instance of Cloud Foundry in a comparatively
small footprint.

[BOSH-Lite Setup](https://github.com/cloudfoundry/bosh-lite)

[Deploy Cloud Foundry using BOSH](deploy_cf_boshlite.html)

Troubleshooting:

* For trouble with BOSH-Lite, consult the [BOSH-Lite README](https://github.com/cloudfoundry/bosh-lite/blob/master/README.md) or [open a Github issue](https://github.com/cloudfoundry/bosh-lite/issues)

* For trouble with deploying Cloud Foundry to BOSH-Lite, consult the [cf-release README](https://github.com/cloudfoundry/cf-release/blob/master/README.md) or [open a Github issue](https://github.com/cloudfoundry/cf-release/issues)