---
title: Deploying Cloud Foundry on AWS
---

There are several ways to deploy Cloud Foundry with BOSH on AWS.

The following methods describe deploying Cloud Foundry to the VPC flavor of AWS, rather than the explicit EC2 flavor of AWS.
We recommend these deployment methods in production environments for the following reasons:

* Additional security offered by network isolation
* Service discovery using Static IPs

<p class="note"><strong>Note</strong>: You can also configure service discovery
	to use the BOSH internal DNS feature. The PowerDNS implemented in BOSH is
	backed by PostgreSQL, with PostgreSQL acting as a single point of failure.
	For this reason, we do not recommend this choice in production
	environments.</p>

## <a id='bootstrap-vpc'></a>Bootstrap on AWS VPC ##

The BOSH and Cloud Foundry Runtime teams maintain a bootstrapping tool to deploy Cloud Foundry within AWS VPC.

Follow the [tutorial](./bootstrap-aws-vpc.html).

<p class="note"><strong>Note</strong>: We deploy the <a
	href="http://run.pivotal.io">hosted Cloud Foundry solution</a> using this
	method.</p>

## <a id='low-level-ec2'></a>Step-by-Step on AWS VPC ##

Follow this low-level, step-by-step tutorial for more details on deploying Cloud
Foundry on BOSH.

Follow the [tutorial](./aws_steps.html).

## <a id='bosh-bootstrap'></a>BOSH Bootstrap ##

The community tool,
[bosh-bootstrap](https://github.com/cloudfoundry-community/bosh-bootstrap),
automates most of the steps in the "Deploy MicroBOSH to AWS" sections of
the tutorials above.