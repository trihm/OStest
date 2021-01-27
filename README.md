# Migrate Server Project

This is an Ansible-project dealing with moving credentials; database, web container, and its data.

## Tested environment

This project works well with ansible version 2.9.16 and python version 2.7.16. OS using on both sides of the transfer is Debian 10.

## inventory

- In this inventory, host or group will be defined in **hosts**. **hosts** is written in yaml structure.

- In this inventory, host or group variables will be store in separated files. The name of these files is corresponding with defined **host/group** name.

- **group_vars/all** file acts as global variable storing file.

# Use instruction

## Preparation

On Ansible-host, some ansible-module must be installed before running playbooks in this repository.

```bash
ansible-galaxy collection install -r requirements.yml
```
## Define variables

- **inventory\create_ansible_user_env\group_vars\all**
This file defines all global the variable in the user root environment used by **create_ansible_user.yml** playbook.
**ansible_uid** value must be higher than the highest uid exist on all host.
```yaml
ansible_uid: 1050
ssh_public_key: *keyvalue
```
- **inventory\create_ansible_user_env\hosts**
This file defines host, group in the user root environment used by **create_ansible_user.yml** playbook.
All hosts using must be defined in this files as below form:
```yaml
all:
  hosts:
    xxx.xxx.xxx.xxx.:
    yyy.yyy.yyy.yyy.:
```
If there is more than one destination host, please also define below the last IP as this form:
```yaml
all:
  hosts:
    xxx.xxx.xxx.xxx:
    yyy.yyy.yyy.yyy:
    zzz.zzz.zzz.zzz: #Another IP
```
- **inventory\data_trans_env\group_vars\all**
This file defines all global variables in the user ansible environment used by **play-book.yml** playbook.

```yaml
ansible_user: ansible
#private key must be a pair with ssh_public_key defined in simple_inventory\root_user\group_vars\all
ansible_ssh_private_key_file: /path/to/ssh/private/key
ansible_become: yes
ansible_become_method: sudo
DestIP:
  - yyy.yyy.yyy.yyy
  - zzz.zzz.zzz.zzz #If there is more than one destination host.
SourceIP: xxx.xxx.xxx.xxx #IP of the source OS.
dirpath_create: /opt/rsync #Path to folder used in migration progress.
```
- **inventory\data_trans_env\hosts**
This file defines all host/group used in the user ansible environment by **data-trans.yml** playbook.
Variables are defined as below form:
```yaml
---
all:
  children:
    SourceOS:
      hosts:
        xxx.xxx.xxx.xxx:
    DestOS:
      hosts:
        yyy.yyy.yyy.yyy:
        zzz.zzz.zzz.zzz: #If there is another destination OS.
```
- **inventory\data_trans_env\host_vars\xxx.xxx.xxx.xxx**
```yaml
dbcontainername: db #This is the database container name.
```
## Prepare encrypted variables.
- Create an encrypted file as below form. Assume the file path is **/path/to/encrypted/file**
```yaml
encrypt_DBpassword: 123 #Please change. This value is root password of the database in container."
```
- Encrypt file with **ansible-vault**.
```bash
ansible-vault encrypt /path/to/encrypted/file
```
Ansible will ask for **New Vault password**, input twice to create.
## Define ansible.cfg
This file will set default argument for ansible command. Ansible can work without this file, it will use default **ansible.cfg** at **/etc/ansible/ansible.cfg**
```yaml
[defaults]
inventory = ./inventory/data_trans_env/hosts
host_key_checking = False
log_path = ./out_put.log #This is where you save output from ansible 
debugger = always
```
## Run Playbook.
With this project, please run **create_ansible_user.yml** playbook first. The command runs this playbook is shown as below:
```bash
ansible-playbook create_ansible_user.yml -i inventory/create_ansible_user_env/hosts -u root --ask-pass #add -vv to run in debug mode
#Ansible will ask for SSH password. Please input your root password to proceed.
```
Run **data-trans.yml** playbook as command shown below:
```bash
ansible-playbook data-trans.yml -e "@/path/to/encrypted/file" --ask-vault-pass #add -vv to run in debug mode
#Ansible will ask for Vault password created at **Prepare encrypted variables** step. Please input your Vault password to proceed.
```