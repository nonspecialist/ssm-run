# ssm-run

A simple (really, quite trivial) `bash` wrapper around the AWS CLI for `aws ssm send-command`.

Allows you to:

* execute one or more commands on one or more instances
* wait for and display the output from all of the commands

## Usage

```shell
localhost$ ssm-run "command1; command2" <instance-id> [<instance-id> ...]
```
