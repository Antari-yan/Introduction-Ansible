# Templates
---
[Template testing with lookup](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_lookup.html)  
[Template testing with python script](https://github.com/Antari-yan/python_jinja2_renderer)

`Jinja2` is a template engine for Python used by Ansible nearly everywhere (e.g. variables, when...).  
It can also be used to define `templates` for any kind of file that should be placed on the target system.  
These templates can containing anything from variables, conditionals to loops and more.  
The `example_role` contains a short example in how it can be used.

Within a role:
```yml
- name: Generate file from Template
  ansible.builtin.template:
    src: example.j2
    dest: /etc/exports
```
