---
title: Deploying Cloud Foundry on AWS
---

There are several paths you can choose to deploy Cloud Foundry with BOSH on AWS.

**Note**: This tutorial deploys Cloud Foundry to the VPC flavor of AWS, rather than
the explicit EC2 flavor of AWS.
This is the production choice for running Cloud Foundry currently.
There are two reasons:

* Additional security offered by network isolation
* Static IPs are used for service discovery

Another way to configure service discovery is to use BOSH internal DNS feature. The PowerDNS implemented
in BOSH has a single point of failure in its backend PostgreSQL server. Therefore this method
is not appropriate for production and is **not** documented here.

## <a id='bootstrap-vpc'></a>Bootstrap on AWS VPC ##

The BOSH and Cloud Foundry Runtime teams maintain a bootstrapping tool to deploy
Cloud Foundry within AWS VPC.

Follow the [tutorial](./bootstrap-aws-vpc.html).

This is the solution Pivotal uses to deploy and operate the [hosted Cloud
Foundry solution](http://run.pivotal.io).

## <a id='low-level-ec2'></a>Step-by-step on AWS VPC ##

To learn many of the steps of deploying Cloud Foundry on BOSH by
following this
low-level, step-by-step tutorial for AWS EC2.

Follow the [tutorial](./aws_steps.html).

## <a id='low-level-ec2'></a> Bosh Bootstrap ##

There is a community tool,  [bosh-bootstrap](https://github.com/cloudfoundry-community/bosh-bootstrap "cloudfoundry-community/bosh-bootstrap · GitHub"), that automates most of the
steps in the "Deploy MicroBOSH to AWS" part of the tutorials above.