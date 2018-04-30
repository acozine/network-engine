How do I use the Network Engine role? 
----------------------------------

The Network Engine Role can be used directly or as a dependency of other Roles and/or Playbooks. The Network Engine Role extracts elements of network device information (state, configuration, etc.) and returns them as Ansible facts in a JSON data structure, ready to be added to your inventory host facts and/or consumed by Ansible tasks and templates. With the Network Engine role, you can normalize your Ansible facts across network platforms. 

The initial release of the Network Engine role includes modules for defining and using the following parsing engines:
* [text_parser](https://github.com/ansible-network/network-engine/blob/devel/docs/user_guide/text_parser.md) accepts YAML-formatted input, uses an internally maintained, loosely defined parsing language based on Ansible playbook directives
* [textfsm](https://github.com/ansible-network/network-engine/blob/devel/docs/user_guide/textfsm.md) accepts Google TextFSM-formatted input, uses Google TextFSM parsing language

Both modules iterate over the rules you define, parse the output of structured ASCII text, and then convert it into Ansible facts in a JSON data structure.

In the following example text_parser is used to read the text provided via content and run it through the parser specified via the file module argument. The parser itself (show_interfaces.yaml) contains a set of rules that defines how to extract text and what variables they will be made accessible as facts. Different parser files will be generally be needed for different network platforms to deal with the differences in how data is returned. 

- name: "Collect information show_interface"
  text_parser:
    file: "parsers/show_interfaces.yaml"
    content: "{{ lookup('file', '{{ output_path }}/show_interfaces.txt') }}"
  
- name: Display some details of an interface
  debug:
    msg: "Description of GE0/0 {{ ansible_facts.interface_facts[0]['GigabitEthernet0/0']['config']['description'] }}"

To use the Network Engine Role directly:
Create a file for the show command you want to run, with definitions for the elements you want to retrieve. This file is your parser input. For example, you might want to retrieve information from the show version command, such as uptime and free memory, to automate load management. For an example of YAML-formatted parser input for show version on IOS, see  https://github.com/ansible-network/network-engine/blob/devel/tests/text_parser/parsers/ios/show_version.yaml. The regex in the pattern_match may be different on each platform, but by defining the same variable names for the output (register) on all platforms, you can normalize similar data across platforms.
Once your parser input is ready, create a task to run the parser on your device(s) and store the information about your device(s) in a variable. For an example task using the show version definitions shown above and creating a variable result to contain the JSON data retrieved, see https://github.com/ansible-network/network-engine/blob/devel/tests/text_parser/tasks/ios.yaml#L19-L25 
Consume the JSON facts about your device(s) in templates and tasks.

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