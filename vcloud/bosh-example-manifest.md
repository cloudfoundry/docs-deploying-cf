---
title: BOSH Example Manifest
---

This is an example manifest for deploying BOSH via Micro BOSH. The next step would be to deploy Cloud Foundry.

    ---
    name: bosh_wdc_vdc-2
    director_uuid: micro_bosh_director_uuid

    release:
      name: bosh
      version: latest

    networks:
    - name: default # An internal name for the network in your manifest file
      subnets:
      - reserved:
        - 10.146.21.129 - 10.146.21.150
        - 10.146.21.210 - 10.146.21.252
        static:
        - 10.146.21.151 - 10.146.21.189
        range: 10.146.21.128/25
        gateway: 10.146.21.253
        dns:
        - 10.132.71.1
        - 10.132.71.2
        cloud_properties:
          name: "vdc_network"

    #
    # You shouldn't have to change any of the resource pool parameters
    #

    resource_pools:
    - name: small
      stemcell:
        name: bosh-vcloud-esxi-ubuntu
        version: 1868
      network: default
      size: 5
      cloud_properties:
        ram: 512
        disk: 2048
        cpu: 1
      env:
        vapp: bosh_vapp

    - name: director
      stemcell:
        name: bosh-vcloud-esxi-ubuntu
        version: 1868
      network: default
      size: 1
      cloud_properties:
        ram:  2048
        disk: 8192
        cpu: 2
      env:
        vapp: bosh_vapp

    compilation:
      workers: 4
      network: default
      cloud_properties:
        ram: 2048
        disk: 4048
        cpu: 2

    #
    # If you have errors with canary instances during a deployment, you can try
    # increasing the canary_watch_time and update_watch_time. The values here are in
    # milliseconds. Divide by 1000 to get the value in seconds.
    #
    #

    update:
      canaries: 1
      canary_watch_time: 60000
      update_watch_time: 60000
      max_in_flight: 1
      max_errors: 2

    jobs:

    - name: nats
      template: nats
      instances: 1
      resource_pool: small
      networks:
      - name: default
        static_ips:
        - 10.146.21.157

    - name: postgres
      template: postgres
      instances: 1
      resource_pool: small
      persistent_disk: 2048
      networks:
      - name: default
        static_ips:
        - 10.146.21.151

    - name: redis
      template: redis
      instances: 1
      resource_pool: small
      networks:
      - name: default
        static_ips:
        - 10.146.21.152

    - name: director
      template: director
      instances: 1
      resource_pool: director
      persistent_disk: 2048
      networks:
      - name: default
        static_ips:
        - 10.146.21.153

    - name: blobstore
      template: blobstore
      instances: 1
      resource_pool: small
      persistent_disk: 8192
      networks:
      - name: default
        static_ips:
        - 10.146.21.154

    - name: health_monitor
      template: health_monitor
      instances: 1
      resource_pool: small
      networks:
      - name: default
        static_ips:
        - 10.146.21.155

    properties:
      env:

      blobstore:
        address: 10.146.21.154
        port: 25251
        backend_port: 25552
        agent:
          user: agent
          password: password
        director:
          user: director
          password: password

      networks:
        apps: default
        management: default

      nats:
        user: nats
        password: password
        address: 10.146.21.157
        port: 4222

      postgres:
        user: bosh
        password: password
        address: 10.146.21.151
        port: 5432
        database: bosh

      redis:
        address: 10.146.21.152
        port: 25255
        password: password

      director:
        name: vdc-1-bosh
        address: 10.146.21.153
        port: 25555
        db:
          host: 10.146.21.151
          port: 5432
          user: bosh
          password: password
          database: bosh

      hm:
        http:
          port: 25923
          user: admin
          password: admin
        director_account:
          user: admin
          password: admin
        intervals:
          poll_director: 60
          poll_grace_period: 30
          log_stats: 300
          analyze_agents: 60
          agent_timeout: 180
          rogue_agent_alert: 180
        loglevel: info
        email_notifications: false
        email_recipients:
        - noemail@yourdomain.com
        smtp:
          from: bhm@localhost.localdomain
          host: smtp.vmware.com
          port: 25
          auth: plain
          user: appcloud
          password: 38fhsoeY
          domain: localdomain
        tsdb_enabled: false

      vcd:
        url: <your_vcd_endpoint>
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
