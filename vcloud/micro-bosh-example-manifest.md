---
title: BOSH Example Manifest
---

This is an example manifest for deploying Micro BOSH on vCloud. 

    name: bosh_micro_vcloud

    network:
      ip: <IP addr for micro BOSH VM - e.g. 192.168.1.10>
      netmask: <netmask for network - e.g. 255.255.255.0>
      gateway: <gateway for network e.g. 192.168.1.1>
      dns:
      - <IP addr for DNS server. e.g. 8.8.8.8>
      - <IP adds for second DNS server if desired>
      cloud_properties:
        name: "<vDC network name - e.g. 150-900-default-routed >"

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
          ntp:
          - us.pool.ntp.org
        vcds:
          - url: <your_vcd_endpoint e.g. https://p2v1-vcd.vchs.vmware.com:443 - Do **not** include path on host >
            user: <username - e.g. mickey@mouse.com >
            password: <password>
            entities:
              organization: <Organization name - e.g. 150-900 >
              virtual_datacenter: <VDC name - e.g. 150-900 >
              vapp_catalog: <Organization catalog name>
              media_catalog: <Organization catalog name> 
              media_storage_profile: <Storage proflie e.g. SSD-Accelerated >
              vm_metadata_key: <Metadata associated with Cloud Foundry VMs>
              description: <Text associated with Cloud Foundry VMs> 
    
    env: 
      vapp: micro_bosh_vapp

    logging:
      # If needed increase the default logging level to trace REST traffic with IaaS providers. Default is info
      level: DEBUG
      # Default location is <deployment_dir>/bosh_micro_deploy.log. Change this with:
      # file: <file pathname>


 
      

