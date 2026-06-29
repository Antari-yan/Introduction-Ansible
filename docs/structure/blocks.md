# Blocks & Error Handling
---
[Block guide](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_blocks.html)

Through `blocks` it is possible to group multiple `tasks` together and is the only way to create proper exception handling.  
They are defined like `tasks` with all tasks being grouped below it through indent and
directives like `become` defined on the `block` definition are inherent by all tasks grouped within the `block`.  
Should a `task` within a `block` fail sections like `rescue` and `always` can be defined.  
`rescue` will run when a task within `block` fails and `always` will run no matter what happened in `block` or `rescue`.

```yaml
- name: Sample block
  become: true
  block:
    - name: Some task
      ansible.builtin.debug:
        msg: "some task"
  rescue:
    - name: Some task when block fails
      ansible.builtin.debug:
        msg: "some task"
  always:
    - name: Some task always run
      ansible.builtin.debug:
        msg: "some task"
```

The `example_block.yml` file contains basic examples for block, rescue, always.
```bash
ansible-playbook example_block.yml
```
