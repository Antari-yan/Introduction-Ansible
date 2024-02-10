# Introduction to Ansible
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
- [Inventory](#inventory)
- [Playbook](#playbook)
- [Vars](#vars)
- [Collections](#collections)
- [Tasks](#tasks)
- [Roles](#roles)
- [Jinja2 Templates](#jinja2-templates)
- [Vault](#vault)


## What is Ansible?
Ansible is an IT automation tool. It can be used to Configure and Deploy Systems and Software.  
Through `SSH` it connects to Systems and Devices and manages them based on `YAML` and `Python` Code.  
Because of this Ansible is essentially agent-less. The target Systems only need to be able to accept SSH connections (e.g. openssh-server).  
It also has decent [Documentation](https://docs.ansible.com/ansible/latest/).


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
There are many ways in how to build the directory structure for Ansible.  
You can find additional examples [here](https://docs.ansible.com/ansible/latest/tips_tricks/sample_setup.html).  
The structure within this repository is just what worked best for me.

- In the base directory is the Ansible Configuration file `ansible.cfg` and the so-called playbooks `*.yml`/`*.yaml` (more later).
- The `inventories` directory contains one or multiple inventory files (all files are read automatically) that contain hosts that can be combined by groups.
- `host_vars` and `group_vars` can contain files named after hots or groups as they are defined in the inventories, they are matched automatically.
- In the `roles` directory are multiple roles that each contain individual tasks and generic files, variables etc. to fulfill those tasks.
- The `files` directory is not Ansible related, I just found it useful to have a separate directory for files that I want to copy to multiple target Systems (e.g. public SSH-Keys).

It is possible to have an own directory for the `playbooks` but you would have to define the path for the roles in the `ansible.cfg` using `roles_path`, because Ansible assumes that `roles` are stored in a sub-directory where the playbooks are.


### SSH Configuration
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
Linux (Debian):
```bash
sudo apt update
sudo apt install -y openssh-server
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

After the OpenSSh Server is installed add the SSH-Keys from the System where Ansible will be run from to the target System.
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
https://docs.ansible.com/ansible/latest/reference_appendices/config.html  
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


### Test
Download this repository and switch into its directory.  
Run the first example (Connections to localhost don't require SSH):
```bash
ansible-playbook example_01.yml
```
![Test Run](./docs/test_run.png)




## Inventory
.ini file


Groups with the same name will be grouped together
suffix or prefix like `prod_` and `staging_`


## Playbook

## Vars

[Variable Precedence](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#understanding-variable-precedence)
[Special Variables](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html)

## Collections
https://docs.ansible.com/ansible/latest/collections/index.html

## Tasks

## Roles

## Jinja2 Templates

## Vault
