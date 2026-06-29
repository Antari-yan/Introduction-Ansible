# Ansible Configuration
---
[List of settings](https://docs.ansible.com/ansible/latest/reference_appendices/config.html)  
The `ansible.cfg` allows you to configure Ansible to your needs.
E.g.:

- add log directory
- change directories for playbooks, inventories, etc.
- set defaults
- ...

To generate a fully commented-out example `ansible.cfg` run:
```bash
ansible-config init --disabled > ansible.cfg
```

Or to also include existing plugins:
```bash
ansible-config init --disabled -t all > ansible.cfg
```

For this repository I have preset two settings:  
First the path for the directory containing the inventory files.  
This one is required, otherwise Ansible won't find the files, sadly this also means you have to be in the directory where all the Ansible files and
dirs are located ot it won't work (except if you change the config to the full path).  
Secondly the option to make the terminal output formatted in `YAML`, which is optional and can be removed if needed.
