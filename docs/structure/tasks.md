# Tasks
---
A task is a singular action that should be run.  
It can be anything from a simple console command, to disk formatting, network configuration and more.  
Every possible action is grouped in `namespace` and a [collection](https://docs.ansible.com/ansible/latest/collections/index.html) reaching from
[builtin](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/index.html#plugins-in-ansible-builtin) to [community managed](https://docs.ansible.com/ansible/latest/collections/community/index.html).  
There is also the [ansible-galaxy](https://galaxy.ansible.com/ui/) which contains even more collections outside of the official documentation.

| Namespace | Collection | Module |
| --------- | ---------- | ------ |
| ansible   | builtin    | debug  |
| community | general    | sudoers|
| kubernetes| core       | helm   |

The `example_tasks.yml` file contains basic examples in how tasks are structured.
```bash
ansible-playbook example_tasks.yml
```
A task can additional to it's action contain where, when and with what privileges it should run and more.

Each task is at the end like a `Python` module defined in a `collection` within a `namespace`.  
This is also known as [Fully Qualified Collection Name (FQCN)](https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html#term-Fully-Qualified-Collection-Name-FQCN).  
While it is possible to only use the `module` name (like `debug`) it is highly recommended to use the `FQCN`
to ensure you don't accidentally run a `module` from a different `collection` or `namespace` that has the same name.
