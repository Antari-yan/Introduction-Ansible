# Introduction to Ansible
Intention for this repository is to provide an introduction to Ansible while also providing a usable base structure with examples.  

<<<<<<< HEAD
:warning: WARNING
:-
The examples are tested on a `Debian` based System.  
Be aware of possible differences on other Systems.
=======
[[_TOC_]]
>>>>>>> refs/remotes/origin/main


## Table of Content
- [What is Ansible?](#what-is-ansible)
- [Installation](#installation)
    - [Directory structure](#directory-structure)
    - [SSH Configuration](#ssh-config)
        - [SSH-Agent](#ssh-agent)
        - [TPM 2.0](#tpm-20)
    - [Ansible Configuration Settings](#ansible-configuration-settings)
- [Inventory](#inventory)
- [Playbook](#playbook)
- [Vars](#vars)
- [Tasks](#tasks)
- [Roles](#roles)
- [Vault](#vault)





## What is Ansible?
[Documentation](https://docs.ansible.com/)


## Installation
```bash
sudo apt update
sudo apt install -y openssh-client python3 python3-pip ansible
pip install --user --upgrade pip
pip install --user --upgrade ansible
```

### Directory structure


### SSH Configuration
winrm

[Talk about SSH by Leyrer Part1 (german)](https://media.ccc.de/v/gpn20-8-besser-leben-mit-ssh)
[Talk about SSH by Leyrer Part2 (german)](https://media.ccc.de/v/gpn21-28-noch-besser-leben-mit-ssh#t=829)

[Talk about SSH by Leyrer (english)](https://media.ccc.de/v/mch2022-170-ssh-configuration-intermediate-level)

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

#### TPM 2.0
[Talk about SSH by Leyrer Part2 (german)](https://media.ccc.de/v/gpn21-28-noch-besser-leben-mit-ssh#t=829)



### Ansible Configuration Settings
https://docs.ansible.com/ansible/latest/reference_appendices/config.html

To generate a fully commented-out example `ansible.cfg` run:
```bash
ansible-config init --disabled > ansible.cfg
```

Or to also include existing plugins:
```bash
ansible-config init --disabled -t all > ansible.cfg
```



## Inventory
.ini file


Groups with the same name will be grouped together
suffix or prefix like `prod_` and `staging_`


## Playbook

## Vars

[Special Variables](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html)

## Tasks

## Roles

## Vault
