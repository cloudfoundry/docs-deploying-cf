---
title: Deployment of BOSH on AWS
---

This step leverages the new Elastic IP created in [Configuring AWS for BOSH](./configure_aws_bosh.html), as well as the Security Group and Key Pair file that we created in [Configuring AWS for Micro BOSH](./configure_aws_micro_bosh.html), to deploy a BOSH server on AWS in four steps:

**Note:** CF may be deployed straight from Micro BOSH, but this is typically not considered a reliable method and so it is not discussed here.


1. Create Directory Structure

2. Create BOSH Deployment Manifest

3. Locate the correct BOSH Stemcell

4. Deploy Manifest

The original source of this information was the [cloud foundry blog post](http://blog.cloudfoundry.com/2012/09/06/deploying-to-aws-using-cloud-foundry-bosh/).

### Obtain and Upload Release

Upload the most recent generic release:

<pre class="terminal">
bosh upload release https://s3.amazonaws.com/bosh-jenkins-artifacts/release/bosh-1274.tgz</td>
</pre>


### Create Directory Structure

Create a standard place to store the Deployment Manifest on your
local computer. You should still be targeting the Micro BOSH server:

<pre class="terminal">
  mkdir -p ~/bosh-workspace/deployments/bosh
  cd ~/bosh-workspace/deployments/bosh
  touch bosh.yml
</pre>


**Locate the correct BOSH Stemcell**

It is important to deploy the correct AWS image that is compatible with the version of BOSH you will be installing.

To find and obtain the current BOSH stemcell, navigate to `http://bosh_artifacts.cfapps.io`, right click **Download**, and copy the link for the “bosh (aws xen ubuntu)” tarball file (.tar.gz).  An example of the link URL is:

[https://s3.amazonaws.com/bosh-jenkins-artifacts/bosh-stemcell/aws/bosh-stemcell-1274-aws-xen-ubuntu.tgz](https://s3.amazonaws.com/bosh-jenkins-artifacts/bosh-stemcell/aws/bosh-stemcell-1274-aws-xen-ubuntu.tgz)

![image alt text](ec2/image_25.png)

Upload the latest stemcell of BOSH onto the Micro BOSH server:

<pre class="terminal">
  bosh upload stemcell https://s3.amazonaws.com/bosh-jenkins-artifacts/bosh-stemcell/aws/bosh-stemcell-1274-aws-xen-ubuntu.tgz
</pre>


After the upload completes, view the list of stemcells by calling:
<pre class="terminal">
  bosh stemcells
</pre>


**Create BOSH Deployment Manifest**

Now let’s review what the contents of the bosh.yml deployment
manifest file should include. Notice the first comment in the
example file.  You'll need to replace that uuid with the one for the
results of the command:

<pre class="terminal">
  bosh status
</pre>

~~~yaml
name: bosh

# Execute the "bosh status" command to obtain the director_uuid.
director_uuid: adf6af65-8b61-4b5f-b43a-44f3f662ec51

release:
  name: bosh
  version: latest

compilation:
  workers: 3
  network: default
  reuse_compilation_vms: true
  cloud_properties:
    instance_type: m1.small

update:
  canaries: 1
  canary_watch_time: 3000-120000
  update_watch_time: 3000-120000
  max_in_flight: 4
  max_errors: 1

networks:
  - name: elastic
    type: vip
    cloud_properties: {}
  - name: default
    type: dynamic
    cloud_properties:
      security_groups:
        - bosh # This needs to be changed if you didn’t call the security group “bosh”

resource_pools:
  - name: medium
    network: default
    size: 1
    stemcell:
      name: bosh-aws-xen-ubuntu
      version: latest
    cloud_properties:
      instance_type: m1.medium

jobs:
  - name: bosh
    template:
    - powerdns
    - nats
    - postgres
    - redis
    - director
    - blobstore
    - registry
    - health_monitor
    instances: 1
    resource_pool: medium
    persistent_disk: 20480
    networks:
      - name: default
        default: [dns, gateway]
      - name: elastic
        static_ips:
          - 23.21.249.15 # CHANGE: Elastic IP #2

properties:
  env:

  postgres: &bosh_db
    user: postgres
    password: postges
    host: 0.bosh.default.bosh.microbosh
    listen_address: 0.bosh.default.bosh.microbosh
    database: bosh

  dns:
    address: 23.21.249.15 # CHANGE: Elastic IP #2
    db: *bosh_db
    user: powerdns
    password: powerdns
    database:
      name: powerdns
    webserver:
      password: powerdns
    replication:
      basic_auth: replication:zxKDUBeCfKYX
      user: replication
      password: powerdns
    recursor: 54.204.16.249 # CHANGE: microBOSH IP address, Elastic IP #1

  redis:
    address: 0.bosh.default.bosh.microbosh
    password: redis

  nats:
    address: 0.bosh.default.bosh.microbosh
    user: nats
    password: nats

  director:
    name: bosh
    address: 0.bosh.default.bosh.microbosh
    db: *bosh_db

  blobstore:
    address: 0.bosh.default.bosh.microbosh
    agent:
      user: agent
      password: agent
    director:
      user: director
      password: director

  registry:
    address: 0.bosh.default.bosh.microbosh
    db: *bosh_db
    http:
      user: registry
      password: registry

  hm:
    http:
      user: hm
      password: hm
    director_account:
      user: admin
      password: admin
    event_nats_enabled: false
    email_notifications: false
    tsdb_enabled: false
    pagerduty_enabled: false
    varz_enabled: true

  aws:
    access_key_id: {{access key goes here}}
    secret_access_key: {{secret access key goes here}}
    default_key_name: bosh
    region: us-east-1
    default_security_groups: ["bosh"]
~~~

Save your changes to the file.

### Deploy Manifest

Everything is now in place to use the deployment manifest you have created and deploy BOSH to AWS (the first inception). Let’s now do this.

Enter the deployments folder you created earlier:

<pre class="terminal">
  cd ~/bosh-workspace/deployments
</pre>


Select the deployment you called "bosh" in the first section of [Configuring AWS for BOSH](./configure_aws_bosh.html):

<pre class="terminal">
bosh deployment bosh/bosh.yml
</pre>


Deploy the BOSH:

<pre class="terminal">
  bosh deploy
</pre>

If the deployment fails, clean it up before trying again. This
command takes some time. You will also have several instances running
in EC2 while this happens. These extra instances are used to
distribute the compilation of packages needed for your BOSH virtual
machine. You can control the number of compilation workers in the
bosh.yml file we created above.

<pre class="terminal">
bosh delete deployment bosh
</pre>


Log into the new BOSH server and replace the IP address you created
in Step 4 with the one below:

<pre class="terminal">
bosh target 23.21.249.15
</pre>

The default username and password are "admin" and “admin”.

If the deployment ran successfully, you now have a BOSH instance deployed onto AWS.

Note a few things in this image:

![image alt text](ec2/image_26.png)

1. **bosh** - The name of the instance matches the deployment name.

2. **us-east-1a** - The availability zone in the deployment manifest

3. **bosh** - The name of the key pair

4. **bosh** - The name of the Security Group

5. **23.21.249.15**- The Elastic IP address that we created, which is also the external IP address for the BOSH server

###Go on to [Configuring AWS for CF](./configure_aws_cf.html) or [Return to Index](./index.html)

