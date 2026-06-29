# Playbook
---
[Playbook guide](https://docs.ansible.com/ansible/latest/playbook_guide/index.html)  
[Official examples](https://github.com/ansible/ansible-examples)

[List of playbook keywords](https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html)

A `playbook` is the starting point similar a setup script that is executed.  
In a `playbook` you define a `group` from the inventory which will be targeted by its content, using the `hosts` variable.
```yaml
---
- hosts: linux_server
...

```
It is also possible to exclude one or multiple sub-groups:
```yaml
---
- hosts: linux_server !webservers
...

```
And wildcards:
```yaml
---
- hosts: linux_*
...

```

It can also contain connection settings to override those in the inventory.  
You can also define settings like `become:` (default: false) to define if everything should be run with privilege escalation (sudo) or
`gather_facts:` (default: true) to collect [facts](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html#id3),
which contain information about the target System like operating system,
IP and more, or `serial:`to define how many Systems should be configured at the same time and many more settings.  
Additionally you can define additional `vars` that may also contain or overwrite connection variables like `ansible_user`, `ansible_host` or `ansible_port`.  
It is also possible to define variables under `vars_prompt:` to ask for input for specific variables that you don't want to write down in any file.

Important to note that `become: true` is handled differently when used at the beginning of a `playbook` compared to when used in a specific `task` or `role`.  
When used at the beginning of a `playbook` Ansible will change the user on the target host to `root`, while when used in e.g. a `task` it is similar to using `sudo`.  
This is a majorly change that should be taken into account when e.g.: using `ansible_user_id`.  
Using `become: true` in the heading of a `playbook` should be avoided if possible as running everything as `root` also may create undesired security issues.


After the first 'configuration' blog it is possible to define one or all of the following sections:

- pre_tasks: list of tasks executed before roles
- roles: list of roles to be imported
- tasks: list of tasks executed after roles
- post_tasks: list of tasks executed after tasks section
- handlers: list of tasks, but ony run when a task contains the `notify` keyword and has the state changed.

Futhermore you can define multiple of these sets in one `playbook`:
```yaml
---
- name: Configure Linux server
  hosts: linux_server
  tasks: ...

- name: Configure Windows server
  hosts: windows_server
  tasks: ...
...

```

A `playbook` is run with the command:
```bash
ansible-playbook <playbook_name>.yml
```
With `ansible-playbook --help` you can also check out its multiple additional options.


```bash
ansinle-playbook example_playbook.yml
```
See how in this example `windows_docker` is in the list of targeted host because of the `docker_host` group.  
So again, be careful with how you name your groups.
