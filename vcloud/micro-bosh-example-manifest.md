---
title: Customizing the MicroBOSH Deployment Manifest for vCloud
---

This topic contains a MicroBOSH deployment manifest template.
Save a copy of this template as a YAML file in your deployment directory.
Customize the contents of this file for your vCloud environment, then use the customized file to deploy MicroBOSH to vCloud.

```yml
name: microbosh

network:
  ip: 192.168.1.10        # Change to the MicroBOSH IP address
  netmask: 255.255.255.0  # Change to your network netmask
  gateway: 192.168.1.1    # Change to your network gateway IP address
  dns:
  - 8.8.8.8               # Change to the DNS server IP address
  - 8.8.4.4               # (Optional) Add a second DNS server IP address
  cloud_properties:
    name: "150-900-default-routed"  # Change to your vDC network name

resources:
  persistent_disk: 16384
  cloud_properties:
    ram: 8192
    disk: 16384
    cpu: 4

cloud:
  plugin: vcloud
  properties:
    agent:
      ntp: # Change to an NTP server for your region
      - us.pool.ntp.org
    vcds:
      - url: https://p2v1-vcd.vchs.vmware.com:443  # Change to the endpoint of the target vCloud director
        user: USERNAME      # Change to user name on the target vCloud director
        password: PASSWORD  # Change to password
        entities:
          organization: 150-900        # Change to your organization name
          virtual_datacenter: 150-900  # Change to your virtual datacenter name
          vapp_catalog: VAPP-CAT       # Change to name of catalog for vapp templates
          media_catalog: MEDIA-CAT     # Change to name of catalog for media files
          media_storage_profile: SSD-Accelerated  # Change to the storage profile to use. Use a * to match all storage profiles
          vm_metadata_key: KEYNAME # Change to the key name of VM metadata associated with Cloud Foundry
          description: TEXT # Change to the text associated with the Cloud Foundry VMs
env:
  vapp: micro_bosh_vapp
  # By default, you can ssh into MicroBOSH with username 'vcap' and password 'c1oudc0w'
  # To change this default password, uncomment the following lines and
  # change PASSWORD to an SHA hash password generated `using mkpasswd -m sha-512`
  #
  #bosh:
  	#password: PASSWORD

logging:
  # Increase the default logging level to trace REST traffic with IaaS providers. Default value is `INFO`
  level: DEBUG
  # Default location for the log: <deployment_dir>/bosh_micro_deploy.log
  # To change this default, uncomment the following line and replace PATHNAME

  # file: <file pathname>
```