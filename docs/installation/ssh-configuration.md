# SSH Configuration
---
Ansible is capable of reading the `~/.ssh/config`.  
Through this you con't have to manage any `hosts` file.  
Just add the Systems that you would like to mange with Ansible into the configuration like:
```bash
Host sample1
  HostName target-server.com
  User user1
  IdentityFile ~/.ssh/key_file_name1

Host sample2
  HostName 172.1.0.5
  User user2
  PreferredAuthentications password

Host sample3
  HostName 10.1.0.5
  IdentityFile ~/.ssh/key_file_name3
  ProxyJump jump-host

Host *
  User user_default
  Port 22
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/default_key_file
  IdentitiesOnly yes
  ForwardAgent no
  UseRoaming no
  SendEnv LANG LC_*
  Compression yes
```
It is recommended to avoid always reusing the same SSH-Key and also having one without passphrase.  
If having individual SSH-Keys and passphrases for each System is too tedious consider having one SSH-Key for a group of Systems or
one for each customer or whatever fits your infrastructure and requirements.
```bash
# Generating SSH Key
ssh-keygen -t ed25519 -a 64 -b 4096 -C "Ansible SSH Key" -f ~/.ssh/ansible_key
```

Here are some great talks about SSH and how it can be used in a good way:  
[Talk about SSH by Leyrer Part1 (german)](https://media.ccc.de/v/gpn20-8-besser-leben-mit-ssh)  
[Talk about SSH by Leyrer Part2 (german)](https://media.ccc.de/v/gpn21-28-noch-besser-leben-mit-ssh#t=829)

[Talk about SSH by Leyrer (english)](https://media.ccc.de/v/mch2022-170-ssh-configuration-intermediate-level)

Some are saying that to manage Windows Systems `winrm`(Windows Remote Management) can/should be used.  
I highly discourage you to do that, as you can simply use SSH.  
Also the example scripts that you may find online or by Microsoft to enable `winrm` usually aren't meant for production use and disable security features.


## SSH-Agent
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

## TPM 2.0
It is possible to have SSH-keys in the TPM 2.0 Chip, but I haven't used that yet.  
It is well explained here:
[Talk about SSH by Leyrer Part2 (german)](https://media.ccc.de/v/gpn21-28-noch-besser-leben-mit-ssh#t=829)


## Prepare target Systems
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
