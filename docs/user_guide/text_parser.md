# text_parser

The [text_parser](https://github.com/ansible-network/network-engine/blob/devel/library/text_parser.py)
module is closely modeled after the Ansible playbook language.
This module iterates over matching rules defined in YAML format, extracts data from structured ASCII text based on those rules,
and returns Ansible facts in a JSON data structure that can be added to the inventory host facts and/or consumed by Ansible tasks and templates.
The `text_parser` module accepts two parameters: `content` and `file`.

## Content

The `content` parameter for `text_parser` should point to the ASCII text output of commands run on network devices. The text output can be in a variable or in a file.


## File

The `file` parameter for `text_parser` must point to a data definition file that contains a regular expression rule for each data field you want to extract from your network devices. 

Data definition files for the `text_parser` module in the Network Engine role use YAML notation and a syntax similar to Ansible playbooks.

Directives documentation is available in [parser_directives.md](https://github.com/ansible-network/network-engine/blob/devel/docs/directives/parser_directives.md).

Here are two sample data definition files:

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

## Playbook

To extract the data defined in your file, create a playbook that includes the Network Engine role and references the `content` and `file` parameters of the `text_parser` module. 

The example playbook below runs the `show versions` command, imports the Network Engine role, extracts data defined from the text output of the command by matching it against the rules defined
in your data definition file, and views the results. (The last step is for demonstration purposes only.) 

Make sure the `hosts` definition in the playbook matches a host group in your inventory - in these examples, the playbook expects a group called `ios`.

The first example parses the output of the `show interfaces` command on IOS and creates facts from that output:

```yaml

---

# ~/my-playbooks/gather-interface-info.yml

- hosts: ios
  connection: network_cli
  gather_facts: no

  tasks:
  - name: Collect interface information from device
    ios_command:
      commands: "show interfaces"
    register: ios_interface_output

  - name: import the network-engine role
    import_role:
      name: ansible-network.network-engine

  - name: Generate interface facts as JSON
    text_parser:
      file: "parsers/ios/show_interfaces.yaml"
      content: ios_interface_output['stdout'][0]

  - name: Display interface facts in JSON
    debug:
      var: interface_facts
```

The second example parses the output of the `show interfaces` command on IOS and creates facts from that output:

```yaml

---

# ~/my-playbooks/gather-version-info.yml

- hosts: ios
  connection: network_cli
  gather_facts: no

  tasks:
  - name: Collect version information from device
    ios_command:
      commands: "show version"
    register: ios_version_output

  - name: import the network-engine role
    import_role:
      name: ansible-network.network-engine

  - name: Generate version facts as JSON
    text_parser:
      file: "parser/ios/show_version.yaml"
      content: ios_version_output['stdout'][0]

  - name: Display version facts in JSON
    debug:
      var: system_facts
```

