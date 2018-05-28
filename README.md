# os_patching

This module contains a set of tasks and custom facts to allow the automation of and reporting on operating system patching, currently restricted to Redhat derivatives.

The module is under active development and tasks to carry out RPM based OS patching have now been included.

## Table of contents

- [os_patching](#ospatching)
  - [Table of contents](#table-of-contents)
  - [Description](#description)
  - [Setup](#setup)
    - [What os_patching affects](#what-ospatching-affects)
    - [Beginning with os_patching](#beginning-with-ospatching)
  - [Usage](#usage)
  - [Reference](#reference)
  - [Limitations](#limitations)
  - [Development](#development)
  - [Contributors](#contributors)

## Description

Puppet tasks and bolt have opened up methods to integrate operating system level patching into the pupept workflow.  Providing automation of patch execution through tasks and the robust reporting of state through custom facts and PuppetDB.

If you're looking for a simple way to report on your OS patch levels, this module will show all updates which are outstanding, including which are related to security updates.  Do you want to enable self-service patching?  This module will use Puppet's RBAC and orchestration and task execution facilities to give you that power.

## Setup

### What os_patching affects

The module, when added to a node, creates a directory to cache patch data, installs a script to generate the cache and a cron job to run the script.  It also installs a dynamic custom fact which reports on the patching state of the server.

### Beginning with os_patching

Install the module using the Puppetfile, include it on your nodes and then use the provided tasks to carry out patching.

## Usage

Include the module:
```puppet
include os_patching
```

Run a basic patching task from the command line:
```bash
# puppet task show os_patching::patch_server

os_patching::patch_server - Carry out OS patching on the server, optionally including a reboot

USAGE:
$ puppet task run os_patching::patch_server reboot=<value> security_only=<value> <[--nodes, -n <node-names>] | [--query, -q <'query'>]>

PARAMETERS:
- reboot : Boolean
    Should the server reboot after patching has been applied?
- security_only : Boolean
    Limit patches to those tagged as security related?
```

## Reference

### Facts

Most of the reporting is driven off the custom fact `patchdata`, for example:

```puppet
facter -p patchdata.updates
{
  secnumupdates => 0,
  updatelist => {
    kernel.x86_64 => "3.10.0-862.3.2.el7",
    kernel-tools.x86_64 => "3.10.0-862.3.2.el7",
    kernel-tools-libs.x86_64 => "3.10.0-862.3.2.el7",
    postfix.x86_64 => "2:2.10.1-6.0.1.el7.centos",
    procps-ng.x86_64 => "3.3.10-17.el7_5.2",
    python-perf.x86_64 => "3.10.0-862.3.2.el7.centos.plus"
  },
  numupdates => 6
}
```

This shows there are 6 updates which can be applied to the server and none of them are tagged as security related.

The patchdata fact is dynamic, but it does source information from cache files, generated by a cron job once an hour (at a staggered interval per node).

### Task output

If there is nothing to be done, the task will report:

```puppet
{
  "date" : "Mon May 28 12:23:33 AEST 2018",
  "fqdn" : "example.lab.vm",
  "reboot" : "true",
  "logfile" : "/var/log/os_patching.20180528",
  "message" : "yum dry run shows no patching work to do",
  "return-code" : "success",
  "securityonly" : "false"
}
```

If patching was executed, the task will report:

```puppet
{
  "date": "Mon May 28 12:29:23 AEST 2018",
  "fqdn": "example.lab.vm",
  "reboot": "true",
  "logfile": "/var/log/os_patching.20180528",
  "message": "Patching complete",
  "return-code": "Success",
  "packagesupdated": [
    "kernel-tools-3.10.0-862.2.3.el7.x86_64",
    "kernel-tools-libs-3.10.0-862.2.3.el7.x86_64",
    "postfix-2:2.10.1-6.el7.x86_64",
    "procps-ng-3.3.10-17.el7.x86_64",
    "python-perf-3.10.0-862.2.3.el7.x86_64"
  ],
  "securityonly": "false"
}
```


## Limitations

This module is for PE2018+ with agents capable of running tasks.  It is currently limited to the Red Hat operating system.

## Development

Fork, develop, submit a pull request

## Contributors

- Tony Green <tgreen@albatrossflavour.com>