# Introduction to Ansible
:warning: WiP
:-


Intention for this repository is to provide a basic introduction to Ansible while also providing a usable base structure with examples.  
The goal is not to cover anything and everything Ansible is capable of but enough to make practical use of it

:warning: WARNING
:-
The examples are tested on a `Debian` based System.  
Be aware of possible differences on other Systems.


## Table of Content
- [What is Ansible?](#what-is-ansible)
- [Installation](#installation)
  - [Directory structure](#directory-structure)
  - [SSH Configuration](#ssh-config)
    - [SSH-Agent](#ssh-agent)
    - [TPM 2.0](#tpm-20)
  - [Prepare target Systems](#prepare-target-systems)
  - [Ansible Configuration Settings](#ansible-configuration-settings)
- [Structure](#structure)
  - [Inventory](#inventory)
  - [Playbook](#playbook)
  - [Tasks](#tasks)
  - [Handlers](#handlers)
  - [Roles](#roles)
  - [Vars](#vars)
  - [Conditions and Loops](#conditions-and-loops)
  - [Grouping and Error handling](#grouping-and-error-handling)
  - [Jinja2 Templates](#jinja2-templates)
  - [Vault](#vault)
- [Extras](#extras)
  - [Pull Mode](#pull-mode)
  - [Execution Environment](#execution-environment)


## What is Ansible?
Ansible is an IT automation tool. It can be used to Configure and Deploy Systems and Software.
It can be viewed, in a classical way, as a collection of installer script to ensure a System reaches a desired state of operation, but with the intend of making these scripts reusable on multiple Systems while making it possible to re-run the same scripts on a System multiple times without making chages if the desired state has already been reached ([idempotent](https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html#term-Idempotency)).   
Through `SSH` it connects to Systems and Devices and manages them based on `YAML` and `Python` Code.  
Because of this Ansible is essentially agent-less. The target Systems only need to be able to accept SSH connections (e.g. openssh-server).  
It also has decent [Documentation](https://docs.ansible.com/ansible/latest/).  
Key term definitions can be found [here](https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html).


## Installation
Install Ansible on your System:
```bash
sudo apt update
sudo apt install -y openssh-client python3 python3-pip ansible
pip install --user --upgrade pip
pip install --user --upgrade ansible
```
I always recommend to update `ansible` via `pip` to the latest version, this way you will be warned in advance about `deprecations` when using it.  
Also, **new features**.


### Directory structure
---
There are many ways in how to build the directory structure for Ansible.  
You can find additional examples [here](https://docs.ansible.com/ansible/latest/tips_tricks/sample_setup.html).  
The structure within this repository is just what worked best for me.

- In the base directory is the Ansible Configuration file `ansible.cfg` and the so-called playbooks `*.yml`/`*.yaml` (more later).
- The `inventory` directory contains one or multiple inventory files (all files are read automatically) that contain hosts that can be combined by groups.
- `host_vars` and `group_vars` can contain files named after hots or groups as they are defined in the inventories, they are matched automatically.
- In the `roles` directory are multiple roles that each contain individual tasks and generic files, variables etc. to fulfill those tasks.
- The `files` directory is not Ansible related, I just found it useful to have a separate directory for files that I want to copy to multiple target Systems (e.g. public SSH-Keys).

It is possible to have an own directory for the `playbooks` but you would have to define the path for the roles in the `ansible.cfg` using `roles_path`, because Ansible assumes that `roles` are stored in a sub-directory where the playbooks are.


### SSH Configuration
---
Ansible is capable of reading the `~/.ssh/config`.  
Through this you con't have to manage any `hosts` file.  
Just add the Systems that you would like to mange with Ansible into the configuration like:
```bash
Host sample1
  HostName target-server.com
  User user1
  PreferredAuthentication publickey
  IdentityFile ~/.ssh/key_file_name

Host sample2
  HostName 172.1.0.5
  User user2
  PreferredAuthentication publickey
  IdentityFile ~/.ssh/key_file_name

Host *
  IdentityFile ~/.ssh/default_key
  IdentitiesOnly yes	# Only send configured Identity and not any other
  UseRoaming no
  SendEnv LANG LC_*
  Compression yes
```
It is recommended to avoid always reusing the same SSH-Key and also having one without passphrase.  
If having individual SSH-Keys and passphrases for each System is too tedious consider having one SSH-Key for a group of Systems or one for each customer or whatever fits your infrastructure and requirements.

Here are some great talks about SSH and how it can be used in a good way:  
[Talk about SSH by Leyrer Part1 (german)](https://media.ccc.de/v/gpn20-8-besser-leben-mit-ssh)  
[Talk about SSH by Leyrer Part2 (german)](https://media.ccc.de/v/gpn21-28-noch-besser-leben-mit-ssh#t=829)

[Talk about SSH by Leyrer (english)](https://media.ccc.de/v/mch2022-170-ssh-configuration-intermediate-level)


Some are saying that to manage Windows Systems `winrm` can/should be used.  
I highly discourage you to do that, as you can simply use SSH.  
Also the example scripts that you may find online or by Microsoft to enable `winrm` usually aren't meant for production and disable security features.


#### SSH-Agent
To remove the need to always insert the passphrase for the SSH-Keys, the `ssh-agent` command can be used.  
For some reason the SSH-Agent requires to be started for each new terminal session.  
This can be circumvented by auto-loading the SSH-Agent env vars on each terminal session.  
I am adding it to `/etc/bash_completion.d` because avoiding adding anything and everything to `.bashrc` should be preferred, also having individual files makes cleanup easier.
```bash
echo 'eval "$(ssh-agent -s)" > /dev/null' | sudo tee /etc/bash_completion.d/ssh-agent.bash_completion > /dev/null
```
Alternatively creating a `.bashrc.d` directory in your home is also recommended:
```bash
mkdir ~/.bashrc.d
chmod 700 ~/.bashrc.d

sudo tee -a ~/.bashrc > /dev/null <<EOT
for file in ~/.bashrc.d/*.bashrc;
do
source “\$file”
done
EOT

echo 'eval "$(ssh-agent -s)" > /dev/null' | sudo tee ~/.bashrc.d/ssh-agent.bashrc > /dev/null
chmod +x ~/.bashrc.d/*.bashrc
```
Now you can place files inside of that directory as `<filename>.bashrc`, but you may have to add execution permission to each file.
```bash
chmod +x ~/.bashrc.d/*.bashrc
```

Now you can load the public key into your SSH-Agent which requires you to only enter the passphrase ones for as long as the terminal session is open.
```bash
ssh-add ~/.ssh/key_file_name
```

#### TPM 2.0
It is possible to have SSH-keys in the TPM 2.0 Chip, but I haven't used that yet.  
It is well explained here:
[Talk about SSH by Leyrer Part2 (german)](https://media.ccc.de/v/gpn21-28-noch-besser-leben-mit-ssh#t=829)


### Prepare target Systems
---
Linux (Debian):
```bash
sudo apt update
sudo apt install -y openssh-server python3
```

Windows Powershell:
```powershell
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*' -OutVariable cmdOutput
# should return two entries
# OpenSSH.Client~~~~0.0.1.0
# OpenSSH.Server~~~~0.0.1.0

Add-WindowsCapability -Online -Name $cmdOutput[0].name
Add-WindowsCapability -Online -Name $cmdOutput[1].name

Start-Service sshd

# Add SSH to autostart
Set-Service -Name sshd -StartupType 'Automatic'
```

After the OpenSSH Server is installed add the SSH-Keys from the System where Ansible will be run from to the target System.
```bash
# From Ansible System
ssh-copy-id -i ~/.ssh/key_file_name <target_system>
```
Also consider disabling the option to allow login via SSH using Password (after you made sure login via SSH-Key works):
```bash
#/etc/ssh/sshd_config
PasswordAuthentication no
```
```bash
systemctl restart sshd
```


### Ansible Configuration Settings
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
First the path for the directory containing the inventory files. This one is required, otherwise Ansible won't find the files, sadly this also means you have to be in the directory where all the Ansible files and dirs are located ot it won't work (except if you change the config to the full path).  
Secondly the option to make the terminal output formatted in `YAML`, which is optional and can be removed if needed.


### Test
---
Download this repository and switch into its directory.  
Run the first example (Connections to localhost don't require SSH):
```bash
ansible-playbook example_01.yml
```
![Test Run](./docs/test_run.png)



## Structure

### Inventory
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
            localhost:
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

When building your inventory be careful with naming a group, because groups with the same name will be grouped together when called directly. Whis is also known as `metagroup`  
This can be seen when running the following example:
```bash
ansible-playbook example_grouping.yml
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


### Playbook
---
[Playbook guide](https://docs.ansible.com/ansible/latest/playbook_guide/index.html)  
[Official examples](https://github.com/ansible/ansible-examples)

[List of playbook keywords](https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html)

A `playbook` is the starting point similar a setup script that is executed.  
In a `playbook` you define a `group` from the inventory which will be targeted by its content, using the `hosts` variable.
```yaml
---
- hosts: linux_server
...

```
It is also possible to exclude one or multiple sub-groups:
```yaml
---
- hosts: linux_server !webservers
...

```
And wildcards:
```yaml
---
- hosts: linux_*
...

```

It can also contain connection settings to override those in the inventory.  
You can also define settings like `become:` (default: false) to define if everything should be run with privilege escalation (sudo) or `gather_facts:` (default: true) to collect [facts](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html#id3) which contain information about the target System like operating system, IP and more, or `serial:`to define how many Systems should be configured at the same time and many more settings.  
It is also possible to define variables under `vars_prompt:` to ask for input for specific variables that you don't want to write down in any file.

After the first 'configuration' blog it is possible to define one or all of the following sections:
- pre_tasks: list of tasks executed before roles
- roles: list of roles to be imported
- tasks: list of tasks executed after roles
- post_tasks: list of tasks executed after tasks section
- handlers: list of tasks, but ony run when a task contains the `notify` keyword and has the state changed.

Futhermore you can define multiple of these sets in one `playbook`:
```yaml
---
- name: Configure Linux server
  hosts: linux_server
  tasks: ...

- name: Configure Windows server
  hosts: windows_server
  tasks: ...
...

```

A `playbook` is run with the command:
```bash
ansible-playbook <playbook_name>.yml
```
With `ansible-playbook --help` you can also check out its multiple additional options.


### Tasks
---
A task is a singular action that should be run. It can be anything from a simple console command, to disk formatting, network configuration and more. Every possible action is grouped in `namespace` and a [collection](https://docs.ansible.com/ansible/latest/collections/index.html) reaching from [builtin](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/index.html#plugins-in-ansible-builtin) to [community managed](https://docs.ansible.com/ansible/latest/collections/community/index.html).  
There is also the [ansible-galaxy](https://galaxy.ansible.com/ui/) which contains even more collections outside of the official documentation.

The `example_tasks.yml` file contains basic examples in how tasks are structured.
```bash
ansible-playbook example_tasks.ym
```
A task can additional to it's action contain where, when and with what privileges it should run and more.

Each task is at the end like a `Python` module defined in a `collection` within a `namespace`.  
This is also known as [Fully Qualified Collection Name (FQCN)](https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html#term-Fully-Qualified-Collection-Name-FQCN).  
While it is possible to only use the `module` name (like `debug`) it is highly recommended to use the `FQCN` to ensure you don't accidentally run a `module` from a different `collection` or `namespace` that has the same name.


### Handlers
---
[Handlers guide](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_handlers.html)
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

Important:
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

By default, handlers run after all the tasks in a particular play have been completed. Notified handlers are executed automatically after each of the following sections, in the following order: pre_tasks, roles/tasks and post_tasks. This approach is efficient, because the handler only runs once, regardless of how many tasks notify it. For example, if multiple tasks update a configuration file and notify a handler to restart Apache, Ansible only bounces Apache once to avoid unnecessary restarts.
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
The meta: flush_handlers task triggers any handlers that have been notified at that point in the play.

Once handlers are executed, either automatically after each mentioned section or manually by the flush_handlers meta task, they can be notified and run again in later sections of the play.

Handlers from roles are not just contained in their roles but rather inserted into the global scope with all other handlers from a play. As such they can be used outside of the role they are defined in. It also means that their name can conflict with handlers from outside the role. To ensure that a handler from a role is notified as opposed to one from outside the role with the same name, notify the handler by using its name in the following form: role_name : handler_name.

Handlers notified within the roles section are automatically flushed at the end of the tasks section but before any tasks handlers.


### Roles
---

add a name when calling a role because Ansible otherwise may not run a role if it has been defined a second time in the same playbook.

There is also the [ansible-galaxy](https://galaxy.ansible.com/ui/) which contains even more roles.

### Vars
---

[Variable Precedence](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#understanding-variable-precedence)  
[Variable merging](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/meta_module.html)  
[Facts and magic variables](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html)  
[Custom Facts](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html#facts-d-or-local-facts)   
[Special Variables](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html)

### Conditions and Loops
---

### Grouping and Error handling
---
[Block guide](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_blocks.html)


### Jinja2 Templates
---
[Template testing with lookup](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_lookup.html)  
[Template testing with python script](https://github.com/Antari-yan/python_jinja2_renderer)

### Vault
<hr style="border:0,1px solid grey">


## Extras
There are some additional tools provided by Ansible that may be of interest, but since I haven't used them yet, they will only be listed here.

### Pull mode
[ansible-pull](https://docs.ansible.com/ansible/latest/cli/ansible-pull.html)

### Execution Environment
[Containerized Execution Environments](https://ansible.readthedocs.io/en/latest/getting_started_ee/index.html)  
[Ansible Builder guide](https://ansible.readthedocs.io/projects/builder/en/latest/)
