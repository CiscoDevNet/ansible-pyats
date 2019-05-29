ansible-pyats
=========

ansible-genie is a implementation of the [pyATS](https://developer.cisco.com/pyats/) network testing framework in an
Ansible role.  It contains tasks and filters to:
* Run a command and get structured output
* "snapshot" the output of a command and save it to a file
* Compare the current output of a command to a previous "snapshot"

In addition to tasks to accomplish these things, `ansible-pyats` contains to filters:
* `genie_parser`: provides structured data from unstructured command output
* `genie_diff`: provides the difference between two data structures

Requirements
------------

* pyats
* genie

Role Variables
--------------

* `command`: the command to run on the device
* `snapshot_file`: the name of the file to either store or retrieve the command "shapshot"

Dependencies
------------

None

Example Playbook
----------------

### Run a command and retreive the structured output
```yaml
- hosts: east-rtr1
  connection: network_cli
  gather_facts: no
  roles:
    - ansible-pyats
  tasks:
  - include_role:
      name: ansible-pyats
      tasks_from: parse_command
    vars:
      command: show ntp associations

  - debug:
      var: parsed_output
```

### Snapshot the output of a command to a file
```yaml
- hosts: east-rtr1
  connection: network_cli
  gather_facts: no
  roles:
    - ansible-pyats
  tasks:
  - include_role:
      name: ansible-pyats
      tasks_from: snapshot
    vars:
      command: show ntp associations
      snapshot_file: "{{ inventory_hostname }}_ntp.json"
```

### Compare the output of a command with a previous snapshot
```yaml
- hosts: router
  connection: network_cli
  gather_facts: no
  roles:
    - ansible-pyats
  tasks:
  - include_role:
      name: ansible-pyats
      tasks_from: compare
    vars:
      command: show ntp associations
      snapshot_file: "{{ inventory_hostname }}_ntp.json"
```

### Using the `genie_parser` filter directly
```yaml
- hosts: router
  connection: network_cli
  gather_facts: no
  roles:
    - ansible-pyats
  tasks:
    - name: Run command
      cli_command:
        command: show ntp associations
      register: cli_output
    
    - name: Parsing output
      set_fact:
        parsed_output: "{{ cli_output.stdout | genie_parser('show ntp associations', 'iosxe') }}"
```

### Using the `genie_diff` filter directly
```yaml
- name: Diff current and snapshot
  set_fact:
    diff_output: "{{ current_output | genie_diff(previous_output) }}"
```

License
-------

Cisco Sample License

