# Conditionals & Loops
---
Ansible allows to check for specific conditions before running a `task`, `role`, `block`, etc. using the `when` keyword.  
It can be used to check if some variable contains desired information, like if something is set to `true`, is a specific package installed, etc..  
`when` can either be a single line string or a multiline list.  
When stated with a list each part in that list is connected as if `and` is used.  
Conditions can be linked together with `and` and `or` keywords.
```bash
ansible-playbook  example_conditionals.yml
```


[Loops](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_loops.html) allows repetitive execution of a single task.  
This can be useful to e.g. install multiple packages, create multiple user, etc..
Most commonly loops are used to iterate over a list where each singular part is available with the `"{{ item }}` keyword.

```bash
ansible-playbook  example_loops.yml
```

Loops can be managed with the [loop_control](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_loops.html#adding-controls-to-loops) keyword.  
It allows to set a `label`which can be used to define the console output during the task run. This can be used  e.g. when iterating over list of dicts `"{{ item.name }}"`.  
Additionally it is possible to [extend](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_loops.html#extended-loop-variables) the loop information by
setting `extended: true` in the `loop_control`, which adds the following information:

| Variable                 | Description                                                                             |
| ------------------------ | --------------------------------------------------------------------------------------- |
| `ansible_loop.allitems`  | The list of all items in the loop                                                       |
| `ansible_loop.index`     | The current iteration of the loop. (1 indexed)                                          |
| `ansible_loop.index0`    | The current iteration of the loop. (0 indexed)                                          |
| `ansible_loop.revindex`  | The number of iterations from the end of the loop (1 indexed)                           |
| `ansible_loop.revindex0` | The number of iterations from the end of the loop (0 indexed)                           |
| `ansible_loop.first`     | True if first iteration                                                                 |
| `ansible_loop.last`      | True if last iteration                                                                  |
| `ansible_loop.length`    | The number of items in the loop                                                         |
| `ansible_loop.previtem`  | The item from the previous iteration of the loop. Undefined during the first iteration. |
| `ansible_loop.nextitem`  | The item from the following iteration of the loop. Undefined during the last iteration. |


Sometimes it can also be useful to loop over something but only do something when an `item` meets a specific condition.
```bash
ansible-playbook  example_loop_conditionals.yml
```
