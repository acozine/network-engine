Using the Network Engine Role 
----------------------------------

The Network Engine Role can be used directly or as a dependency of other Roles. The Network Engine Role extracts data about your network devices as Ansible facts in a JSON data structure, ready to be added to your inventory host facts and/or consumed by Ansible tasks and templates. You define the data elements you want to extract from each network OS command as input for the role. You can use either YAML or Google TextFSM to structure this input. The regex in the `pattern_match` may be different on each platform, but by defining the same variable names for the output (register) on all platforms, you can normalize similar data across platforms. That's how the Network Engine Role supports truly platform-agnostic network automation. 

The initial release of the Network Engine role includes modules for defining and using the following parsing engines:
* [text_parser](https://github.com/ansible-network/network-engine/blob/devel/docs/user_guide/text_parser.md) accepts YAML-formatted input, uses an internally maintained, loosely defined parsing language based on Ansible playbook directives
* [textfsm](https://github.com/ansible-network/network-engine/blob/devel/docs/user_guide/textfsm.md) accepts Google TextFSM-formatted input, uses Google TextFSM parsing language

Both modules iterate over the data model you define in your input, parse the output of structured ASCII text, and then convert it into Ansible facts in a JSON data structure.

To use the Network Engine Role:
----------------------------------------
1. Install the role
`ansible-galaxy install ansible-network.network-engine` will copy the Network Engine role to `~/.ansible/roles/`.
1. Select the parser you prefer
For YAML formatting, use `text_parser`; for TextFSM formatting, use `textfsm`. The parser docs include
examples of how to define your data and create your tasks.


Additional Resources
-------------------------------------
https://galaxy.ansible.com/ansible-network/network-engine/#readme
https://github.com/ansible-network/network-engine/tree/devel/tests/textfsm
https://github.com/ansible-network/network-engine/tree/devel/tests/text_parser

Full changelog diff:
https://github.com/ansible-network/network-engine/blob/devel/CHANGELOG.rst

Contributing and Reporting Feedback
-------------------------------------
https://github.com/ansible-network/network-engine/issues