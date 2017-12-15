# ssm-run

A simple (really, quite trivial) `bash` wrapper around the AWS CLI for `aws ssm send-command`.

Allows you to:

* execute one or more commands on one or more instances
* wait for and display the output from all of the commands

and that's all (so far)

## Setup

1. Make sure you have the SSM agent installed and running on your EC2 instances
   (see http://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent.html)
1. Make sure you have AWS credentials available for your command-line

## Usage

```shell
localhost$ ssm-run "<commands>" <instance-id> [<instance-id> ...]
```

## Examples

### Getting the disk usage of the root filesystem of two EC2 instances

```shell
localhost$ ssm-run "df -h /" i-00c369e70f5e6dfbd i-4a684e9fdaec52001
i-00c369e70f5e6dfbd: inprogress
i-4a684e9fdaec52001: inprogress
i-00c369e70f5e6dfbd: success
i-4a684e9fdaec52001: success
RESULTS FROM i-00c369e70f5e6dfbd (STATUS Success):
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1      7.8G  760M  7.0G  10% /
------------------------------------
RESULTS FROM i-4a684e9fdaec52001 (STATUS Success):
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda1      7.8G  1.3G  6.5G  16% /
------------------------------------
```

### Get the date and Linux kernel details from two instances:

```shell
localhost$ ssm-run "date; uname -a" i-00c369e70f5e6dfbd i-08806cf9f5a0a63d6
i-00c369e70f5e6dfbd: inprogress
i-08806cf9f5a0a63d6: success
i-00c369e70f5e6dfbd: success
i-08806cf9f5a0a63d6: success
RESULTS FROM i-00c369e70f5e6dfbd (STATUS Success):
Thu Dec 14 23:58:39 UTC 2017
Linux ip-10-13-48-24 4.9.58-18.51.amzn1.x86_64 #1 SMP Tue Oct 24 22:44:07 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
------------------------------------
RESULTS FROM i-08806cf9f5a0a63d6 (STATUS Success):
Thu Dec 14 23:58:39 UTC 2017
Linux ip-10-13-48-56 4.9.58-18.51.amzn1.x86_64 #1 SMP Tue Oct 24 22:44:07 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
------------------------------------
```
