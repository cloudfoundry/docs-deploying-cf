---
title: Installing Cloud Foundry on OpenStack
---

This topic describes how to install Cloud Foundry on OpenStack.

<p class="note"><strong>Note</strong>: These instructions are for the v183 release of Cloud Foundry. For newer versions of Cloud Foundry, please <a href="https://github.com/cloudfoundry/docs-deploying-cf/issues/new">let us know</a> if this documentation continues to work or needs to be updated.</p>

## Goal ##

By the end of this topic, Cloud Foundry will be installed and running on OpenStack.
You will be able to target it using your own DNS, login, and upload a simple application.

For example, you will be able to do the following:

<pre class="terminal">
$ cf target api.mycloud.com
$ cf login admin
Password> ******
$ git clone https://github.com/cloudfoundry-community/cf_demoapp_ruby_rack.git
$ cd cf_demoapp_ruby_rack
$ cf push
$ open http://hello.mycloud.com
Hello World!
</pre>

## Requirements ##

To complete the steps in this topic, you must [validate your OpenStack](validate_openstack.html) and [have a BOSH deployment running](deploying_microbosh.html).

You must also provision an IP address and setup your DNS to map `*` records to this IP address. For example, if you were using `mycloud.com` domain as the base domain for your Cloud Foundry, you need a `*` A record for this zone mapping to your provisioned IP address.

Confirm in your local terminal that you have the installed the BOSH CLI and have targeted your BOSH:

<pre class="terminate">
$ bosh status
Config
             /Users/example/.bosh_config

Director
  Name       mybosh
  URL        https://1.2.3.4:25555
  Version    1.5.0.pre.939 (release:930f73f5 bosh:930f73f5)
  User       admin
  UUID       18bbfe79-6ccf-4020-b1c7-6f19d67a8c9c
  CPI        openstack
  dns        enabled (domain_name: microbosh)
  compiled_package_cache disabled

Deployment
  not set
</pre>

You will use the `UUID` value found above when configuring your manifest.

You have two flavors setup **with ephemeral disks**:

* `m1.small` - for example, 1 CPU, 2G RAM, 20G ephemeral disk
* `m1.medium` - for example, 2 CPU, 4G RAM, 20G ephemeral disk

You have two security groups:

* `default`
* `cf` - opens all ports (until further understanding of what ports are required for each job of Cloud Foundry)

## Upload BOSH Release ##

To deploy Cloud Foundry, your BOSH needs to be given the BOSH release it will use. This includes all the packages and jobs. The Cloud Foundry BOSH release includes dozens of jobs and almost 100 packages. Enforcing consistency and guaranteed output of a deployment is one of the reasons we prefer BOSH for deploying a complex system like Cloud Foundry.

You can now create and upload the Cloud Foundry BOSH release ([cf-release](https://github.com/cloudfoundry/cf-release)).

[Follow these instructions.](../common/cf-release.html)

Confirm that you have a release "cf" uploaded:

<pre class="terminate">
$ bosh releases

+------+----------+-------------+
| Name | Versions | Commit Hash |
+------+----------+-------------+
| cf   | 170      | aabs1233ad  |
+------+----------+-------------+
</pre>

## Upload a base stemcell ##

A cloud provider needs a base image to provision VMs/servers. BOSH explicitly requires that the base image includes the [Agent](/bosh/terminology.html#agent). Therefore, we use specific base images which are known to have a BOSH agent installed. These base images are called BOSH **stemcells**.

To upload the latest BOSH stemcell to your BOSH:

<pre class="terminate">
$ stemcell_url=$(bosh public stemcells --full | grep openstack | grep trusty | awk '{print $4}')
$ bosh upload stemcell $stemcell_url
</pre>

<p class="note"><strong>Note</strong>: There has been a <a href="https://www.pivotaltracker.com/story/show/62108468">report</a> on the vcap-dev mailing list that cf-release v147 and other releases through v150 are incompatible with some latest versions of openstack-kvm-ubuntu stemcell. The stemcell that works for the user reporting the issue is <a href="https://bosh-jenkins-artifacts.s3.amazonaws.com/bosh-stemcell/openstack/bosh-stemcell-1256-openstack-kvm-ubuntu.tgz">1256</a>.</p>

Confirm that you have at least one BOSH stemcell loaded into your BOSH:

<pre class="terminate">
$ bosh stemcells

+--------------------------------+---------+-----------------------------------------+
| Name                           | Version | CID                                     |
+--------------------------------+---------+-----------------------------------------+
| bosh-openstack-kvm-ubuntu      | 1055    | sc-f66fc9db-150d-4e2e-b78f-ad4ab7fc7ad1 |
+--------------------------------+---------+-----------------------------------------+

Stemcells total: 1
</pre>

## Other preparation

OpenStack uses security groups for restricting/allowing ports to a set of servers.

You need to create a security group `cf`:

* allow all ports to talk to all ports of servers using the same `cf` security group
* allow port 22 for administrator SSH access
* allow 80 for HTTP traffic (primarily for the router)

## Create a minimal deployment file ##

The next step towards deploying Cloud Foundry is to create a `deployment file`. This is a YAML file that describes exactly what will be included in the next deployment. This includes:

* the VMs to be created
* the persistent disks to be attached to different VMs
* the networks and IP addresses to be bound to each VM
* the one or more job templates from the BOSH release to be applied to each VM
* the custom properties to be applied into configuration files and scripts for each job template

A deployment file can describe 10,000 VMs using a complex set of Neutron subnets all the way down to one or more VMs without any complex inter-networking. You can specify one job per VM or colocate multiple jobs for small deployments.

There are many different parts of Cloud Foundry that can be deployed. In this section, only the bare basics will be deployed that allow user applications to be run that do not require any services. The minimal set of jobs to be included are:

* `dea`
* `cloud_controller_ng`
* `gorouter`
* `nats`
* `postgres`
* `hm9000`
* `syslog_aggregator`
* `debian_nfs_server` (see below for using Swift as the droplet blobstore)
* `uaa`

There are different ways that networking can be configured. If your OpenStack has Neutron running, then you can use advanced compositions of subnets to isolate and protect each job, thus providing greater security. In this section, no advanced networking will be used.

Only a single public IP is required. Replace `REPLACE-IP-ADDRESS` with your allocated IP address below.

### Initial manifest for OpenStack ###

Create a `~/bosh-workspace/deployments/cf` folder and create `~/bosh-workspace/deployments/cf/demo.yml` with the initial deployment file displayed below.

Change the following at the top of the file:

* replace `REPLACE-DIRECTOR_UUID` with the UUID from `bosh status`
* replace `REPLACE-IP-ADDRESS` with the IP address you allocated above
* replace `root_domain` value with a DNS, say `mycloud.com` that has `*.mycloud.com` mapped to your IP; defaults to using http://xip.io service for DNS
* replace the `common_password`; even better is to put in lots of different passwords and tokens throughout the deployment file

Further, note that you need to have configured the "cf-public" and "cf-private" security groups as outlined in [these instructions](../common/security_groups.html).

~~~
<%
DIRECTOR_UUID = "REPLACE-DIRECTOR_UUID"
static_ip = "REPLACE-IP-ADDRESS"
root_domain = "#{static_ip}.xip.io"
deployment_name = 'cf'
cf_release = '183'
protocol = 'http'
common_password = 'c1oudc0wc1oudc0w'
%>

---
name: <%= deployment_name %>
director_uuid: <%= director_uuid %>

releases:
- name: cf
  version: <%= cf_release %>

compilation:
  workers: 2
  network: default
  reuse_compilation_vms: true
  cloud_properties:
    instance_type: bosh.medium

update:
  canaries: 1
  canary_watch_time: 30000-600000
  update_watch_time: 30000-600000
  max_in_flight: 3
  serial: true

networks:
- name: default
  type: dynamic
  - cloud_properties:
      net_id: 0a3ac1cf-5019-4028-9d56-433b470fda46 # This is 10.0.0.0/24
          # network. If you change this, also change the `nfs_server` property
          # in "allow_from_entries:" listed below
      security_groups:
      - default
      - bosh
      - cf-private

- name: external
  type: dynamic
  - cloud_properties:
      net_id: 0a3ac1cf-5019-4028-9d56-433b470fda46
      security_groups:
      - default
      - bosh
      - cf-private
      - cf-public

- name: float
  type: vip
  - cloud_properties:
      net_id: a331fd43-51de-4fa6-b82d-71137a36bc06

resource_pools:
- cloud_properties:
    instance_type: bosh.small
  name: common
  network: default
  stemcell:
    name: bosh-openstack-kvm-ubuntu-trusty-go_agent
    version: latest
- cloud_properties:
    instance_type: bosh.medium
  name: large
  network: default
  stemcell:
    name: bosh-openstack-kvm-ubuntu-trusty-go_agent
    version: latest

jobs:
- default_networks:
  - name: cf1
    static_ips: null
  instances: 1
  name: ha_proxy_z1
  networks:
  - name: floating
    static_ips:
    - 0.0.0.0
  - default: null
    name: cf1
    static_ips:
    - 0.0.0.0
  properties:
    ha_proxy:
      ssl_pem: null
    networks:
      apps: cf1
    router:
      servers:
        z1:
        - 0.0.0.5
        z2: []
  resource_pool: router_z1
  templates:
  - name: haproxy
    release: cf
- instances: 1
  name: nats_z1
  networks:
  - name: cf1
    static_ips:
    - 0.0.0.2
  properties:
    networks:
      apps: cf1
  resource_pool: medium_z1
  templates:
  - name: nats
    release: cf
  - name: nats_stream_forwarder
    release: cf
- instances: 0
  name: nats_z2
  networks:
  - name: cf2
    static_ips: []
  properties:
    networks:
      apps: cf2
  resource_pool: medium_z2
  templates:
  - name: nats
    release: cf
  - name: nats_stream_forwarder
    release: cf
- instances: 1
  name: etcd_z1
  networks:
  - name: cf1
    static_ips:
    - 0.0.0.8
  persistent_disk: 10024
  properties:
    networks:
      apps: cf1
  resource_pool: medium_z1
  templates:
  - name: etcd
    release: cf
  - name: etcd_metrics_server
    release: cf
- instances: 0
  name: etcd_z2
  networks:
  - name: cf2
    static_ips: []
  persistent_disk: 10024
  properties:
    networks:
      apps: cf2
  resource_pool: medium_z2
  templates:
  - name: etcd
    release: cf
  - name: etcd_metrics_server
    release: cf
- instances: 1
  name: logs_z1
  networks:
  - name: cf1
    static_ips:
    - 0.0.0.1
  persistent_disk: 100000
  properties:
    networks:
      apps: cf1
  resource_pool: medium_z1
  templates:
  - name: syslog_aggregator
    release: cf
- instances: 0
  name: logs_z2
  networks:
  - name: cf2
    static_ips: []
  persistent_disk: 100000
  properties:
    networks:
      apps: cf2
  resource_pool: medium_z2
  templates:
  - name: syslog_aggregator
    release: cf
- instances: 1
  name: stats_z1
  networks:
  - name: cf1
  properties:
    networks:
      apps: cf1
  resource_pool: small_z1
  templates:
  - name: collector
    release: cf
- instances: 1
  name: nfs_z1
  networks:
  - name: cf1
    static_ips:
    - 0.0.0.3
  persistent_disk: 102400
  resource_pool: medium_z1
  templates:
  - name: debian_nfs_server
    release: cf
- instances: 1
  name: postgres_z1
  networks:
  - name: cf1
    static_ips:
    - 0.0.0.4
  persistent_disk: 4096
  resource_pool: medium_z1
  templates:
  - name: postgres
    release: cf
- instances: 1
  name: uaa_z1
  networks:
  - name: cf1
  properties:
    networks:
      apps: cf1
  resource_pool: medium_z1
  templates:
  - name: uaa
    release: cf
- instances: 0
  name: uaa_z2
  networks:
  - name: cf2
  properties:
    networks:
      apps: cf2
  resource_pool: medium_z2
  templates:
  - name: uaa
    release: cf
- instances: 1
  name: login_z1
  networks:
  - name: cf1
  properties:
    networks:
      apps: cf1
  resource_pool: medium_z1
  templates:
  - name: login
    release: cf
- instances: 0
  name: login_z2
  networks:
  - name: cf2
  properties:
    networks:
      apps: cf2
  resource_pool: medium_z2
  templates:
  - name: login
    release: cf
- instances: 1
  name: api_z1
  networks:
  - name: cf1
  persistent_disk: 0
  properties:
    metron_agent:
      zone: z1
    networks:
      apps: cf1
    nfs_server:
      address: 0.0.0.3
      allow_from_entries:
      - null
      - null
      share: null
  resource_pool: large_z1
  templates:
  - name: cloud_controller_ng
    release: cf
  - name: metron_agent
    release: cf
- instances: 0
  name: api_z2
  networks:
  - name: cf2
  persistent_disk: 0
  properties:
    metron_agent:
      zone: z2
    networks:
      apps: cf2
    nfs_server:
      address: 0.0.0.3
      allow_from_entries:
      - null
      - null
      share: null
  resource_pool: large_z2
  templates:
  - name: cloud_controller_ng
    release: cf
  - name: metron_agent
    release: cf
- instances: 1
  name: clock_global
  networks:
  - name: cf1
  persistent_disk: 0
  properties:
    metron_agent:
      zone: z1
    networks:
      apps: cf1
  resource_pool: medium_z1
  templates:
  - name: cloud_controller_clock
    release: cf
  - name: metron_agent
    release: cf
- instances: 1
  name: api_worker_z1
  networks:
  - name: cf1
  persistent_disk: 0
  properties:
    metron_agent:
      zone: z1
    networks:
      apps: cf1
    nfs_server:
      address: 0.0.0.3
      allow_from_entries:
      - null
      - null
      share: null
  resource_pool: small_z1
  templates:
  - name: cloud_controller_worker
    release: cf
  - name: metron_agent
    release: cf
- instances: 0
  name: api_worker_z2
  networks:
  - name: cf2
  persistent_disk: 0
  properties:
    metron_agent:
      zone: z2
    networks:
      apps: cf2
    nfs_server:
      address: 0.0.0.3
      allow_from_entries:
      - null
      - null
      share: null
  resource_pool: small_z2
  templates:
  - name: cloud_controller_worker
    release: cf
  - name: metron_agent
    release: cf
- instances: 1
  name: hm9000_z1
  networks:
  - name: cf1
  properties:
    networks:
      apps: cf1
  resource_pool: medium_z1
  templates:
  - name: hm9000
    release: cf
- instances: 0
  name: hm9000_z2
  networks:
  - name: cf2
  properties:
    networks:
      apps: cf2
  resource_pool: medium_z2
  templates:
  - name: hm9000
    release: cf
- instances: 1
  name: runner_z1
  networks:
  - name: cf1
    static_ips: null
  properties:
    dea_next:
      zone: z1
    metron_agent:
      zone: z1
    networks:
      apps: cf1
  resource_pool: runner_z1
  templates:
  - name: dea_next
    release: cf
  - name: dea_logging_agent
    release: cf
 name: metron_agent
    release: cf
  update:
    max_in_flight: 1
- instances: 0
  name: runner_z2
  networks:
  - name: cf2
    static_ips: null
  properties:
    dea_next:
      zone: z2
  metron_agent:
      zone: z2
    networks:
      apps: cf2
  resource_pool: runner_z2
  templates:
  - name: dea_next
    release: cf
  - name: dea_logging_agent
    release: cf
  - name: metron_agent
    release: cf
  update:
    max_in_flight: 1
- instances: 1
  name: loggregator_z1
  networks:
  - name: cf1
    static_ips:
    - 0.0.0.6
  properties:
    doppler:
      zone: z1
    networks:
      apps: cf1
  resource_pool: medium_z1
  templates:
  - name: doppler
    release: cf
- instances: 0
  name: loggregator_z2
  networks:
  - name: cf2
    static_ips: []
  properties:
    doppler:
      zone: z2
    networks:
      apps: cf2
  resource_pool: medium_z2
  templates:
  - name: doppler
    release: cf
- instances: 1
  name: loggregator_trafficcontroller_z1
  networks:
  - name: cf1
    static_ips:
    - 0.0.0.7
  properties:
    metron_agent:
      zone: z1
    networks:
      apps: cf1
    traffic_controller:
      zone: z1
  resource_pool: small_z1
  templates:
- name: loggregator_trafficcontroller
    release: cf
  - name: metron_agent
    release: cf
- instances: 0
  name: loggregator_trafficcontroller_z2
  networks:
  - name: cf2
    static_ips: []
  properties:
    metron_agent:
      zone: z2
    networks:
      apps: cf2
    traffic_controller:
      zone: z2
  resource_pool: small_z2
  templates:
  - name: loggregator_trafficcontroller
    release: cf
  - name: metron_agent
    release: cf
- instances: 1
  name: router_z1
  networks:
  - name: cf1
    static_ips:
    - 0.0.0.5
  properties:
    metron_agent:
      zone: z1
    networks:
      apps: cf1
  resource_pool: router_z1
  templates:
  - name: gorouter
    release: cf
  - name: metron_agent
    release: cf
- instances: 0
  name: router_z2
  networks:
  - name: cf2
    static_ips: []
  properties:
    metron_agent:
      zone: z2
    networks:
      apps: cf2
  resource_pool: router_z2
  templates:
  - name: gorouter
    release: cf
  - name: metron_agent
  release: cf
- instances: 0
  lifecycle: errand
  name: acceptance_tests
  networks:
  - name: cf1
  resource_pool: small_errand
  templates:
  - name: acceptance-tests
    release: cf
- instances: 0
  lifecycle: errand
  name: smoke_tests
  networks:
  - name: cf1
  resource_pool: small_errand
  templates:
  - name: smoke-tests
    release: cf

properties:
  domain: <%= root_domain %>
  system_domain: <%= root_domain %>
  system_domain_organization: 'admin'
  app_domains:
    - <%= root_domain %>
  nats:
    user: nats_user
    password: <%= common_password %>
    address: 0.nats.default.<%= deployment_name %>.microbosh
    port: 4222
    machines:
    - 0.nats.default.<%= deployment_name %>.microbosh
  syslog_aggregator:
    address: 0.syslog-aggregator.default.<%= deployment_name %>.microbosh
    port: 54321
  nfs_server:
    address: 0.nfs-server.default.<%= deployment_name %>.microbosh
    network: "*.<%= deployment_name %>.microbosh"
    allow_from_entries:
    - 10.0.0.0/24
  metron_agent:
    zone: z1
  metron_endpoint:
    zone: z1
    shared_secret: <%= common_password %>
  loggregator:
    zone: z1
    servers:
    zone:
    - 0.loggregator.default.<%= deployment_name %>.microbosh
  loggregator_endpoint:
    shared_secret: <%= common_password %>
    host: 0.trafficcontroller.default.<%= deployment_name %>.microbosh
  logger_endpoint:
    use_ssl: <%= protocol == 'https' %>
    port: 80
  ssl:
    skip_cert_verify: true
  router:
    endpoint_timeout: 60
    status:
    port: 8080
    user: gorouter
    password: <%= common_password %>
    servers:
      z1:
      - 0.router.default.<%= deployment_name %>.microbosh
      z2: []
  etcd:
    machines:
    - 0.etcd.default.<%= deployment_name %>.microbosh
  dea_next: *dea
  dea: &dea
    advertise_interval_in_seconds: 5
    allow_networks: null
    default_health_check_timeout: 60
    deny_networks: null
    directory_server_protocol: https
    disk_mb: 102400
    disk_overcommit_factor: 2
    evacuation_bail_out_time_in_seconds: 600
    heartbeat_interval_in_seconds: 10
    instance_disk_inode_limit: 200000
    kernel_network_tuning_enabled: true
    logging_level: debug
    memory_mb: 15000
    memory_overcommit_factor: 3
    staging_disk_inode_limit: 200000
    staging_disk_limit_mb: 4096
    staging_memory_limit_mb: 1024
  disk_quota_enabled: true
  databases: &databases
    db_scheme: postgres
    address: 0.postgres.default.<%= deployment_name %>.microbosh
    port: 5524
    roles:
    - tag: admin
      name: ccadmin
      password: <%= common_password %>
    - tag: admin
      name: uaaadmin
      password: <%= common_password %>
    databases:
    - tag: cc
      name: ccdb
      citext: true
    - tag: uaa
      name: uaadb
      citext: true
  ccdb: &ccdb
    db_scheme: postgres
    address: 0.postgres.default.<%= deployment_name %>.microbosh
    port: 5524
    roles:
    - tag: admin
      name: ccadmin
      password: <%= common_password %>
    databases:
    - tag: cc
      name: ccdb
      citext: true
  ccdb_ng: *ccdb
  uaadb:
    db_scheme: postgresql
    address: 0.postgres.default.<%= deployment_name %>.microbosh
    port: 5524
    roles:
    - tag: admin
      name: uaaadmin
      password: <%= common_password %>
    databases:
    - tag: uaa
      name: uaadb
      citext: true
  cc: &cc
    security_group_definitions:
    - name: public_networks
      rules:
      - protocol: all
        destination: 0.0.0.0-9.255.255.255
      - protocol: all
        destination: 11.0.0.0-169.253.255.255
      - protocol: all
        destination: 169.255.0.0-172.15.255.255
      - protocol: all
        destination: 172.32.0.0-192.167.255.255
      - protocol: all
        destination: 192.169.0.0-255.255.255.25
    - name: internal_network
      rules:
      - protocol: all
        destination: 10.0.0.0-10.255.255.255
    - name: dns
      rules:
      - destination: 0.0.0.0/0
        ports: '53'
        protocol: tcp
      - destination: 0.0.0.0/0
        ports: '53'
        protocol: udp
    default_running_security_groups:
    - public_networks
    - internal_network
    - dns
    default_staging_security_groups:
    - public_networks
    - internal_network
    - dns
    srv_api_uri: <%= protocol %>://api.<%= root_domain %>
    jobs:
      local:
        number_of_workers: 2
      generic:
        number_of_workers: 2
      global:
        timeout_in_seconds: 14400
      app_bits_packer:
        timeout_in_seconds: null
      app_events_cleanup:
        timeout_in_seconds: null
      app_usage_events_cleanup:
        timeout_in_seconds: null
      blobstore_delete:
        timeout_in_seconds: null
      blobstore_upload:
        timeout_in_seconds: null
      droplet_deletion:
        timeout_in_seconds: null
      droplet_upload:
        timeout_in_seconds: null
      model_deletion:
        timeout_in_seconds: null
    bulk_api_password: <%= common_password %>
    staging_upload_user: upload
      staging_upload_password: <%= common_password %>
    quota_definitions:
      default:
        memory_limit: 10240
        total_services: 100
        non_basic_services_allowed: true
        total_routes: 1000
        trial_db_allowed: true
    resource_pool:
      resource_directory_key: cloudfoundry-resources
      fog_connection:
        provider: Local
          local_root: /var/vcap/nfs/shared
    packages:
      app_package_directory_key: cloudfoundry-packages
      fog_connection:
      provider: Local
      local_root: /var/vcap/nfs/shared
    droplets:
      droplet_directory_key: cloudfoundry-droplets
      fog_connection:
      provider: Local
      local_root: /var/vcap/nfs/shared
    buildpacks:
      buildpack_directory_key: cloudfoundry-buildpacks
      fog_connection:
      provider: Local
      local_root: /var/vcap/nfs/shared
    install_buildpacks:
    - name: java_buildpack
      package: buildpack_java
    - name: ruby_buildpack
      package: buildpack_ruby
    - name: nodejs_buildpack
      package: buildpack_nodejs
    - name: go_buildpack
      package: buildpack_go
    db_encryption_key: <%= common_password %>
    hm9000_noop: false
    diego: false
    newrelic:
      license_key: null
      environment_name: <%= deployment_name %>
  ccng: *cc
    login:
    enabled: false
  uaa:
    url: <%= protocol %>://uaa.<%= root_domain %>
    no_ssl: <%= protocol == 'http' %>
    cc:
      client_secret: <%= common_password %>
    admin:
      client_secret: <%= common_password %>
    batch:
      username: batch
      password: <%= common_password %>
    clients:
      cf:
        override: true
        authorized-grant-types: password,implicit,refresh_token
        authorities: uaa.none
        scope: cloud_controller.read,cloud_controller.write,openid,password.write,cloud_controller.admin,scim.read,scim.write
        access-token-validity: 7200
        refresh-token-validity: 1209600
      admin:
        secret: <%= common_password %>
        authorized-grant-types: client_credentials
        authorities: clients.read,clients.write,clients.secret,password.write,scim.read,uaa.admin
      scim:
        users:
        - admin|<%= common_password %>|scim.write,scim.read,openid,cloud_controller.admin,uaa.admin,password.write
        - services|<%= common_password %>|scim.write,scim.read,openid,cloud_controller.admin
    jwt:
      signing_key: RSA PRIVATE KEY
      verification_key: RSA PUBLIC KEY
  dea_logging_agent:
    status:
      user: admin
      password: <%= common_password %>
  haproxy: {}
  debian_nfs_server:
  no_root_squash: true
  traffic_controller:
    zone: 'zone'
~~~

## Deploying your own Cloud Foundry ##

In this section, your BOSH will be instructed to provision a number of VMs, bind the router to your IP address, and run the minimal, useful set of jobs mentioned above. In the subsequent section, you will deploy a sample application to your Cloud Foundry

First, target your BOSH CLI to your manifest file.

<pre class="terminal">
$ bosh deployment ~/bosh-workspace/deployments/cf/demo.yml
</pre>

Then, we upload the deployment file to your BOSH and instruct it to "deploy" your Cloud Foundry service.

<pre class="terminal">
$ bosh deploy
</pre>

### What is happening now? ###

The first time you deploy a BOSH release it will compile every package that it needs for your deployment. You will see something like:

<pre class="terminal">
Compiling packages
buildpack_cache/2,...  |                        | 0/23 00:00:32  ETA: --:--:--
</pre>

And later...

<pre class="terminal">
  Compiling packages
    buildpack_cache/2 (00:02:39)
    insight_agent/2 (00:02:49)
    nginx/9 (00:00:49)
golang/2, gorouter/10,...  |ooo                     | 4/23 00:02:15  ETA: 00:04:04
</pre>

If you visit your OpenStack dashboard, you will see a number of VMs have been provisioned. Each of these VMs is being assigned a single package to compile. When it completes, it uploads the compiled binaries and libraries for that package into the BOSH blobstore. These compiled packages can be used over and over and never need compilation again.

Finally, the initial compilation of packages ends (after about 15 minutes in the example below):

<pre class="terminal">
  ...
  login/19 (00:00:37)
  uaa/30 (00:00:38)
  dea_next/22 (00:00:51)
Done                    23/23 00:09:16
</pre>

Next it boots the nine VMs mentioned in the deployment file above:

<pre class="terminal">
Creating bound missing VMs
  common/1 (00:01:49)
  common/0 (00:02:53)
  common/2 (00:03:31)
  common/3 (00:01:53)
  common/5 (00:01:32)
  common/4 (00:02:10)
  common/6 (00:01:51)
  common/7 (00:01:07)
  large/0 (00:01:54)
Done                    9/9 00:07:34
</pre>

Finally it assigns each VM a job (which is a list of one or more job templates from the [cf-release BOSH release jobs folder](https://github.com/cloudfoundry/cf-release/tree/master/jobs))

<pre class="terminal">
Binding instance VMs
  nats/0 (00:00:01)
  postgres/0 (00:00:01)
  nfs_server/0 (00:00:01)
  syslog_aggregator/0 (00:00:01)
  uaa/0 (00:00:01)
  hm9000/0 (00:00:01)
  router/0 (00:00:01)
  cloud_controller/0 (00:00:01)
  dea/0 (00:00:01)
Done                    9/9 00:00:04

Preparing configuration
  binding configuration (00:00:01)
Done                    1/1 00:00:01

Updating job syslog_aggregator
syslog_aggregator/0 (canary)       |oooooooooooooooooooo    | 0/1 00:01:09  ETA: --:--:--
</pre>

First the `syslog_aggregator` job is started.

The ordering of the jobs within the deployment file is deliberate. It determines the order that the job VMs are started. The later jobs may have dependencies on the earlier jobs already running.

DEAs (job `dea`) need the NATS (job `nats`) and the Cloud Controller (job `cloud_controller`). The Cloud Controller needs the UAA (job `uaa`). The Cloud Controller and the UAA need a PostgreSQL database (job `postgres`).

When the deployment is finished:

<pre class="terminal">
Updating job core
  core/0 (canary) (00:01:04)
Done                    1/1 00:01:04

Updating job uaa
  uaa/0 (canary) (00:00:47)
Done                    1/1 00:00:47

Updating job api
  api/0 (canary) (00:01:50)
Done                    1/1 00:01:50

Updating job dea
  dea/0 (canary) (00:01:21)
Done                    1/1 00:01:21
</pre>

## Running example application on Cloud Foundry ##

You can now target and push your first example application, as per the promise at the top of the page.

First, target the Cloud Foundry, create an Organization, and create the first Space within that organization:

<pre class="terminal">
$ gem install cf
$ cf target api.mycloud.com
$ cf login admin
Password> ******

$ cf create-org demo
Creating organization demo... OK
Switching to organization demo... OK
There are no spaces. You may want to create one with create-space.

$ cf create-space development
Creating space development... OK
Adding you as a manager... OK
Adding you as a developer... OK
Space created! Use `cf switch-space development` to target it.

$ cf switch-space development
Switching to space development... OK

target: http://api.mycloud.com
organization: demo
space: development
</pre>

Finally, clone an example application that does not require database services and push it to deploy:

<pre class="terminal">
$ git clone https://github.com/cloudfoundry-community/cf_demoapp_ruby_rack.git
$ cd cf_demoapp_ruby_rack
$ bundle
$ cf push
cf push
Using manifest file manifest.yml

Custom startup command> <strong>none</strong>

Creating hello... OK

1: hello
2: none
Subdomain> <strong>hello</strong>

1: mycloud.com
2: none
Domain> <strong>mycloud.com</strong>

Creating route hello.mycloud.com... OK
Binding hello.mycloud.com to hello... OK
Uploading hello... OK
Starting hello... OK
-----> Downloaded app package (4.0K)
Installing ruby.
-----> Using Ruby version: ruby-1.9.2
-----> Installing dependencies using Bundler version 1.3.2
       Running: bundle install --without development:test --path vendor/bundle --binstubs vendor/bundle/bin --deployment
       Fetching gem metadata from https://rubygems.org/..........
       Installing rack (1.5.2)
       Using bundler (1.3.2)
       Your bundle is complete! It was installed into ./vendor/bundle
       Cleaning up the bundler cache.
-----> Uploading staged droplet (24M)
-----> Uploaded droplet
Checking hello...
Staging in progress...
  0/1 instances: 1 starting
  0/1 instances: 1 starting
  0/1 instances: 1 starting
  1/1 instances: 1 running
OK


$ open hello.mycloud.com
Hello World!
</pre>
