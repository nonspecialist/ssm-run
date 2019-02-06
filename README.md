# ssm-run

A simple (really, quite trivial) `bash` wrapper around the AWS CLI for `aws ssm send-command`.

Allows you to:

* execute one or more commands on one or more instances
* wait for and display the output from all of the commands

and that's all (so far)

## Prerequisites

1. `bash`
1. Make sure you have the SSM agent installed and running on your EC2 instances
   (see http://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent.html)
1. Make sure you have AWS credentials available for your command-line

## Installation

Copy `ssm-run` from this repo to somewhere in your `$PATH`. That's it.

## Usage

```shell
localhost$ ssm-run --help
ssm-run '<commands>' <instance-id> [<instance-id> ...]
ssm-run '<commands>' --tag 'foo=bar'
```

The command-ID is printed at the start of every run, which means you can use the regular
`aws ssm get-command-invocation` syntax to inspect particular outputs more closely

## Examples

### Getting the disk usage of the root filesystem of two EC2 instances

```shell
localhost$ ssm-run "df -h /" i-00c369e70f5e6dfbd i-4a684e9fdaec52001
Command launched with id 97aa89b4-5afc-4b16-bc74-48c6c47ee3da
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

### Get the date and Linux kernel details from two instances

```shell
localhost$ ssm-run "date; uname -a" i-00c369e70f5e6dfbd i-08806cf9f5a0a63d6
Command launched with id 46ce90e0-65e2-4d24-9464-547b5f9d0e41
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

### Execute a command 5 times, with a pause in between

```shell
localhost$ ssm-run 'for n in {1..5}; do df -k /; sleep 5; done' i-00c369e70f5e6dfbd
Command launched with id 0a7bdc66-4878-4bce-9284-7d62258eab59
i-00c369e70f5e6dfbd: inprogress
i-00c369e70f5e6dfbd: inprogress
i-00c369e70f5e6dfbd: inprogress
i-00c369e70f5e6dfbd: inprogress
i-00c369e70f5e6dfbd: inprogress
i-00c369e70f5e6dfbd: success
RESULTS FROM i-00c369e70f5e6dfbd (STATUS Success):
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/xvda1       8123812 1386512   6637052  18% /
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/xvda1       8123812 1386520   6637044  18% /
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/xvda1       8123812 1386520   6637044  18% /
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/xvda1       8123812 1386520   6637044  18% /
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/xvda1       8123812 1386520   6637044  18% /
------------------------------------
```

### Inspecting the output of a command later

```shell
localhost$ ssm-run date i-00c369e70f5e6dfbd
Command launched with id 2002a76f-e1a5-45ce-a64c-c7310856ed10
i-00c369e70f5e6dfbd: inprogress
i-00c369e70f5e6dfbd: success
RESULTS FROM i-00c369e70f5e6dfbd (STATUS Success):
Fri Dec 15 00:58:20 UTC 2017
------------------------------------

localhost$ aws ssm get-command-invocation \
    --command-id 2002a76f-e1a5-45ce-a64c-c7310856ed10 \
    --instance-id i-00c369e70f5e6dfbd
{
    "Comment": "", 
    "ExecutionElapsedTime": "PT0.005S", 
    "ExecutionEndDateTime": "2017-12-15T00:58:20.834Z", 
    "StandardErrorContent": "", 
    "InstanceId": "i-00c369e70f5e6dfbd", 
    "StandardErrorUrl": "", 
    "DocumentName": "AWS-RunShellScript", 
    "StandardOutputContent": "Fri Dec 15 00:58:20 UTC 2017\n", 
    "Status": "Success", 
    "StatusDetails": "Success", 
    "PluginName": "aws:runShellScript", 
    "ResponseCode": 0, 
    "ExecutionStartDateTime": "2017-12-15T00:58:20.834Z", 
    "CommandId": "2002a76f-e1a5-45ce-a64c-c7310856ed10", 
    "StandardOutputUrl": ""
}
```
