# Practical Tasks

1. Change in `example_01.yml` the target Host from `localhost` to your current Host connected with SSH (Hint: Create an SSH Key and add it to `~/.ssh/authorized_keys`)
2. Create a Playbook to install the following packages with their respective state in a single task
  - Packages:
    - vim: absent
    - nano: latest
    - htop: present
    - tree: present
3. Create a Playbook to install Docker
4. Create a role that setups an NFS Server/Client
    - Packages:
      - Server: `nfs-kernel-server`
      - Client: `nfs-common`
    - Services:
      - Server: `nfs-kernel-server.service`
    - Through Host Variables `nfs_server` and `nfs_client` the Server/Client is installed
    - NFS dir should be `/opt/nfs_data` set through a variable
    - NFS requires a `/etc/exports` file created through a template with the following template result:
      - `/mnt/nfs_storage/data *(rw)`
    - The Client should mount the NFS volume to `/mnt/nfs_data`