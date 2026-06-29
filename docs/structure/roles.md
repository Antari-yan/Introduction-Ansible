# Roles
---
A [role](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html) can be viewed as a set of tasks and other things grouped together.  
It has a defined directory structure with eight main standard directories, but it is only required to include at least one of them.  
Directories not used can be omitted.
```bash
roles/
  example_role/           # this hierarchy represents a "role"
    defaults/       #
      main.yml      #  <-- default lower priority variables for this role
    tasks/          #
      main.yml      #  <-- tasks file can include smaller files if warranted
    handlers/       #
      main.yml      #  <-- handlers file
    files/          #
      bar.txt       #  <-- files for use with the copy resource
      foo.sh        #  <-- script files for use with the script resource
    templates/      #  <-- files for use with the template resource
      ntp.conf.j2   #  <------- templates end in .j2
    vars/           #
      main.yml      #  <-- variables associated with this role
    meta/           #
      main.yml      #  <-- role dependencies
    library/        # roles can also include custom modules
    module_utils/   # roles can also include custom module_utils
    lookup_plugins/ # or other types of plugins, like lookup in this case


  webtier/          # same kind of structure as "example_role" was above, done for the webtier role
  monitoring/       # ""
  fooapp/           # ""
```
Ansible looks for roles in the following locations:
- in collections (if you are using them)
- in a directory named `roles/`, relative to the playbook file
- in the configured `roles_path`. The default search path is `~/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles`
- in the directory where the playbook file is located

Alternatively the full path can be provided when calling a role:
```yaml
---
- hosts: webservers
  roles:
    - role: '/path/to/my/roles/common'
...

```

You can use roles in three ways:

- At the play level with the `roles` option: This is the classic way of using roles in a play.
- At the tasks level with `include_role`: You can reuse roles dynamically anywhere in the `tasks` section of a play using `include_role`.
- At the tasks level with `import_role`: You can reuse roles statically anywhere in the `tasks` section of a play using `import_role`.


By default a role is only run once no matter how often it is defined in a playbook.  
There are two ways to circumvent this:

- Passing different variables to the role
- Within `meta/main.yml` of a role set `allow_duplicates: true`

There is also the [ansible-galaxy](https://galaxy.ansible.com/ui/) which contains even more roles.

```bash
ansible-playbook example_role.yml
```
Uncommenting `allow_duplicates: true` in `roles/example_role/meta/main.yml` will show the difference when passing different variables.
