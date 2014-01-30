---
title: Destroying Deployments on AWS
---

If you have deployed Cloud Foundry, BOSH and MicroBOSH on AWS, you are incurring cost for every hour you are using the servers. To destroy the servers:

Delete the CF deployment:

<pre class="terminal">
  bosh delete deployment tutorial
</pre>

Repoint the target to the MicroBOSH server and destroy the BOSH server. Replace the IP address below with the one you created for the BOSH server:

<pre class="terminal">
  bosh target 54.204.16.249
  bosh delete deployment bosh**
</pre>

Repoint the target to the MicroBOSH server and destroy the BOSH server. Replace the IP address below with the one you created for the BOSH server:

<pre class="terminal">
 bosh micro delete
</pre>

Now you can de-allocate the Elastic IPs that were created for CF, BOSH and MicroBOSH.

###[Return to Index](./index.html)