# ansible-fluentd

Fork of balanced-ops/ansible-fluentd.

Add in vars for secure_forward

Example:

fluentd_shared_key: randomstring
fluentd_host: 10.200.200.200
fluentd_port: 9000

[![Build Status](https://travis-ci.org/balanced-ops/ansible-fluentd.svg?branch=master)](https://travis-ci.org/balanced-ops/ansible-fluentd)

This ansible role installs the td-agent, the distribution of [fluentd](http://fluentd.org/).

This playbook was tested on Ubuntu 12.04 x86_64.

## Features

* Install fluentd
* ~~Uses monit to monitor fluentd~~ TODO: [Issue #4](https://github.com/balanced-ops/ansible-fluentd/issues/4)

## Running playbook

## Example setup

At Balanced we use fluentd as a log transport and aggregator service. It runs
locally on each node to transport logs and the on a cluster of aggregator
machines to distribute the logs for processing and archiving.

Our setup basically looks like `/var/log/some.log -> local fluentd -> central fluentd -> various data services`

We use the tail file plugin for moving data from the remote nodes because it is
most resilient to failure. If the fluentd service crashes on the node or the
aggregator it does not interrupt anything.

You can setup fluentd on the node like this

```yaml
fluentd_matches:
  - priority: 99
    name: forwarding
    options:
      servers: [10.20.3.11, 10.20.3.12, 10.20.3.13]

fluentd_in_tail_files:
  - name: debug.log
    path: /var/log/td-agent/debug.log
    format: json
```

This will setup fluentd to transport any log entries from
`/var/log/td-agent/debug.log` to any of the servers specified in the
`fluentd_matches` section.

On the aggregator nodes we configure fluentd like this

```yaml
fluentd_services:
  - name: balanced-integration
    pattern: 'balanced-integration.**'
    stores:
      s3:
      file:
      kafka:
```

This will ensure that any message that comes in tagged with `balanced-integration.*`
(that's a wildcard btw) will be stored into s3, locally in a file and fed into
a kafka cluster.

{{ td-agent-groups }} gives permissions to td-agent to access the log files

## Testing

`vagrant up`

It should provision two servers and a client. The client should log to both servers.

LICENSE
------
The source code is licensed under GPL v3.
