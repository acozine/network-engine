# text_parser

The [text_parser](https://github.com/ansible-network/network-engine/blob/devel/library/text_parser.py)
module is closely modeled after the Ansible playbook language.
This module iterates over matching rules defined in YAML format, extracts data from structured ASCII text based on those rules,
and returns Ansible facts in a JSON data structure that can be added to the inventory host facts and/or consumed by Ansible tasks and templates.

1. Define the data you want to extract
In this example we will copy a data definition from the Network Engine test suite.
`mkdir my-parsers`
`cp ~/.ansible/roles/ansible-network.network-engine/tests/text_parser/parsers/ios/show_version.yaml my-parsers/ios_show_version.yaml`
The `show_version.yaml` schema retrieves information from the `show version` command, such as uptime and free memory, on IOS, and records it in a variable called `system_facts`.
1. Create a playbook to extract the data you've defined
`mkdir ~/my-playbooks`
`cd ~/my-playbooks`
The example playbook below runs the `show versions` command, imports the Network Engine role, extracts the data you defined from the text output of the command, and views the results.
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
    text_parser:
      file: "parsers/ios/show_interfaces.yaml"
      content: ios_interface_output['stdout'][0]

```

## Parser

The `file` parameter for `text_parser` contains rules to parse text.
The rules in parser file uses directives written closely to Ansible language.

Directives documentation is available in [parser_directives.md](https://github.com/ansible-network/network-engine/blob/devel/docs/directives/parser_directives.md).

The following describes how a parser file looks like:

`parser/ios/show_interfaces.yaml`
```yaml

---
- name: parser meta data
  parser_metadata:
    version: 1.0
    command: show interface
    network_os: ios

- name: match sections
  pattern_match:
    regex: "^(\\S+) is up,"
    match_all: yes
    match_greedy: yes
  register: section

- name: match interface values
  pattern_group:
    - name: match name
      pattern_match:
        regex: "^(\\S+)"
        content: "{{ item }}"
      register: name

    - name: match hardware
      pattern_match:
        regex: "Hardware is (\\S+),"
        content: "{{ item }}"
      register: type

    - name: match mtu
      pattern_match:
        regex: "MTU (\\d+)"
        content: "{{ item }}"
      register: mtu

    - name: match description
      pattern_match:
        regex: "Description: (.*)"
        content: "{{ item }}"
      register: description
  loop: "{{ section }}"
  register: interfaces

- name: generate json data structure
  json_template:
    template:
      - key: "{{ item.name.matches.0 }}"
        object:
        - key: config
          object:
            - key: name
              value: "{{ item.name.matches.0 }}"
            - key: type
              value: "{{ item.type.matches.0 }}"
            - key: mtu
              value: "{{ item.mtu.matches.0 }}"
            - key: description
              value: "{{ item.description.matches.0 }}"
  loop: "{{ interfaces }}"
  export: yes
  register: interface_facts

```

`parser/ios/show_version.yaml`

```yaml

---
- name: parser meta data
  parser_metadata:
    version: 1.0
    command: show version
    network_os: ios

- name: match version
  pattern_match:
    regex: "Version (\\S+),"
  register: version

- name: match model
  pattern_match:
    regex: "^Cisco (.+) \\(revision"
  register: model

- name: match image
  pattern_match:
    regex: "^System image file is (\\S+)"
  register: image

- name: match uptime
  pattern_match:
    regex: "uptime is (.+)"
  register: uptime

- name: match total memory
  pattern_match:
    regex: "with (\\S+)/(\\w*) bytes of memory"
  register: total_mem

- name: match free memory
  pattern_match:
    regex: "with \\w*/(\\S+) bytes of memory"
  register: free_mem

- name: export system facts to playbook
  set_vars:
    model: "{{ model.matches.0 }}"
    image_file: "{{ image.matches.0 }}"
    uptime: "{{ uptime.matches.0 }}"
    version: "{{ version.matches.0 }}"
    memory:
      total: "{{ total_mem.matches.0 }}"
      free: "{{ free_mem.matches.0 }}"
  export: yes
  register: system_facts

```

## Content

The `content` paramter for `text_parser` should have the ASCII text output of commands run on
network devices.
