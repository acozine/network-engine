# textfsm_parser

The [textfsm_parser](https://github.com/ansible-network/network-engine/blob/devel/library/textfsm_parser.py)
module is based on [Google TextFSM](https://github.com/google/textfsm/wiki/TextFSM) definitions. 
This module iterates over matching rules defined in TextFSM format, extracts data from structured ASCII text based on those rules,
and returns Ansible facts in a JSON data structure that can be added to inventory host facts and/or consumed by Ansible tasks and templates.
The `textfsm_parser` module accepts three parameters: `content`, `file`, and `src`. The parameters `file` and `src` are 
mutually exclusive - use one or the other, but not both.

## Content

The `content` parameter for `textfsm_parser` should point to the ASCII text output of commands run on network devices. The text output can be in a variable or in a file.

## File

The `file` parameter for `textfsm_parser` must point to a parser template that contains a TextFSM rule for each data field you want to extract from your network devices. 

Parser templates for the `textfsm_parser` module in the Network Engine role use TextFSM notation.

### src

The `src` parameter for `textfsm_parser` loads your parser template from an external source, usually a URL.

## Sample Parser Templates

Here is a sample TextFSM parser template:

`parser_templates/ios/show_interfaces`
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
    textfsm_parser:
      file: "parser_templates/ios/show_interfaces"
      content: ios_interface_output['stdout'][0]
      name: interface_facts

```
