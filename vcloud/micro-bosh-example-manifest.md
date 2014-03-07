---
title: BOSH Example Manifest
---

This is an example manifest for deploying Micro BOSH on vCloud. 

    ---
    name: bosh_micro_wdc_vcd-2

    logging:
      level: DEBUG

    network:
      ip: 10.146.21.150
      netmask: 255.255.255.128
      gateway: 10.146.21.253
      dns:
      - 10.132.71.1
      - 10.132.71.2
      cloud_properties:
        name: "vdc_network"

    resources:
      persistent_disk: 16384

    cloud:
      plugin: vcloud
      properties:
        agent:
          ntp:
          - us.pool.ntp.org
        vcds:
          - url: <your_vcd_endpoint>
            user: <username>
            password: <password>
            entities:
              organization: <Organization name>
              virtual_datacenter: <VDC name>
              vapp_catalog: <Organization catalog name>
              media_catalog: <Organization catalog name> 
              media_storage_profile: <Storage proflie>
              vm_metadata_key: <Metadata associated with Cloud Foundry VMs>
              description: <Text associated with Cloud Foundry VMs> 
    
    env: 
      vapp: micro_bosh_vapp 

