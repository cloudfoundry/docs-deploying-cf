# Running Cloud Foundry Locally

If you want to run an instance of Cloud Foundry on a single
machine or VM, the only supported mechanism is with
[BOSH Lite](https://github.com/cloudfoundry/bosh-lite).

BOSH Lite uses Warden containers instead of VMs, allowing you
to deploy an instance of Cloud Foundry in a comparatively
small footprint.