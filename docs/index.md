# Introduction to Ansible
Intention for this repository is to provide a basic introduction to Ansible while also providing a usable base structure with examples.  
The goal is not to cover everything Ansible is capable of but enough to make practical use of it.

!!! note
    The examples are tested on a `Debian` based System.  
    Be aware of possible differences on other Systems.


## What is Ansible?
Ansible is an IT automation tool. It can be used to Configure and Deploy Systems and Software.  
It can be viewed, in a classical way, as a collection of installer script to ensure a System reaches a desired state of operation,
but with the intend of making these scripts reusable on multiple Systems while making it possible to re-run
the same scripts on a System multiple times without making changes if the desired state has already been reached ([idempotent](https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html#term-Idempotency)).  
Through `SSH` it connects to Systems and Devices and manages them based on `YAML` and `Python` Code.  
Because of this Ansible is essentially agent-less. The target Systems only need to be able to accept SSH connections (e.g. openssh-server).  
It also has decent [Documentation](https://docs.ansible.com/ansible/latest/).  
Key term definitions can be found [here](https://docs.ansible.com/ansible/latest/reference_appendices/glossary.html).
