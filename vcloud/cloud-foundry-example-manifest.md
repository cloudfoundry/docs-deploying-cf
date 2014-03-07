---
title: Cloud Foundry Example Manifest
---

    ---
    name: cf
    director_uuid: bosh_director_uuid
    releases:
    - name: cf
      version: latest
    networks:
    - name: default
      subnets:
      - range: 10.146.21.128/25
        gateway: 10.146.21.253
        dns:
        - 10.132.71.1
        - 10.132.71.2
        static:
        - 10.146.21.171 - 10.146.21.184
        reserved:
        - 10.146.21.129 - 10.146.21.160
        - 10.146.21.200 - 10.146.21.223
        cloud_properties:
          name: "vdc_network"
    resource_pools:
    - name: nats
      stemcell:
        name: bosh-vcloud-esxi-ubuntu
        version: 1868
      network: default
      size: 1
      cloud_properties:
        ram: 1024
        disk: 2048
        cpu: 1
      env:
        vapp: cf_vapp
    - name: health_manager
      stemcell:
        name: bosh-vcloud-esxi-ubuntu
        version: 1868
      network: default
      size: 1
      cloud_properties:
        ram: 1024
        disk: 2048
        cpu: 1
      env:
        vapp: cf_vapp
    - name: nfs_server
      stemcell:
        name: bosh-vcloud-esxi-ubuntu
        version: 1868
      network: default
      size: 1
      cloud_properties:
        ram: 1024
        disk: 2048
        cpu: 1
      env:
        vapp: cf_vapp
    - name: ccdb
      stemcell:
        name: bosh-vcloud-esxi-ubuntu
        version: 1868
      network: default
      size: 1
      cloud_properties:
        ram: 1024
        disk: 2048
        cpu: 1
      env:
        vapp: cf_vapp
    - name: cloud_controller
      stemcell:
        name: bosh-vcloud-esxi-ubuntu
        version: 1868
      network: default
      size: 1
      cloud_properties:
        ram: 1024
        disk: 2048
        cpu: 1
      env:
        vapp: cf_vapp
    - name: router
      stemcell:
        name: bosh-vcloud-esxi-ubuntu
        version: 1868
      network: default
      size: 1
      cloud_properties:
        ram: 1024
        disk: 2048
        cpu: 1
      env:
        vapp: cf_vapp
    - name: syslog
      stemcell:
        name: bosh-vcloud-esxi-ubuntu
        version: 1868
      network: default
      size: 1
      cloud_properties:
        ram: 1024
        disk: 2048
        cpu: 1
      env:
        vapp: cf_vapp
    - name: collector
      stemcell:
        name: bosh-vcloud-esxi-ubuntu
        version: 1868
      network: default
      size: 1
      cloud_properties:
        ram: 1024
        disk: 2048
        cpu: 1
      env:
        vapp: cf_vapp
    - name: uaadb
      stemcell:
        name: bosh-vcloud-esxi-ubuntu
        version: 1868
      network: default
      size: 1
      cloud_properties:
        ram: 1024
        disk: 2048
        cpu: 1
      env:
        vapp: cf_vapp
    - name: uaa
      stemcell:
        name: bosh-vcloud-esxi-ubuntu
        version: 1868
      network: default
      size: 1
      cloud_properties:
        ram: 1024
        disk: 2048
        cpu: 1
      env:
        vapp: cf_vapp
    - name: saml_login
      stemcell:
        name: bosh-vcloud-esxi-ubuntu
        version: 1868
      network: default
      size: 1
      cloud_properties:
        ram: 1024
        disk: 2048
        cpu: 1
      env:
        vapp: cf_vapp
    - name: mysql_gateway
      stemcell:
        name: bosh-vcloud-esxi-ubuntu
        version: 1868
      network: default
      size: 1
      cloud_properties:
        ram: 1024
        disk: 2048
        cpu: 1
      env:
        vapp: cf_vapp
    - name: mysql_node
      stemcell:
        name: bosh-vcloud-esxi-ubuntu
        version: 1868
      network: default
      size: 1
      cloud_properties:
        ram: 2048
        disk: 3072
        cpu: 1
      env:
        vapp: cf_vapp
    - name: dea
      stemcell:
        name: bosh-vcloud-esxi-ubuntu
        version: 1868
      network: default
      size: 1
      cloud_properties:
        ram: 16384
        disk: 10240
        cpu: 2
      env:
        vapp: cf_vapp
    compilation:
      workers: 6
      network: default
      cloud_properties:
        ram: 2048
        disk: 4096
        cpu: 2
    update:
      canaries: 1
      canary_watch_time: 60000
      update_watch_time: 60000
      max_in_flight: 1
      max_errors: 3
    jobs:
    - name: nats
      release: cf
      template: nats
      instances: 1
      resource_pool: nats
      persistent_disk: 0
      networks:
      - name: default
        static_ips:
        - 10.146.21.172
    - name: health_manager
      release: cf
      template: health_manager_next
      instances: 1
      resource_pool: health_manager
      persistent_disk: 0
      networks:
      - name: default
        static_ips:
        - 10.146.21.173
    - name: nfs_server
      release: cf
      template: debian_nfs_server
      instances: 1
      resource_pool: nfs_server
      persistent_disk: 8192
      networks:
      - name: default
        static_ips:
        - 10.146.21.174
    - name: ccdb
      release: cf
      template: postgres
      instances: 1
      resource_pool: ccdb
      persistent_disk: 2048
      networks:
      - name: default
        static_ips:
        - 10.146.21.175
      properties:
        db: ccdb
    - name: cloud_controller
      release: cf
      template: cloud_controller_ng
      instances: 1
      resource_pool: cloud_controller
      persistent_disk: 0
      networks:
      - name: default
        static_ips:
        - 10.146.21.176
      properties:
        ccdb: ccdb
        login:
          url: http://login.cloudfoundry.com
    - name: router
      release: cf
      template: gorouter
      instances: 1
      resource_pool: router
      persistent_disk: 0
      networks:
      - name: default
        default:
        - dns
        - gateway
        static_ips:
        - 10.146.21.171
    - name: syslog
      release: cf
      template: syslog_aggregator
      instances: 1
      resource_pool: syslog
      persistent_disk: 8192
      networks:
      - name: default
        static_ips:
        - 10.146.21.177
    - name: collector
      release: cf
      template: collector
      instances: 1
      resource_pool: collector
      persistent_disk: 0
      networks:
      - name: default
        static_ips:
        - 10.146.21.178
    - name: uaadb
      release: cf
      template: postgres
      instances: 1
      resource_pool: uaadb
      persistent_disk: 8192
      networks:
      - name: default
        static_ips:
        - 10.146.21.179
      properties:
        db: uaadb
        networks:
          services: default
    - name: uaa
      release: cf
      template: uaa
      instances: 1
      resource_pool: uaa
      persistent_disk: 0
      networks:
      - name: default
        static_ips:
        - 10.146.21.180
      properties:
        db: uaadb
        networks:
          services: default
    - name: saml_login
      release: cf
      template: saml_login
      instances: 1
      resource_pool: saml_login
      persistent_disk: 0
      networks:
      - name: default
        static_ips:
        - 10.146.21.181
    - name: dea
      release: cf
      template: dea_next
      instances: 1
      resource_pool: dea
      persistent_disk: 0
      networks:
      - name: default
        default:
        - dns
        - gateway
      properties:
        dea_next:
          stacks:
          - lucid64
    properties:
      saml_login:
        uaa_base: http://uaa.cloudfoundry.com
        serviceProviderKey: <serviceProviderKey>
        serviceProviderCertificate: <serviceProviderCertificate>
        idp_entity_alias: sso-sp
        idp_metadata_url: ! ''''''
      domain: cloudfoundry.com
      system_domain: cloudfoundry.com
      system_domain_organization: system
      app_domains:
      - cloudfoundry.com
      networks:
        apps: default
        management: default
      nats:
        user: nats
        password: <password>
        address: 10.146.21.172
        port: 4222
      nfs_server:
        address: 10.146.21.174
        network: 10.146.21.128/25
      service_plans:
        mysql:
          '100':
            description: Shared service instance, 1MB memory, 10MB storage, 10 connections
            job_management:
              high_water: 900
              low_water: 100
            configuration:
              capacity: 500
              max_db_size: 10
              key_buffer: 128
              innodb_buffer_pool_size: 128
              max_allowed_packet: 16
              thread_cache_size: 8
              query_cache_size: 8
              max_long_query: 3
              max_long_tx: 30
              max_clients: 10
              max_connections: 100
              table_open_cache: 200
              innodb_tables_per_database: 100
              connection_pool_size:
                min: 5
                max: 10
              backup: 
              lifecycle:
                serialization: enable
                snapshot:
                  quota: 1
              warden:
                enable: false
      ccdb: &70259539504520
        address: 10.146.21.175
        port: 2544
        db_scheme: postgres
        roles:
        - tag: admin
          name: admin
          password: <password>
        databases:
        - tag: cc
          name: ccdb
          citext: true
      ccdb_ng: *70259539504520
      uaadb:
        address: 10.146.21.179
        port: 2544
        db_scheme: postgresql
        roles:
        - tag: admin
          name: root
          password: <password>
        databases:
        - tag: uaa
          name: uaa
      cc_api_verstion: v2
      cc: &70259539500020
        srv_api_uri: http://api.cloudfoundry.com
        external_host: api
        logging_level: debug
        uaa_resource_id: cloud_controller
        staging_upload_user: staging_upload_user
        staging_upload_password: <password>
        bulk_api_password: <password>
        bootstrap_admin_email: admin
        cc_partition: default
        stacks:
        - name: lucid64
          description: Ubuntu 10.04
        db_encryption_key: <encryption_key>
        quota_definition:
          standard:
            memory_limit: 256
            total_services: -1
      ccng: *70259539500020
      router:
        status:
          port: 8080
          user: router_status
          password: <password>
      dea_next:
        memory_mb: 16384
        memory_overcommit_factor: 1
        disk_mb: 10240
        disk_overcommit_factor: 1
        num_instances: 10
      collector:
        use_tsdb: false
      serialization_data_server:
        upload_token: dummy
        upload_timeout: 9999
        port: dummy
      service_lifecycle:
        serialization_data_server: []
      syslog_aggregator:
        address: 10.146.21.177
        port: 54321
      uaa:
        catalina_opts: -Xmx768m -XX:MaxPermSize=256m
        url: http://uaa.cloudfoundry.com
        jwt:
          signing_key: <signing_key>
          verification_key: <verification_key>
        cc:
          client_secret: <client_secret>
        admin:
          client_secret: <client_secret>
        client:
          override: true
        clients:
          login:
            id: login
            override: true
            autoapprove: true
            authorities: oauth.login
            authorized-grant-types: authorization_code,client_credentials,refresh_token
            scope: openid
            secret: <secret>
          cf:
            id: cf
            override: true
            autoapprove: true
            authorities: uaa.none
            authorized-grant-types: implicit,password,refresh_token
            scope: cloud_controller.read,cloud_controller.write,openid,password.write,cloud_controller.admin,scim.read,scim.write
          system_passwords:
            id: system_passwords
            override: true
            autoapprove: true
            authorities: uaa.admin,scim.read,scim.write,password.write
            authorized-grant-types: client_credentials
            secret: <secret>
        scim:
          user:
            override: false
          users:
          - admin|25ed20f0bd6cba8d21f2|scim.write,scim.read,openid,cloud_controller.admin,dashboard.user
          - system_services|2033611d7dc97f7f624b|cloud_controller.admin
          - system_verification|7090dcfb61f228a44220|cloud_controller.admin
