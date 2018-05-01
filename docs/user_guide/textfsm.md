# textfsm

The [textfsm](https://github.com/ansible-network/network-engine/blob/devel/library/textfsm.py)
module is based on [Google TextFSM](https://github.com/google/textfsm/wiki/TextFSM) definitions. 
This module iterates over matching rules defined in TextFSM format, extracts data from structured ASCII text based on those rules,
and returns Ansible facts in a JSON data structure that can be added to inventory host facts and/or consumed by Ansible tasks and templates.
The `textfsm` module accepts two parameters: `content` and `file`.

## Content

The `content` parameter for `textfsm` should point to the ASCII text output of commands run on network devices. The text output can be in a variable or in a file.

## File

The `file` parameter for `textfsm` must point to a data definition file that contains a textfsm rule for each data field you want to extract from your network devices. 

Data definition files for the `textfsm` module in the Network Engine role use TextFSM notation.

Here is a sample data definition file:

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

## Sample Playbooks

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

