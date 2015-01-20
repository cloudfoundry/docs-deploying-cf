---
title: Troubleshooting
---


## <a id="ssh_to_vms"></a> SSHing to BOSH created VMS

If you need to debug a Cloud Foundry deployment you will probably need to log into one or more of the deployed VMs at some point. You can get the IP address using `bosh vms` . Then use `ssh` to connect to the VM. By default you can use the userid `vcap` with the password `c1oudc0w` . You can change the default password in the manifest file as described in the [micro BOSH deployment instructions](../../bosh/deploy_microbosh_to_vcloud.html)

Once you are logged into the target machine the Cloud Foundry components are typically deployed in `/var/vcap`. The following session demonstrates getting the VM IPs, logging into the UAA and showing the log.

<pre class='terminal'>

$ bosh vms
Deployment `cf_oss'

Director task 15

Task 15 done

+---------------------------------+---------+-------------------------------+-----------------+
| Job/index                       | State   | Resource Pool                 | IPs             |
+---------------------------------+---------+-------------------------------+-----------------+
| ccdb/0                          | running | ccdb                          | 192.168.109.85  |
| cloud_controller/0              | running | cloud_controller              | 192.168.109.86  |
| collector/0                     | running | collector                     | 192.168.109.89  |
| consoledb/0                     | running | consoledb                     | 192.168.109.93  |
| dea/0                           | running | dea                           | 192.168.109.120 |
| ha_proxy/0                      | running | ha_proxy                      | 192.168.109.119 |
| health_manager/0                | running | health_manager                | 192.168.109.83  |
| loggregator/0                   | running | loggregator                   | 192.168.109.94  |
| loggregator_trafficcontroller/0 | running | loggregator_trafficcontroller | 192.168.109.95  |
| login/0                         | running | login                         | 192.168.109.92  |
| nats/0                          | running | nats                          | 192.168.109.82  |
| nfs_server/0                    | running | nfs_server                    | 192.168.109.84  |
| router/0                        | running | router                        | 192.168.109.87  |
| syslog/0                        | running | syslog                        | 192.168.109.88  |
| uaa/0                           | running | uaa                           | 192.168.109.91  |
| uaadb/0                         | running | uaadb                         | 192.168.109.90  |
+---------------------------------+---------+-------------------------------+-----------------+

VMs total: 16
killian@ubuntu:~$ ssh vcap@192.168.109.91
vcap@192.168.109.91's password:
Linux 2c2112c9-4966-4f34-9f1d-4c18cf0517f1 3.0.0-32-virtual #51~lucid1-Ubuntu SMP Fri Mar 22 18:13:07 UTC 2013 x86_64 GNU/Linux
Ubuntu 10.04.4 LTS

Welcome to Ubuntu!
 * Documentation:  https://help.ubuntu.com/
Last login: Tue Apr  1 00:55:40 2014 from 192.168.109.15
vcap@2c2112c9-4966-4f34-9f1d-4c18cf0517f1:~$ cd /var/vcap/
vcap@2c2112c9-4966-4f34-9f1d-4c18cf0517f1:/var/vcap$ ls
bosh  data  jobs  micro  micro_bosh  monit  packages  sys
vcap@2c2112c9-4966-4f34-9f1d-4c18cf0517f1:/var/vcap$ less data/sys/log/uaa/uaa.log

</pre>


## <a id="cleaning_up"></a>Cleaning up a broken deployment

Often when you run into trouble doing a deployment, destroying the Micro BOSH, BOSH, and Cloud Foundry deployments and starting over is the simplest approach.

### <a id="deployment"></a>Deleting a deployment

To destroy a deployment, use `bosh delete deployment <deployment name>`. For example:
<pre class='terminal'>
bosh delete deployment cf_oss
</pre>


### <a id="microbosh"></a>Deleting Micro BOSH
To destroy a Micro BOSH, use `bosh micro delete`, as follows:

<pre class='terminal'>
bosh micro delete
</pre>


### <a id="vms"></a>Deleting remaining VMs and other artifacts

Sometimes some VMs may get stuck in the process of being created, and BOSH doesn't then delete them. If you specified a vApp name for the VMs in the manifest, you can use the vCloud Director UI ("My Cloud" tab) to stop and delete the vApp, including all VMs within it. If you didn't specify a vApp name you will have to delete each VM individually.

If you do specify a vApp name in the manifest file, make sure to change the name if a deployment fails for some reason. The vApp can get into an inconsistent state preventing future successful deployments.

Once you have removed all the VMs there may be some additional artifacts remaining, which you may have to clean up by hand. These include independent disks and / or ISOs.


If you want to start completely clean, you should also remove these:

* `bosh-deployments.yml` - in the `~/deployments` directory

<pre class='terminal'>
cd ~/deployments
rm bosh-deployments.yml - in the ~/deployments directory
</pre>


### <a id="example"></a>Sample session

The following shows an example of a session where the Cloud Foundry deployment is deleted, along with its associated Micro BOSH

<pre class='terminal'>
killian@ubuntu:~/deployments$ bosh status
Config
             /home/killian/.bosh_config

Director
  Name       bosh_micro_vchs
  URL        https://192.168.109.81:25555
  Version    1.0000.0 (release:125d9104 bosh:125d9104)
  User       admin
  UUID       438edfae-4dfe-4d0c-9098-cb2cd2adb3c9
  CPI        vcloud
  dns        enabled (domain_name: microbosh)
  compiled_package_cache disabled
  snapshots  disabled

Deployment
  Manifest   /home/killian/deployments/cf.yml
killian@ubuntu:~/deployments$ bosh vms
Deployment `cf_oss'

Director task 13

Task 13 done

+---------------------------------+---------+-------------------------------+-----------------+
| Job/index                       | State   | Resource Pool                 | IPs             |
+---------------------------------+---------+-------------------------------+-----------------+
| ccdb/0                          | running | ccdb                          | 192.168.109.85  |
| cloud_controller/0              | running | cloud_controller              | 192.168.109.86  |
| collector/0                     | running | collector                     | 192.168.109.89  |
| consoledb/0                     | running | consoledb                     | 192.168.109.93  |
| dea/0                           | running | dea                           | 192.168.109.120 |
| ha_proxy/0                      | running | ha_proxy                      | 192.168.109.119 |
| health_manager/0                | running | health_manager                | 192.168.109.83  |
| loggregator/0                   | running | loggregator                   | 192.168.109.94  |
| loggregator_trafficcontroller/0 | running | loggregator_trafficcontroller | 192.168.109.95  |
| nats/0                          | running | nats                          | 192.168.109.82  |
| nfs_server/0                    | running | nfs_server                    | 192.168.109.84  |
| router/0                        | running | router                        | 192.168.109.87  |
| saml_login/0                    | running | saml_login                    | 192.168.109.92  |
| syslog/0                        | running | syslog                        | 192.168.109.88  |
| uaa/0                           | running | uaa                           | 192.168.109.91  |
| uaadb/0                         | running | uaadb                         | 192.168.109.90  |
+---------------------------------+---------+-------------------------------+-----------------+

VMs total: 16
killian@ubuntu:~/deployments$ bosh delete deployment cf_oss

You are going to delete deployment `cf_oss'.

THIS IS A VERY DESTRUCTIVE OPERATION AND IT CANNOT BE UNDONE!

Are you sure? (type 'yes' to continue): yes

Director task 14
  Started deleting instances
  Started deleting instances: router/0
  Started deleting instances: nfs_server/0
  Started deleting instances: health_manager/0
  Started deleting instances: ha_proxy/0
  Started deleting instances: nats/0
  Started deleting instances: cloud_controller/0
  Started deleting instances: ccdb/0
  Started deleting instances: consoledb/0
  Started deleting instances: syslog/0
  Started deleting instances: loggregator_trafficcontroller/0
  Started deleting instances: uaa/0
  Started deleting instances: collector/0
  Started deleting instances: saml_login/0
  Started deleting instances: dea/0
  Started deleting instances: uaadb/0
  Started deleting instances: loggregator/0
     Done deleting instances: nats/0 (00:00:42)
     Done deleting instances: router/0 (00:01:12)
     Done deleting instances: ha_proxy/0 (00:01:37)
     Done deleting instances: uaa/0 (00:02:01)
     Done deleting instances: dea/0 (00:02:24)
     Done deleting instances: cloud_controller/0 (00:02:47)
     Done deleting instances: collector/0 (00:03:10)
     Done deleting instances: health_manager/0 (00:03:38)
     Done deleting instances: saml_login/0 (00:04:00)
     Done deleting instances: loggregator/0 (00:04:22)
     Done deleting instances

     Done deleting instances: loggregator_trafficcontroller/0 (00:09:40)
     Done deleting instances: consoledb/0 (00:10:01)
     Done deleting instances: uaadb/0 (00:10:26)
     Done deleting instances: ccdb/0 (00:10:44)
     Done deleting instances: syslog/0 (00:11:03)
     Done deleting instances: nfs_server/0 (00:11:21)

Removing deployment artifacts
  detach stemcells (00:00:00)
  detaching releases (00:00:00)
Done                    3/3 00:00:00

Deleting properties
  delete DNS records (00:00:00)
  destroy deployment (00:00:00)
Done                    0/0 00:00:00

Task 14 done

Started		2014-03-27 19:19:31 UTC
Finished	2014-03-27 19:30:52 UTC
Duration	00:11:21

Deleted deployment `cf_oss'
killian@ubuntu:~/deployments$
killian@ubuntu:~/deployments$
killian@ubuntu:~/deployments$ bosh micro delete

You are going to delete micro BOSH deployment `bosh_micro_vchs'.

THIS IS A VERY DESTRUCTIVE OPERATION AND IT CANNOT BE UNDONE!

Are you sure? (type 'yes' to continue): yes

Delete micro BOSH
  stopping agent services (00:00:01)
  unmount disk (00:00:11)
  detach disk (00:01:00)
  delete disk (00:00:05)
  delete VM (00:00:25)
  delete stemcell (00:00:02)
Done                    7/7 00:01:47
Deleted deployment 'bosh_micro_vchs', took 00:01:47 to complete

</pre>
