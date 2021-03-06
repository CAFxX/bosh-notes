- State: finished
- Docs: http://bosh.io/docs/runtime-config#addons

# Addons

Majority of operators want to make sure that *all* VMs that BOSH creates and starts have some software installed.

Here are few examples of such software:

- misc security agents (tripwire*, qualys agent*)
- antivirus (clamav, mcafee*, symantec*)
  - should not be stopped
  - should be safely
  - compilation vms?
  - upgrade storms? since antivirus might update itself continiously
- monitoring agents (sensu, nagios)
- logging (metron agent)
- ipsec

## Proposal

Define a separate configuration file e.g. `config.yml`:

```yaml
releases:
- {name: ipsec, version: 29}

addons:
- name: ipsec
  tags:
    include_on: all
    exclude_on: loggregator
  jobs:
  - {name: ipsec_agent, release: ipsec}
  - {name: ipsec_openswan, release: ipsec}
  properties:
    ipsec: {shared_key: UYasbvh872f}
- name: metron
  jobs:
  - {name: metron_agent, release: loggregator}
  properties:
    metron_agent:
      syslog_drain: 0.public.syslog_ingestor.logsearch_release
      # resolves to something like this. Can links do ports as well?!?
      # syslog_drain: 155.234.155.234:5514
```

```
$ bosh update runtime-config ./config.yml
$ bosh deploy my-redis-cluster-v1.yml
```

Note: This is somewhat similar to `update cloud-config` command but in this case config contains generic properties.

Given above configuration the following jobs will be colocated on all your VMs including redis VMs:

- metron_agent
- ipsec_agent
- ipsec_openswan

## Resolutions

When release versions are resolved:

- when running update config command

When links are resolved:

- when updating config command
  - problem with bootstrapping?
  - updating: update logsearch, update config.yml, update all other deployments.
- when deploying
  - updating: update logsearch, update all other deployments (just like normal links)

## Stories

[see `addons` label in Tracker]

# TBD

- persistent disk
- networks?
  - especially security group settings might be a problem? What if your co-located thing needs ports which are locked down?
- ways to exlude co-location or limit co-location to certain vms?
  - e.g. jobs in cf-release already bring the metron agent, but you want it for all other vms.
  - tags on the jobs in their deployment manifests?
