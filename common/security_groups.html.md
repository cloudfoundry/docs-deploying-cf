---
title: Security Groups for Cloud Foundry
---

AWS and OpenStack offer Security Groups as a mechanism to restrict inbound traffic to servers.

The examples below show the Security Groups that are referenced in other sections of this documentation.

## <a id="ssh"></a>ssh

| Direction | Ether Type | IP Protocol | Port Range | Remote           |
|-----------|------------|-------------|------------|------------------|
| Egress    | IPv4       | Any         | -          | 0.0.0.0/0 (CIDR) |
| Ingress   | IPv4       | UDP         | 68         | 0.0.0.0/0 (CIDR) |
| Ingress   | IPv4       | ICMP        | -          | 0.0.0.0/0 (CIDR) |
| Egress    | IPv6       | Any         | -          | ::/0 (CIDR)      |
| Ingress   | IPv4       | TCP         | 22         | 0.0.0.0/0 (CIDR) |



## <a id="cf-public"></a>cf-public

| Direction | Ether Type | IP Protocol | Port Range | Remote           |
|-----------|------------|-------------|------------|------------------|
| Ingress    | IPv4       | TCP         | 443       | 0.0.0.0/0 (CIDR) |
| Ingress    | IPv4       | UDP         | 68        | 0.0.0.0/0 (CIDR) |
| Ingress    | IPv4       | TCP         | 80        | 0.0.0.0/0 (CIDR) |
| Egress     | IPv4       | Any         | -         | 0.0.0.0/0 (CIDR) |
| Egress     | IPv6       | Any         | -         | ::/0 (CIDR)      |


## <a id="cf-private"></a>cf-private

| Direction | Ether Type | IP Protocol | Port Range | Remote           |
|-----------|------------|-------------|------------|------------------|
| Egress    | IPv6       | Any         | -          | ::/0 (CIDR)      |
| Egress    | IPv4       | Any         | -          | 0.0.0.0/0 (CIDR) |
| Ingress   | IPv4       | UDP         | 68         | 0.0.0.0/0 (CIDR) |
| Ingress   | IPv4       | TCP         | 1-65535    | bosh             |
| Ingress   | IPv4       | UDP         | 3456-3457  | 0.0.0.0/0 (CIDR) |



