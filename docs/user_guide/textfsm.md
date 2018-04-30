# textfsm

The [textfsm](https://github.com/ansible-network/network-engine/blob/devel/library/textfsm.py)
module is based on [Google TextFSM](https://github.com/google/textfsm/wiki/TextFSM) definitions. 
This module iterates over matching rules defined in TextFSM format, extracts data from structured ASCII text based on those rules,
and returns Ansible facts in a JSON data structure that can be added to inventory host facts and/or consumed by Ansible tasks and templates.

## Playbook

```yaml

---

# The following task runs against network device

- hosts: ios

  tasks:
  - name: Collect interface information from device
    ios_command:
      commands: "show interfaces"
    register: ios_interface_output

  - name: Generate interface facts as JSON
    textfsm:
      file: "parsers/ios/show_interfaces"
      content: ios_interface_output['stdout'][0]
      name: interface_facts

```

## Parser

The `file` parameter for `textfsm` contains the standard textfsm rules.

The following describes how a parser file looks like:

`parsers/ios/show_interfaces`
```

Value Required name (\S+)
Value type ([\w ]+)
Value description (.*)
Value mtu (\d+)

Start
  ^${name} is up
  ^\s+Hardware is ${type} -> Continue
  ^\s+Description: ${description}
  ^\s+MTU ${mtu} bytes, -> Record

```

## Content

The `content` paramter for `textfsm` should have the ASCII text output of commands run on
network devices.
