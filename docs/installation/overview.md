# Overview
Install Ansible on your System:
```bash
sudo apt update
sudo apt install -y openssh-client python3 python3-pip ansible
pip install --user --upgrade pip
pip install --user --upgrade ansible
```
I always recommend to update `ansible` via `pip` to the latest version, this way you will be warned in advance about `deprecations` when using it.  
Alternatively use tools like [ansible-builder](https://github.com/ansible/ansible-builder) to create reliable and reproducible container images.

Default search paths are:

- `~/.ansible/`
- `/usr/share/ansible/`
- `/etc/ansible/`

But it is also possible to use a different path, like we will be this repository structure.
