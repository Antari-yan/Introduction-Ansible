# Directory Structure
---
There are many ways in how to build the directory structure for Ansible.  
You can find additional examples [here](https://docs.ansible.com/ansible/latest/tips_tricks/sample_setup.html).  
The structure within this repository is just what worked best for me.

- In the base directory is the Ansible Configuration file `ansible.cfg` and the so-called playbooks `*.yml`/`*.yaml` (more later).
- The `inventory` directory contains one or multiple inventory files (all files are read automatically) that contain hosts that can be combined by groups.
- `host_vars` and `group_vars` can contain files named after hots or groups as they are defined in the inventories, they are matched automatically.
- In the `roles` directory are multiple roles that each contain individual tasks and generic files, variables etc. to fulfill those tasks.
- The `files` directory is not Ansible related, I just found it useful to have a separate directory for files that I want to copy to multiple target Systems (e.g. public SSH-Keys).

It is possible to have an own directory for the `playbooks` but you would have to define the path for the roles in the `ansible.cfg` using `roles_path`,
because Ansible assumes that `roles` are stored in a sub-directory where the playbooks are.
