# Network Configuration

## Log Drain Blacklists

The loggregator component allows application log drains.
An application developer deploying their app to Cloud Foundry
can bind a drain to a log analysis service.

If the application developer attempted to bind that drain to
an internal Cloud Foundry component (if they knew the IP address),
it could affect your cloud performance.

You can blacklist the range of IP addresses that you are allocating
to Cloud Foundry components in your manifest.

Add the loggregator.blacklisted\_syslog\_ranges property to your manifest
with one or more ip ranges with a starting and ending ip addresses.
For example:

```
properties:

  ... other properties ...

  loggregator:
    blacklisted_syslog_ranges:
    - start: 10.10.16.0
      end: 10.10.31.255
    - start: 10.10.80.0
      end: 10.10.95.255

```

