# Variables
---
[Using Variables](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html)
[Filter plugins](https://docs.ansible.com/ansible/latest/plugins/filter.html)

Variables can be bool, integer, string, list, dictionary and any combination of lists and dictionaries
```yaml
vars:
  int_var: 1
  string_var: fuu
  list_var_1: [ fuu, bar, 1 ]
  list_var_2:
    - fuu
    - bar
    - 1
  dict_var_1: { name: fuu, number: 2 }
  dict_var_2:
    name: fuu
    number: 1
  list_dict:
    - name: fuu
      number: 3
    - name: bar
      number: 4
  dict_list:
    names:
      - fuu
      - bar
    number: 5
```

There are also [Special Variables](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html) and
[Facts](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html) that can be used as variables.  
These contain variables containing connection information, paths to ansible files/dirs, facts about a host like OS, IP Address und much more.

Variables can be defined and overwritten in multiple places,
which results in a list of [Variable Precedence](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#understanding-variable-precedence)
from least to greatest (the last listed variables override all other variables):

1. role defaults (defined in role/defaults/main.yml) 1
2. inventory file or script group vars 2
3. inventory group_vars/all 3
4. playbook group_vars/all 3
5. inventory group_vars/* 3
6. playbook group_vars/* 3
7. inventory file or script host vars 2
8. inventory host_vars/* 3
9. playbook host_vars/* 3
10. host facts / cached set_facts 4
11. play vars
12. play vars_prompt
13. play vars_files
14. role vars (defined in role/vars/main.yml)
15. block vars (only for tasks in block)
16. task vars (only for the task)
17. include_vars
18. set_facts / registered vars
19. role (and include_role) params
20. include params
21. extra vars (for example, -e "user=my_user")(always win precedence)


To use variable, they can be referenced via double curly braces (Jinja2 syntax) `"{{ int_var }}"`.  
To create valid YAML syntax, usually the whole expression containing a variable needs to be quoted.  
This is usually not needed when used in a `when:` statement.


It is also possible to define [Custom Facts](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html#facts-d-or-local-facts).  
They will be added automatically when running `gather_facts`, but they can also be loaded manually with optionally adding path using:
```yaml
tasks:
    - name: Filter and return only selected facts
      ansible.builtin.setup:
        filter:
          - 'ansible_local'
        fact_path: "{{ playbook_dir }}/facts.d"
      delegate_to: localhost
```
The default path Ansible is looking for is `/etc/ansible/facts.d` but that can be changed in `ansible.cfg` with `fact_path=`.  
Custom facts need to be either in `JSON` or `INI` format.  
The content of the files under `facts.d` can be anything, just the return value needs to be correctly formatted.  
This allows to also run script files (`bash`, `python`, ...) that return e.g. `JSON` formatted `key/value` pairs.  
For that the scripts have to be executable with shebang (e.g.: `#!/usr/bin/env bash`) and proper permissions (e.g.: `chmod +x facts.d/date_time.fact`).

```bash
ansible-playbook  example_vars.yml
ansible-playbook  example_facts.yml
ansible-playbook  example_custom_facts.yml
ansible-playbook  example_query.yml
ansible-playbook  example_lookup.yml
ansible-playbook  example_vars_nested.yml
```
