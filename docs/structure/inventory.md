# Inventory
---
[Inventory building](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html)

The inventory which contains a structure of your host, groups and if desired variables and connection settings can be built in multiple ways.  
The most common ones are either as `YAML` or `ini`.  
As there is not much of a difference, except for how the files are structured, I will only go over the `YAML` structure for now.

```YAML
---
all:
  children:
    internal:
      children:
        ansible_vm:
          hosts:
            local_system:
              ansible_connection: local
              host_test_var: test
          vars:
            group_test_var: test
hosts:
  ungrouped_host:
...
```

- The `---` and `...` mark the beginning and end of a `YAML` file and are used in all of them. The `...` is optional but I add anyway.
- Under `all` you place all your hosts and groups, if needed it is also possible to define hosts separately.
- With `childern:`you can define sub-groups to separate `hosts` by type, task, service, cluster or however you want
- `hosts` contains all Systems for a group or sub-group.
- You can define any variable or [connection](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#connecting-to-hosts-behavioral-inventory-parameters) option either directly below a specific System or for a whole group or sub-group.

When running a `playbook` you can specify with `-i <path>` which inventory should be used.  
With my configuration any `YAML` or `ini` will be automatically loaded from the `inventory/` directory.

When building your inventory be careful with naming a group, because groups with the same name will be grouped together when called directly (also known as `metagroup`).  
This can be seen when running the following example:
```bash
ansible-playbook example_host_grouping.yml
```
In the recap you will see two host because both are in the sub-group `docker_host` even though they are in separate different sub-groups.  
Because of this I recommend using unique names for a group like adding a suffix or prefix to the name of the parent group or something like `prod_` and `staging_`.

But you can also use this function to build your inventory like this:
```yaml
leafs:
  hosts:
    leaf01:
      ansible_host: 192.0.2.100
    leaf02:
      ansible_host: 192.0.2.110

spines:
  hosts:
    spine01:
      ansible_host: 192.0.2.120
    spine02:
      ansible_host: 192.0.2.130

network:
  children:
    leafs:
    spines:

webservers:
  hosts:
    webserver01:
      ansible_host: 192.0.2.140
    webserver02:
      ansible_host: 192.0.2.150

datacenter:
  children:
    network:
    webservers:
```
This example contains a metagroup `network` that includes all network devices and a metagroup `datacenter` that includes the `network` and `webservers` groups.

```bash
ansible-playbook example_inventory.yml
```
