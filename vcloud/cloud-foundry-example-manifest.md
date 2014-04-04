---
title:	Cloud Foundry Example Manifest
---

Here is an example manifest for deploying Cloud Foundry on vCloud. Put the following in a file called cf.yml.erb

<pre class='terminal'>

<%#
  The following are the parameters you need to fill to generate a CF manifest file. Here is an example.
  Note that you need to fill those parameters according to your vCloud environment.
  The parameters in this example will NOT work for your vCloud environment.
#%>
<%
  deployment_name                  = "cf_oss"                               # Name of this deployment
  director_uuid                    = "566c47b3-ca85-47e2-95f6-32d095747acf" # UUID of the Micro BOSH / BOSH being used to deploy
  cf_release_name                  = "cf"                                   # The name of the release to be deployed
  cf_release_version               = "147"                                  # The version of the release to be deployed
  vapp_name                        = ""                                     # The name of the vApp into which all VMs are placed. To make a new vApp for each VM, leave this blank.
  ip_range                         = "192.168.109.0/24"                     # The total IP range for the network where the instance is placed
  gateway                          = "192.168.109.1"                        # The gateway for the network
  dns                              = "[192.168.109.15, 8.8.8.8]"            # The DNS address for the VMs. This should include the jumpbox IP first for internal name lookup
  static_ip_range                  = "[192.168.109.82 - 192.168.109.102, 192.168.109.119]" # The range of static IPs assigned below
  reserved_ip_range                = "[192.168.109.2 - 192.168.109.81, 192.168.109.126 - 192.168.109.254]" # The range of IPs to exclude
  network_name                     = "164-935-default-routed"               # The name of the network to use
  stemcell_name                    = "bosh-vcloud-esxi-ubuntu"              # The name of the stemcell to use
  stemcell_version                 = "0000"                                 # The version of the stemcell being used
  ha_proxy_ip                      = "192.168.109.119"                      # The static IP for HA_Proxy
  nats_ip                          = "192.168.109.82"                       # The static IP for NATS
  health_manager_ip                = "192.168.109.83"                       # The static IP for Health Manager
  nfs_server_ip                    = "192.168.109.84"                       # The static IP for the NFS server
  ccdb_ip                          = "192.168.109.85"                       # The static IP for Cloud Controller DB
  cloud_controller_ip              = "192.168.109.86"                       # The static IP for the Cloud Controller
  router_ip                        = "192.168.109.87"                       # The static IP for the router
  syslog_ip                        = "192.168.109.88"                       # The static IP for the syslog server
  collector_ip                     = "192.168.109.89"                       # The static IP for the collector
  uaadb_ip                         = "192.168.109.90"                       # The static IP for the UAA DB
  uaa_ip                           = "192.168.109.91"                       # The static IP for the UAA
  login_ip                         = "192.168.109.92"                       # The static IP for the Login server
  consoledb_ip                     = "192.168.109.93"                       # The static IP for the Console DB
  loggregator_ip                   = "192.168.109.94"                       # The static IP for the Loggregrator server
  loggregator_trafficcontroller_ip = "192.168.109.95"                       # The static IP for the Loggregator Traffic Controller
  

  # The login server communicates with the UAA over https. Change this setting to http if your test deployment does not use a trusted SSL certificate at the CF api endpoint
  # This includes if you're using a self-signed cert.
  # Note: An update in the login server in March 2014 will allow login to trust a self-signed cert. You can use https for releases later than this date.
  uaa_login_http_s                 = "http"

  # The wildcard domain name for the instance. To simplify this deployment we use this domain for both system and apps.
  # xip.io provides a very useful free service providing free wildcard dns for IP addresses.
  wildcard_domain_name             = "192.240.155.234.xip.io"               

  # The following certificate / private key pair can be self-signed to simplify the initial deployment.
  #
  # Use the following commands on the jumpbox to generate these
  # openssl genrsa -des3 -passout pass:x -out server.pass.key 2048
  # openssl rsa -passin pass:x -in server.pass.key -out server.key
  # rm server.pass.key
  # openssl req -new -key server.key -out server.csr
  # openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
  # 
  # Paste the resulting cert and private key below

  domain_public_cert = <<-END_CERT
    -----BEGIN CERTIFICATE-----
    MIIDvDCCAqQCCQDhBLWaOceNHzANBgkqhkiG9w0BAQUFADCBnzELMAkGA1UEBhMC
    VVMxEzARBgNVBAgMCkNhbGlmb3JuaWExFjAUBgNVBAcMDVNhbiBGcmFuY2lzY28x
    DzANBgNVBAoMBlZNd2FyZTEMMAoGA1UECwwDRW5nMSEwHwYDVQQDDBgqLjE5Mi4y
    NDAuMTU1LjIzNC54aXAuaW8xITAfBgkqhkiG9w0BCQEWEmttdXJwaHlAdm13YXJl
    LmNvbTAeFw0xNDAzMjcyMzUzNDNaFw0xNTAzMjcyMzUzNDNaMIGfMQswCQYDVQQG
    EwJVUzETMBEGA1UECAwKQ2FsaWZvcm5pYTEWMBQGA1UEBwwNU2FuIEZyYW5jaXNj
    bzEPMA0GA1UECgwGVk13YXJlMQwwCgYDVQQLDANFbmcxITAfBgNVBAMMGCouMTky
    LjI0MC4xNTUuMjM0LnhpcC5pbzEhMB8GCSqGSIb3DQEJARYSa211cnBoeUB2bXdh
    cmUuY29tMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAuLwjoKwhv5f5
    PlOSI6/5fHIEbMP8mbZOjDNRIgsNmXItxdbpX136+SyvQV7yi+p/9Hcd0j5X+tnT
    E1O53pEL3zkG2025x6mQqwjYLXWQD39IN8fbq+tmK5jVSd61dO++BfHJ762mv/Gn
    HAUOPCpGGOhVlrNxNGUHx67FsbO4Ej57MmqZu+JGdw7+7KccOIyI1gdXALUr+8K/
    OL8MBbvU/6etqcqcCNvjO0lBDVhVVtA/ejxhNNv7qHExkrZr+7CIYyr8D8pfvLHJ
    6hIEkmtiDVSWy9PyNfvVZG5PlyYTpMHx6Vqutzkxap9JnZE9oBTbTfBdxos/5KJy
    HoUQkTO6MQIDAQABMA0GCSqGSIb3DQEBBQUAA4IBAQCu3eZiXJZTUlQT9VYCNoMO
    +166LZQCgvIXs2tZOGwrUGSd45RX/kOPCYb3ufKlHuj5s+tWX5JPrOA92qGDReCN
    urngFSZD/CXqOmyS5kUNZMnOVhHOFXcFeS8W1DrOUgnrdqkHtltuATXpGdEdAlV/
    EUyFju1uyZFw3zBtW7eIun6zTvD4omUkiiLCuLCo7CS7RXf3KUNGIboTIaeQetLl
    A6lfLf8ogH3OAFX5a4iyNZu7ys+2feSzlWgZGKiLdPQWpdt8DV2v4FoZGU+7hR7L
    JKQqCKWI445CT5oGDZdeFAPe/cw+9Xe3wZaj8QpmzviweUABD4XTJoeZ5MIjePFw
    -----END CERTIFICATE-----
    END_CERT

  domain_private_key = <<-END_PRIVATE_KEY
    -----BEGIN RSA PRIVATE KEY-----
    MIIEogIBAAKCAQEAuLwjoKwhv5f5PlOSI6/5fHIEbMP8mbZOjDNRIgsNmXItxdbp
    X136+SyvQV7yi+p/9Hcd0j5X+tnTE1O53pEL3zkG2025x6mQqwjYLXWQD39IN8fb
    q+tmK5jVSd61dO++BfHJ762mv/GnHAUOPCpGGOhVlrNxNGUHx67FsbO4Ej57MmqZ
    u+JGdw7+7KccOIyI1gdXALUr+8K/OL8MBbvU/6etqcqcCNvjO0lBDVhVVtA/ejxh
    NNv7qHExkrZr+7CIYyr8D8pfvLHJ6hIEkmtiDVSWy9PyNfvVZG5PlyYTpMHx6Vqu
    tzkxap9JnZE9oBTbTfBdxos/5KJyHoUQkTO6MQIDAQABAoIBAFGqemWRMuosGPdA
    op48MSKelO4wRf795QN9vCQ8lqp7G1kWhNywAz8cTe2sN7U62Y4NCpXjEanHmdQ1
    czm9DW6FG07fsX1erKGvq0GNcz4mmppuM+Jwkh471i5t0fH7+hlOpmLadZjtD18H
    rR9T4OEp9IxGj4kGEMZpsOO5+2m0jP0O5Y3pgffYoYFHWzgX2inivXrQeAYXXo3N
    O7J0s5CbZXhMzM+xaR46vcQLVos8+OLlW3Xl2XfAe/rylASqnijmN+u67zTQPp2A
    dojjr8O+tU5cniTEbqfKYGVFnx7u//he+rN/qGf2azItEofj4RVSqlrQICicoU3V
    LAHgXTECgYEA4jqBKX1PlsM8qz2QHlM5+6G2AZqJaz7X6cb8pj9G83gNRt+ZYP/K
    QgbffUVewc5EGjVey1gzbPUjHS5Srnp+dgmvEWqtLfpWUKP7maqMaj5dFabEgX9X
    /ColeQAPjbvwYSSJtkyrL3nXHvQd4Eq7CC9mZN/6bz0lA0kzll/OBa8CgYEA0Qu9
    usddSCqgcaOhboIVWbEuhl9wMKa58liGebZWAeTHw/XkszPH288d/HwCh+rqVmYj
    5msn37IfEummqA6n/b98HxQ2OkeKRlzAY4rKEtodIxVrynnlPay4hm6Zyfpdt0zC
    qgATijJkQVkKI0oOAI/I7HJCrXHUgTnZ9h58Fh8CgYBWud7yLNvqDAaiDwPE3FsK
    IEBJ9RhhSMI1GNeaU/+7LnbIiMef6+95yHC88W8WFSD+ex9QDQwJ5SAE+9Eumj8I
    uUWoA6FIUwPr/jFiA4O45xeASWJj0pHEVdPvwxozV60bUIqKnHGzzZ2ufB9H8N4q
    kSFL4qF7K5GY5OMl7qxoeQKBgCtleZSdsIK7vqT4qBmNzarZ+mOQynR/GBj0Qa5g
    qMgp20KV+E0vUa0S+RGiGNBodw9KkudRlWx9yK+fa6Z1rHAj4Tt+cad1lIH43UOM
    21hAiU3wM3lMBsff5EqcCTcBz5SuzbaG34eP4HokZtNemzuIndhf+/GPsOLGxLWw
    LGhXAoGAQyEPTaNH8NwShZQ7x9/5susJQVYh7gqjfZCnW4pJEUgEeHWNqGDzAAHh
    aO7OZ4nkmHOEoy7iUQiFXWjhrO12B0dbjf8fkJfJU7hdEdOh+WE4JaxD/8hMls4+
    6pwI6EHLmkSW4ScHj8o3/i/qrfD7dHfnyHO15Z/oEabg/Cj+T2s=
    -----END RSA PRIVATE KEY-----
    END_PRIVATE_KEY


  # The following public key / private key pair are used for signing and validating
  # the OAuth tokens from the UAA
  # These should be generated using:
  # 
  # openssl genrsa -des3 -out server.key 1024
  # mv server.key server.key.protected && openssl rsa -in server.key.protected -out server.key
  # openssl rsa -in server.key -pubout > server.pub
  #
  # Paste the resulting public and private keys below

  jwt_public_key = <<-END_PUBLIC_KEY
    -----BEGIN PUBLIC KEY-----
    MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDduW/tIISShh5+Hlx36P5VGUX4
    ANQNkTHZwF+40RmRUvOk+GMCncCNQz8l17EqmNJpjUkE9+7N/4zFwUZ+L/W+d+9A
    pWGtraHQxaNGufndysj8IBEF2FGp1KSe+ur3Ctw/0a8gq3mrhiZD1dsMkGWFrd8R
    CqFcu9/3Olp6ittLfwIDAQAB
    -----END PUBLIC KEY-----
    END_PUBLIC_KEY

  jwt_private_key = <<-END_PRIVATE_KEY
    -----BEGIN RSA PRIVATE KEY-----
    MIICXAIBAAKBgQDduW/tIISShh5+Hlx36P5VGUX4ANQNkTHZwF+40RmRUvOk+GMC
    ncCNQz8l17EqmNJpjUkE9+7N/4zFwUZ+L/W+d+9ApWGtraHQxaNGufndysj8IBEF
    2FGp1KSe+ur3Ctw/0a8gq3mrhiZD1dsMkGWFrd8RCqFcu9/3Olp6ittLfwIDAQAB
    AoGAZA0akZE74XZ96gE/Tqino7Ts2tVc2uZq7UyepSJN/ELHSOkAnJyc1+HBbA0h
    mAwv3otvqLtMWk53soDdk3GG3N3rYGlgPqhd1sjKRX8fANN40Q6B/gPP1dM3Mc1C
    n0xsKfFYYDNJY7t4Y2oW8l7iRb6CMtKpKZbgPSBXNvCR7AkCQQDyad5g9q47hFxA
    3iQ5UG3yeUtUhN/NTD45tjb2glCjhF8t5H8v8YlerTg/1b+B5pd0i+qnVIojeEFC
    ZqIwTr31AkEA6ia4v49u1Tgz24IWXNqkIpH6VGPC5IbdYuojdBVMym5E0U4Muz0k
    QBLdSWPM7rzwy/wHEdnNC9DPVfxRRSsnIwJAGLg5CBQ/oiwWKDs+4GVWQOKjjuPZ
    2pqKweHV6v9Q78vA1PI3EhGEW5Y4ZTILzFhSW30lGZkiWQmbRgUnRtvQvQJAEmDn
    r2F6uZGnwFr9llwy9eOvWmBaM8XCKrll/v6NAHaXQDZ4GVo7NixE4jXLKBH8dIZb
    p7MIvRyuqXkch+lTMQJBAI+B/9UhsSbgWNwVYmLTYimoJT6aHkG+7rRuIv1OR0fi
    Kbj/O3je0t7U4UraJdg2sLoTNGdn7ekw2mvRubDQOXE=
    -----END RSA PRIVATE KEY-----
    END_PRIVATE_KEY

  # A little code used to indent keys and certs correctly when inserted below
  # This should be used as follows:
  # name: <%= YamlLiteralIndenter(foo, "  ")
  # The output will be:
  # name: |
  #   <text from foo>
  #   <more text from foo>
  #
  class YamlLiteralIndenter
    def self.indent(text, indentation)
      "|\n" +                               # Start with the literal indicator, then go to the next line where indentation is important
      text.
        split("\n").                        # Split the block of text by newline
        map{|l| l.strip }.                  # For each line, remove any leading or trailing whitespace
        map{|line| indentation + line }.    # Add the indentation to the beginning of each stripped line
        join("\n")                          # Join all the lines together with newline
    end
  end
%>

---
name: <%= deployment_name %>
director_uuid: <%= director_uuid %>
releases:
- name: <%= cf_release_name %>
  version: <%= cf_release_version %>
networks:
- name: default
  subnets:
  - range: <%= ip_range %>
    gateway: <%= gateway %>
    dns: <%= dns %>
    static: <%= static_ip_range %>
    reserved: <%= reserved_ip_range %>
    cloud_properties:
      name: <%= network_name %>
resource_pools:
- name: ha_proxy
  stemcell:
    name: <%= stemcell_name %>
    version: '<%= stemcell_version %>'
  network: default
  size: 1
  cloud_properties:
    ram: 1024
    disk: 2048
    cpu: 1
% unless vapp_name.empty?
  env:
    vapp: <%= vapp_name %>
% end
- name: nats
  stemcell:
    name: <%= stemcell_name %>
    version: '<%= stemcell_version %>'
  network: default
  size: 1
  cloud_properties:
    ram: 1024
    disk: 2048
    cpu: 1
% unless vapp_name.empty?
  env:
    vapp: <%= vapp_name %>
% end
- name: health_manager
  stemcell:
    name: <%= stemcell_name %>
    version: '<%= stemcell_version %>'
  network: default
  size: 1
  cloud_properties:
    ram: 1024
    disk: 2048
    cpu: 1
% unless vapp_name.empty?
  env:
    vapp: <%= vapp_name %>
% end
- name: nfs_server
  stemcell:
    name: <%= stemcell_name %>
    version: '<%= stemcell_version %>'
  network: default
  size: 1
  cloud_properties:
    ram: 1024
    disk: 2048
    cpu: 1
% unless vapp_name.empty?
  env:
    vapp: <%= vapp_name %>
% end
- name: ccdb
  stemcell:
    name: <%= stemcell_name %>
    version: '<%= stemcell_version %>'
  network: default
  size: 1
  cloud_properties:
    ram: 1024
    disk: 2048
    cpu: 1
% unless vapp_name.empty?
  env:
    vapp: <%= vapp_name %>
% end
- name: cloud_controller
  stemcell:
    name: <%= stemcell_name %>
    version: '<%= stemcell_version %>'
  network: default
  size: 1
  cloud_properties:
    ram: 1024
    disk: 10240
    cpu: 1
% unless vapp_name.empty?
  env:
    vapp: <%= vapp_name %>
% end
- name: router
  stemcell:
    name: <%= stemcell_name %>
    version: '<%= stemcell_version %>'
  network: default
  size: 1
  cloud_properties:
    ram: 1024
    disk: 2048
    cpu: 1
% unless vapp_name.empty?
  env:
    vapp: <%= vapp_name %>
% end
- name: syslog
  stemcell:
    name: <%= stemcell_name %>
    version: '<%= stemcell_version %>'
  network: default
  size: 1
  cloud_properties:
    ram: 1024
    disk: 2048
    cpu: 1
% unless vapp_name.empty?
  env:
    vapp: <%= vapp_name %>
% end
- name: collector
  stemcell:
    name: <%= stemcell_name %>
    version: '<%= stemcell_version %>'
  network: default
  size: 1
  cloud_properties:
    ram: 1024
    disk: 2048
    cpu: 1
% unless vapp_name.empty?
  env:
    vapp: <%= vapp_name %>
% end
- name: uaadb
  stemcell:
    name: <%= stemcell_name %>
    version: '<%= stemcell_version %>'
  network: default
  size: 1
  cloud_properties:
    ram: 1024
    disk: 2048
    cpu: 1
% unless vapp_name.empty?
  env:
    vapp: <%= vapp_name %>
% end
- name: uaa
  stemcell:
    name: <%= stemcell_name %>
    version: '<%= stemcell_version %>'
  network: default
  size: 1
  cloud_properties:
    ram: 1024
    disk: 2048
    cpu: 1
% unless vapp_name.empty?
  env:
    vapp: <%= vapp_name %>
% end
- name: login
  stemcell:
    name: <%= stemcell_name %>
    version: '<%= stemcell_version %>'
  network: default
  size: 1
  cloud_properties:
    ram: 1024
    disk: 2048
    cpu: 1
% unless vapp_name.empty?
  env:
    vapp: <%= vapp_name %>
% end
- name: consoledb
  stemcell:
    name: <%= stemcell_name %>
    version: '<%= stemcell_version %>'
  network: default
  size: 1
  cloud_properties:
    ram: 1024
    disk: 2048
    cpu: 1
% unless vapp_name.empty?
  env:
    vapp: <%= vapp_name %>
% end
- name: dea
  stemcell:
    name: <%= stemcell_name %>
    version: '<%= stemcell_version %>'
  network: default
  size: 1
  cloud_properties:
    ram: 16384
    disk: 32768
    cpu: 2
% unless vapp_name.empty?
  env:
    vapp: <%= vapp_name %>
% end
- name: loggregator
  stemcell:
    name: <%= stemcell_name %>
    version: '<%= stemcell_version %>'
  network: default
  size: 1
  cloud_properties:
    ram: 1024
    disk: 2048
    cpu: 1
% unless vapp_name.empty?
  env:
    vapp: <%= vapp_name %>
% end
- name: loggregator_trafficcontroller
  stemcell:
    name: <%= stemcell_name %>
    version: '<%= stemcell_version %>'
  network: default
  size: 1
  cloud_properties:
    ram: 1024
    disk: 2048
    cpu: 1
% unless vapp_name.empty?
  env:
    vapp: <%= vapp_name %>
% end
compilation:
  workers: 5
  network: default
  cloud_properties:
    ram: 1024
    disk: 4096
    cpu: 2
update:
  canaries: 1
  canary_watch_time: 30000-300000
  update_watch_time: 30000-300000
  max_in_flight: 1
  max_errors: 2
jobs:
- name: ha_proxy
  template: haproxy
  instances: 1
  resource_pool: ha_proxy
  persistent_disk: 0
  networks:
  - name: default
    static_ips:
    - <%= ha_proxy_ip %>
  properties:
    networks:
      apps: default
    ha_proxy:
      timeout: 300
      ssl_pem: <%= YamlLiteralIndenter.indent(domain_public_cert + domain_private_key, "        ") %>
    router:
      servers:
        z1:
        - <%= router_ip %>
        z2: []
    syslog_aggregator:
      address: <%= syslog_ip %>
      port: 54321
  update:
    max_in_flight: 1
- name: nats
  template: nats
  instances: 1
  resource_pool: nats
  persistent_disk: 0
  networks:
  - name: default
    static_ips:
    - <%= nats_ip %>
  properties:
    networks:
      apps: default
    nats:
      user: nats
      password: 028d7bd275852ede9f24
      address: <%= nats_ip %>
      port: 4222
    syslog_aggregator:
      address: <%= syslog_ip %>
      port: 54321
  update:
    max_in_flight: 1
- name: health_manager
  template: health_manager_next
  instances: 1
  resource_pool: health_manager
  persistent_disk: 0
  networks:
  - name: default
    static_ips:
    - <%= health_manager_ip %>
  properties:
    cc:
      srv_api_uri: https://api.<%= wildcard_domain_name %>
    networks:
      apps: default
    nats:
      user: nats
      password: 028d7bd275852ede9f24
      address: <%= nats_ip %>
      port: 4222
    health_manager:
      intervals:
        giveup_crash_number: 4
    syslog_aggregator:
      address: <%= syslog_ip %>
      port: 54321
  update:
    max_in_flight: 1
- name: nfs_server
  template: debian_nfs_server
  instances: 1
  resource_pool: nfs_server
  persistent_disk: 102400
  networks:
  - name: default
    static_ips:
    - <%= nfs_server_ip %>
  properties:
    nfs_server:
      address: <%= nfs_server_ip %>
      network: <%= ip_range %>
  update:
    max_in_flight: 1
- name: ccdb
  template: postgres
  instances: 1
  resource_pool: ccdb
  persistent_disk: 2048
  networks:
  - name: default
    static_ips:
    - <%= ccdb_ip %>
  properties:
    db: ccdb
    ccdb:
      address: <%= ccdb_ip %>
      port: 2544
      db_scheme: postgres
      roles:
      - tag: admin
        name: admin
        password: f67fc7778aae83ae84ff
      databases:
      - tag: cc
        name: ccdb
        citext: true
  update:
    max_in_flight: 1
- name: cloud_controller
  template: cloud_controller_ng
  instances: 1
  resource_pool: cloud_controller
  persistent_disk: 0
  networks:
  - name: default
    static_ips:
    - <%= cloud_controller_ip %>
  properties:
    domain: <%= wildcard_domain_name %>
    system_domain: <%= wildcard_domain_name %>
    system_domain_organization: system
    app_domains:
    - <%= wildcard_domain_name %>
    ccng:
      external_host: api
      logging_level: debug
      uaa_resource_id: cloud_controller
      staging_upload_user: staging_upload_user
      staging_upload_password: a842a43502b787f25e9f
      bulk_api_password: 58c9a865ba760c5324f8
      db_encryption_key: 2ee20d59fb3c9c1a8a1d
      bootstrap_admin_email: admin
      tasks_disabled: true
      default_quota_definition: runaway
    ccdb_ng:
      address: <%= ccdb_ip %>
      port: 2544
      db_scheme: postgres
      roles:
      - tag: admin
        name: admin
        password: f67fc7778aae83ae84ff
      databases:
      - tag: cc
        name: ccdb
        citext: true
    networks:
      apps: default
    nfs_server:
      address: <%= nfs_server_ip %>
    uaa:
      url: <%= uaa_login_http_s %>://uaa.<%= wildcard_domain_name %>
      jwt:
        verification_key: <%= YamlLiteralIndenter.indent(jwt_public_key, "          ") %>
    login:
      url: <%= uaa_login_http_s %>://login.<%= wildcard_domain_name %>
    loggregator:
      trafficcontroller: <%= loggregator_trafficcontroller_ip %>
      incoming_port: 3456
    nats:
      user: nats
      password: 028d7bd275852ede9f24
      address: <%= nats_ip %>
      port: 4222
    syslog_aggregator:
      address: <%= syslog_ip %>
      port: 54321
  update:
    max_in_flight: 1
- name: router
  template: gorouter
  instances: 1
  resource_pool: router
  persistent_disk: 0
  networks:
  - name: default
    static_ips:
    - <%= router_ip %>
  properties:
    router:
      endpoint_timeout: 300
      status:
        user: router_status
        password: c13f90e568bdaa6c6c23
    loggregator:
      trafficcontroller: <%= loggregator_trafficcontroller_ip %>
      incoming_port: 3456
    nats:
      user: nats
      password: 028d7bd275852ede9f24
      address: <%= nats_ip %>
      port: 4222
    syslog_aggregator:
      address: <%= syslog_ip %>
      port: 54321
  update:
    max_in_flight: 1
- name: syslog
  template: syslog_aggregator
  instances: 1
  resource_pool: syslog
  persistent_disk: 8192
  networks:
  - name: default
    static_ips:
    - <%= syslog_ip %>
  properties:
    domain: <%= wildcard_domain_name %>
    nats:
      user: nats
      password: 028d7bd275852ede9f24
      address: <%= nats_ip %>
      port: 4222
    syslog_aggregator:
      address: <%= syslog_ip %>
      port: 54321
  update:
    max_in_flight: 1
- name: collector
  template: collector
  instances: 1
  resource_pool: collector
  persistent_disk: 0
  networks:
  - name: default
    static_ips:
    - <%= collector_ip %>
  properties:
    nats:
      user: nats
      password: 028d7bd275852ede9f24
      address: <%= nats_ip %>
      port: 4222
    syslog_aggregator:
      address: <%= syslog_ip %>
      port: 54321
  update:
    max_in_flight: 1
- name: uaadb
  template: postgres
  instances: 1
  resource_pool: uaadb
  persistent_disk: 8192
  networks:
  - name: default
    static_ips:
    - <%= uaadb_ip %>
  properties:
    db: uaadb
    uaadb:
      address: <%= uaadb_ip %>
      port: 2544
      db_scheme: postgresql
      roles:
      - tag: admin
        name: root
        password: d769054790a6432baaf2
      databases:
      - tag: uaa
        name: uaa
    networks:
      services: default
  update:
    max_in_flight: 1
- name: uaa
  template: uaa
  instances: 1
  resource_pool: uaa
  persistent_disk: 0
  networks:
  - name: default
    static_ips:
    - <%= uaa_ip %>
  properties:
    uaa:
      catalina_opts: -Xmx768m -XX:MaxPermSize=256m
      url: <%= uaa_login_http_s %>://uaa.<%= wildcard_domain_name %>
      jwt:
        signing_key: <%= YamlLiteralIndenter.indent(jwt_private_key, "          ") %>
        verification_key: <%= YamlLiteralIndenter.indent(jwt_public_key, "          ") %>
      cc:
        client_secret: 8a2b6658679c88fdc0d5
      admin:
        client_secret: 1accd5c45cf70d4c0242
      clients:
        login:
          id: login
          override: true
          autoapprove: true
          authorities: oauth.login
          authorized-grant-types: authorization_code,client_credentials,refresh_token
          scope: openid
          secret: 14f08354afcf36e6057c
        portal:
          id: portal
          override: true
          autoapprove: true
          authorities: scim.write,scim.read,cloud_controller.read,cloud_controller.write,password.write,uaa.admin,uaa.resource,cloud_controller.admin
          authorized-grant-types: authorization_code,client_credentials,password,implicit
          scope: openid,cloud_controller.read,cloud_controller.write,password.write,console.admin,console.support,cloud_controller.admin
          secret: d5b4e6be8a3d95706eab
          access-token-validity: 1209600
          refresh-token-validity: 1209600
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
          secret: 6677b7905ed885b8901c
      scim:
        user:
          override: false
        users:
        - admin|cdd0549963e8749b4a04|scim.write,scim.read,openid,cloud_controller.admin,dashboard.user,console.admin,console.support
        - system_services|d61bc2c89595476ff0b2|cloud_controller.admin
        - system_verification|afa5830d1ef8c3518996|scim.write,scim.read,openid,cloud_controller.admin,dashboard.user,console.admin,console.support
    ccdb:
      address: <%= ccdb_ip %>
      port: 2544
      db_scheme: postgres
      roles:
      - tag: admin
        name: admin
        password: f67fc7778aae83ae84ff
      databases:
      - tag: cc
        name: ccdb
        citext: true
    domain: <%= wildcard_domain_name %>
    db: uaadb
    uaadb:
      address: <%= uaadb_ip %>
      port: 2544
      db_scheme: postgresql
      roles:
      - tag: admin
        name: root
        password: d769054790a6432baaf2
      databases:
      - tag: uaa
        name: uaa
    networks:
      apps: default
    nats:
      user: nats
      password: 028d7bd275852ede9f24
      address: <%= nats_ip %>
      port: 4222
    syslog_aggregator:
      address: <%= syslog_ip %>
      port: 54321
  update:
    max_in_flight: 1
- name: login
  template: login
  instances: 1
  resource_pool: login
  persistent_disk: 0
  networks:
  - name: default
    static_ips:
    - <%= login_ip %>
  properties:
    domain: <%= wildcard_domain_name %>
    login:
      links:
        home: https://console.<%= wildcard_domain_name %>
        passwd: https://console.<%= wildcard_domain_name %>/password_resets/new
        signup: https://console.<%= wildcard_domain_name %>/register
      uaa_base: <%= uaa_login_http_s %>://uaa.<%= wildcard_domain_name %>
      uaa_certificate: <%= YamlLiteralIndenter.indent(domain_public_cert, "        ") %>
    uaa:
      clients:
        login:
          secret: 14f08354afcf36e6057c
    networks:
      apps: default
    nats:
      user: nats
      password: 028d7bd275852ede9f24
      address: <%= nats_ip %>
      port: 4222
    syslog_aggregator:
      address: <%= syslog_ip %>
      port: 54321
  update:
    max_in_flight: 1
- name: consoledb
  template: postgres
  instances: 1
  resource_pool: consoledb
  persistent_disk: 1024
  networks:
  - name: default
    static_ips:
    - <%= consoledb_ip %>
  properties:
    db: consoledb
    consoledb:
      address: <%= consoledb_ip %>
      port: 2544
      db_scheme: postgresql
      roles:
      - tag: admin
        name: root
        password: d50c5217a87ac68899b9
      databases:
      - tag: console
        name: console
    networks:
      services: default
  update:
    max_in_flight: 1
- name: dea
  templates:
  - name: dea_next
  - name: dea_logging_agent
  instances: 1
  resource_pool: dea
  persistent_disk: 0
  networks:
  - name: default
  properties:
    domain: <%= wildcard_domain_name %>
    dea_next:
      directory_server_protocol: https
      memory_mb: 16384
      memory_overcommit_factor: 1
      disk_mb: 32768
      disk_overcommit_factor: 1
      num_instances: 10
      stacks:
      - lucid64
    loggregator:
      trafficcontroller: <%= loggregator_trafficcontroller_ip %>
      incoming_port: 3456
      status:
        user: f8dab05b115082c38c0a
        password: 5c93945267a6f48f02cc
        port: 5768
    nats:
      user: nats
      password: 028d7bd275852ede9f24
      address: <%= nats_ip %>
      port: 4222
    syslog_aggregator:
      address: <%= syslog_ip %>
      port: 54321
  update:
    max_in_flight: 1
- name: loggregator
  template: loggregator
  instances: 1
  resource_pool: loggregator
  persistent_disk: 0
  networks:
  - name: default
    static_ips:
    - <%= loggregator_ip %>
  properties:
    loggregator:
      incoming_port: 3456
      outgoing_port: 8080
      status:
        user: f8dab05b115082c38c0a
        password: 5c93945267a6f48f02cc
        port: 5768
    cc:
      srv_api_uri: https://api.<%= wildcard_domain_name %>
    system_domain: <%= wildcard_domain_name %>
    uaa:
      jwt:
        verification_key: <%= YamlLiteralIndenter.indent(jwt_public_key, "          ") %>
    nats:
      user: nats
      password: 028d7bd275852ede9f24
      address: <%= nats_ip %>
      port: 4222
    syslog_aggregator:
      address: <%= syslog_ip %>
      port: 54321
  update:
    max_in_flight: 1
- name: loggregator_trafficcontroller
  template: loggregator_trafficcontroller
  instances: 1
  resource_pool: loggregator_trafficcontroller
  persistent_disk: 0
  networks:
  - name: default
    static_ips:
    - <%= loggregator_trafficcontroller_ip %>
  properties:
    loggregator:
      disableEmailDomainAuthorization: true
      trafficcontroller: <%= loggregator_trafficcontroller_ip %>
      incoming_port: 3456
      outgoing_port: 8080
      shared_secret: d328ef4e5b10a562ef7e
      servers:
        z1:
        - <%= loggregator_ip %>
    zone: z1
    ssl:
      skip_cert_verify: true
    cc:
      srv_api_uri: https://api.<%= wildcard_domain_name %>
    system_domain: <%= wildcard_domain_name %>
    nats:
      user: nats
      password: 028d7bd275852ede9f24
      address: <%= nats_ip %>
      port: 4222
    uaa:
      jwt:
        verification_key: <%= YamlLiteralIndenter.indent(jwt_public_key, "          ") %>
    syslog_aggregator:
      address: <%= syslog_ip %>
      port: 54321
  update:
    max_in_flight: 1

</pre>
