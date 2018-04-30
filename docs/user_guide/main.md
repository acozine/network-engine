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
`ansible-galaxy install ansible-network.network-engine` will copy the role to `~/.ansible/roles/`.
1. Define the data elements you want to extract.
For this example we will copy a file from the test suite for the role:
`mkdir ~/my-playbooks`
`cd ~/my-playbooks`
`mkdir my-parsers`
`cp ~/.ansible/roles/ansible-network.network-engine/tests/text_parser/parsers/ios/show_version.yaml my-parsers/ios_show_version.yaml`
This schema retrieves information from the `show version` command, such as uptime and free memory, on IOS, and records it in a variable called `system_facts`.
1. Create a playbook that runs the command, imports the Network Engine role, extracts the data you defined from the text output of the command, and views the results.
(The last step is for demonstration purposes only.) Make sure the `hosts` definition in the playbook matches a host group in your inventory - in this example, the playbook expects a group called `ios`.
```yaml

---

# ~/my-playbooks/gather-version-info.yml

- hosts: ios
  connection: network_cli
  gather_facts: no

  tasks:
  - name: Collect interface information from device
    ios_command:
      commands: "show versions"
    register: ios_versions_output

  - name: import the network-engine role
    import_role:
      name: ansible-network.network-engine

  - name: Generate interface facts as JSON
    text_parser:
      file: "my-parsers/ios_show_versions.yaml"
      content: ios_versions_output['stdout'][0]

  - name: Display version facts in JSON
    debug:
      var: system_facts 
```

1. Run the playbook with `ansible-playbook -i /path/to/your/inventory -u my_user -k my-plabyooks/gather-version-info.yml
1. Consume the JSON facts about your device(s) in templates and tasks.


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