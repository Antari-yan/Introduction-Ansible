# Handlers
---
[Handlers guide](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_handlers.html)

Handlers are singular tasks that only run when `notified` by a `task` and only when that `task` has the status `changed`.  
This can be useful for example when a change in configuration request the restart of a service but the service shouldn't be restarted when it is not needed.

```yaml
- name: Update package cache
  ansible.builtin.package:
    update_cache: true
      notify: handler1

- name: Update package cache
  ansible.builtin.package:
    update_cache: true
      notify:
        - handler1
        - handler2
```

Handlers are run in the order they are defined in the `handlers` section and not in the order of the `notify` section.  
handlers can have utilize the keyword `listen` so that multiple handlers can be grouped together.
```yaml
tasks:
  - name: Restart everything
    command: echo "this task will restart the web services"
    notify: "restart web services"

handlers:
  - name: Restart memcached
    service:
      name: memcached
      state: restarted
    listen: "restart web services"

  - name: Restart apache
    service:
      name: apache
      state: restarted
    listen: "restart web services"
```

By default, handlers run after all the tasks in a particular play have been completed.  
Notified handlers are executed automatically after each of the following sections, in the following order: pre_tasks, roles/tasks and post_tasks.  
This approach is efficient, because the handler only runs once, regardless of how many tasks notify it.  
For example, if multiple tasks update a configuration file and notify a handler to restart Apache, Ansible only bounces Apache once to avoid unnecessary restarts.  
If you need handlers to run before the end of the play, add a task to flush them using the meta module, which executes Ansible actions:
```yaml
tasks:
  - name: Some tasks go here
    ansible.builtin.shell: ...

  - name: Flush handlers
    ansible.builtin.meta: flush_handlers

  - name: Some other tasks
    ansible.builtin.shell: ...
```
The `meta: flush_handlers` task triggers any handlers that have been notified at that point in the play.

Once handlers are executed, either automatically after each mentioned section or manually by the flush_handlers meta task, they can be notified and run again in later sections of the play.

Handlers from roles are not just contained in their roles but rather inserted into the global scope with all other handlers from a play.  
As such they can be used outside of the role they are defined in. It also means that their name can conflict with handlers from outside the role.  
To ensure that a handler from a role is notified as opposed to one from outside the role with the same name, notify the handler by using its name in the following form: `role_name : handler_name`.

Handlers notified within the roles section are automatically flushed at the end of the tasks section but before any tasks handlers.

!!! note
    Handlers can run `include_tasks`, `import_task` and meta modules.  
    Handlers can't run `include_role`, `import_role`, and meta module `flush_handlers`.


The `example_handlers.yml` file contains basic examples for handlers.
```bash
ansible-playbook example_handlers.yml
```
