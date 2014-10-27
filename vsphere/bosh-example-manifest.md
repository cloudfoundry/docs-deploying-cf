---
title: Example Manifest for Deploying BOSH with MicroBOSH on vSphere
---

This topic contains a manifest for deploying BOSH with MicroBOSH on vSphere.
Save a copy of this template as a YAML file in your deployment directory.
Customize the contents of this file for your environment, then use the
customized file to deploy BOSH to vSphere using MicroBOSH.

```yaml
name: bosh
director_uuid: DIRECTOR-UUID # Replace with the MicroBOSH Director UUID
release:
  name: bosh
  version: 111 # Replace with the current BOSH version

networks:
- name: default
  subnets:
  - range: 192.168.50.0/24             # Replace with an IP address range on your private network
    reserved:
    - 192.168.50.2 - 192.168.50.100    # Replace with an IP address range on your private network
    static:
    - 192.168.50.141 - 192.168.50.149  # Replace with an IP address range on your private network
    gateway: 192.168.50.1              # Change to your network gateway IP address
    dns:
    - 8.8.8.8                          # Change to the DNS server IP address
    cloud_properties:
      name: Servers                    # Change to the VLAN where you are deploying BOSH as provisioned on your vCenter

resource_pools:
- name: small
  stemcell:
    name: bosh-vsphere-esxi-ubuntu  # Replace with the name of the BOSH stemcell
    version: latest
  network: default
  size: 5
  cloud_properties:
    ram: 512
    disk: 2048
    cpu: 1
- name: director
  stemcell:
    name: bosh-vsphere-esxi-ubuntu  # Replace with the name of the BOSH stemcell
    version: latest
  network: default
  size: 1
  cloud_properties:
    ram: 2048
    disk: 8192
    cpu: 2

compilation:
  workers: 6
  network: default
  cloud_properties:
    ram: 2048
    disk: 4048
    cpu: 4

update:
  canaries: 1
  canary_watch_time: 60000
  update_watch_time: 60000
  max_in_flight: 1

jobs:
- name: nats
  template: nats
  instances: 1
  resource_pool: small
  networks:
  - name: default
    static_ips:
    - 192.168.50.142  # Replace with an IP address in the range you set for `network: static`

- name: postgres
  template: postgres
  instances: 1
  resource_pool: small
  persistent_disk: 2048
  networks:
  - name: default
    static_ips:
    - 192.168.50.143  # Replace with an IP address in the range you set for `network: static`

- name: redis
  template: redis
  instances: 1
  resource_pool: small
  networks:
  - name: default
    static_ips:
    - 192.168.50.144  # Replace with an IP address in the range you set for `network: static`

- name: director
  template: director
  instances: 1
  resource_pool: director
  persistent_disk: 2048
  networks:
  - name: default
    static_ips:
    - 192.168.50.145  # Replace with an IP address in the range you set for `network: static`

- name: blobstore
  template: blobstore
  instances: 1
  resource_pool: small
  persistent_disk: 20480
  networks:
  - name: default
    static_ips:
    - 192.168.50.146  # Replace with an IP address in the range you set for `network: static`

- name: health_monitor
  template: health_monitor
  instances: 1
  resource_pool: small
  networks:
  - name: default
    static_ips:
    - 192.168.50.147  # Replace with an IP address in the range you set for `network: static`

properties:
  env:

  blobstore:
    address: 192.168.50.146  # Replace with the IP address you set for the `static_ips` of the `blobstore` job
    port: 25251
    backend_port: 25552
    agent:
      user: agent           # Creates a read-only user for BOSH Agents
      password: mysecretpw  # Password for BOSH Agents
    director:
      user: director        # Creates a read/write user for BOSH Director
      password: mysecretpw  # Password for BOSH Director

  networks:
    apps: default
    management: default

  nats:
    user: nats
    password: mysecretpw
    address: 192.168.50.142 # Replace with the IP address you set for the `static_ips` of the `nats` job
    port: 4222

  postgres:
    user: bosh
    password: mysecretpw
    address: 192.168.50.143  # Replace with the IP address you set for the `static_ips` of the `postgres` job
    port: 5432
    database: bosh

  redis:
    address: 192.168.50.144  # Replace with the IP address you set for the `static_ips` of the `redis` job
    port: 25255
    password: mysecretpw

  director:
    name: microBOSH-director
    address: 192.168.50.145 # Replace with the IP address you set for the `static_ips` of the `director` job
    port: 25555
    encryption: false
    db:
      user: bosh
      password: mysecretpw
      host: 192.168.50.143  # Replace with the IP address you set for the `static_ips` of the `postgres` job

  hm:
    http:
      port: 25923
      user: admin           # Replace with a user name
      password: mysecretpw  # Replace with a password
    director_account:
      user: admin           # Replace with a user name
      password: mysecretpw  # Replace with a password
    intervals:
      poll_director: 60
      poll_grace_period: 30
      log_stats: 300
      analyze_agents: 60
      agent_timeout: 180
      rogue_agent_alert: 180
    loglevel: info
    email_notifications: false # If set to `false`, the values in the `smtp` section are ignored
    email_recipients:
    - anyuser@example.com  # Replace with any email address
    smtp:
      from: anyuser@example.com  # Replace
      host: smtp.example.com     # Replace
      port: 25
      auth: plain
      user: your-smtp-user       # Replace with a user name
      password: smtp-password    # Replace with a password
      domain: localdomain        # Replace with your domain
    tsdb_enabled: false # If set to `false`, the values in the `tsdb` section are ignored.
                        # Can only be set to `true` if you have a complete running deployment of Cloud Foundry.
    tsdb:
      address: 172.20.218.14 # (Optional) The opentsdb static IP address from your Cloud Foundry deployment
      port: 4242

  vcenter: # The values in this section must match your vCenter information
    address: vcenter.example.com # IP address or FQDN of your vCenter
    user: domain\bosh-user # User name provisioned for BOSH
    password: mysecretpw # Password for BOSH
    datacenters:
      - name: mybosh
        vm_folder: bosh_vms
        template_folder: bosh_templates
        disk_path: microbosh_datastore
        datastore_pattern: DS411_NFS_1
        persistent_datastore_pattern: DS411_NFS_1
        allow_mixed_datastores: true
        clusters:
        - ShuttleXPC:
            resource_pool: bosh