# Vault
---
[Vault Guide](https://docs.ansible.com/ansible/latest/vault_guide/)

Ansible provides `vault` capabilities to encrypt files or strings.  
This part is kept short to give you a rough idea on what you can do with it,
but it is recommended to use a proper `Vault` like [HashiCorp Vault](https://www.hashicorp.com/products/vault),
because it provides permission management and more.  
The Ansible vault is essentially just an encrypted file providing limited security improvement.

You can use either fully encrypted files containing multiple variables or a singular encrypted string.  
In both cases you have to provide a password through either prompt or a file:
```bash
ansible-playbook example_vault.yml --ask-vault-password
# password: test
```
```bash
ansible-playbook playbook.yml --vault-password-file
```

Should you need to provide different passwords for your vault files/strings,
you have to provide for each of them a `Vault ID` with `--vault-id label@source`.

It is also possible to provide the information in the `ansible.cfg`, e.g.:
```ini
ask_vault_pass=False
vault_encrypt_identity=
vault_identity=default
vault_password_file=
```


## Using a file
Create a `Vault` file:
```bash
env EDITOR=nano ansible-vault create <Vault>.yml
```

edit a `Vault` file:
```bash
# Using default editor (vi)
ansible-vault edit <Vault>.yml

# Specifying editor
env EDITOR=nano ansible-vault edit <Vault>.yml
```

```yaml
- name: Vault
  hosts: localhost
  vars_files:
    - "vault.yml"
```
```bash
ansible-playbook <name>_playbook.yml --ask-vault-pass
```


## Using a string
To create a single encrypted string that can be used as a variable run:
```bash
ansible-vault encrypt_string secure_password
# Type in your desired password
```
This returns a multiline string that can be added in full directly in a playbook.
```yaml
  vars:
    my_secret: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          35643133326464663036346464306666623738343062636663666536346239396233626331346435
          6335623333626634316262663337336533333730353633660a346530316433656533613332363438
          34303163353263353263613938653239353266393735626361346233323831653061336338653162
          3637626338613261360a323938656564383734303139383135343662313230663839383738646430
          3433
```
