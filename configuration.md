---
title: configuration
tags: 5, 3days, 4days, 5days
---
# configuration

## Debug
### callbacks
```shell
export ANSIBLE_LOAD_CALLBACK_PLUGINS=1
export ANSIBLE_STDOUT_CALLBACK=yaml
```
```shell
ANSIBLE_STDOUT_CALLBACK=oneline ANSIBLE_LOAD_CALLBACK_PLUGINS=true ansible all -a  who
```

## callbacks
[Callback plugins — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/plugins/callback.html)
### list all callbacks
```shell
ansible-doc -t callback -l 
```
### configuration
#### `/home/qskills/code/ansible.cfg`:
``` ini
[defaults]
inventory = /home/qskills/code/inventory
roles_path = roles
### Use the YAML callback plugin.
stdout_callback = yaml
### Use the stdout_callback when running ad-hoc commands. 
### (Be carefull some output may not be displayed anymore)
# bin_ansible_callbacks = True

### can be overriden with 
### ANSIBLE_STDOUT_CALLBACK=json ansible ubuntu1 -m setup
```




## Setup-Szenarien für Ansible
### User `root@controller` oder `username@controller` auf dem Ansible-Controller, User `root@zielsystem` auf dem Zielsystem
Hier wird der Public Key vom User `root@controller` zu den `root@zielsystem:/root/.ssh/authorized_keys` hinzugefügt. Damit kann sich Ansible passwortfrei einloggen.
Das ist ein seltenes Szenario, da dem User `root` in der Regel kein Remote Login erlaubt ist.

### User `username@controller`, User `ansible_user@zielsystem` mit `sudo`
Der Public Key vom User `username@controller` wird `ansible_user@zielsystem:/home/ansible_user/.ssh/authorized_keys` hinzugefügt. Der User `ansible_user@zielsystem` erledigt seine Verwaltungsaufgaben mittels `sudo` und muss dafür mit einem Eintrag in `/etc/sudoers` die Berechtigung bekommen.
#### Ausschnitt aus `/etc/sudoers.d/ansible_user_permissions`
```
### ...
# Allow members of group sudo to execute any command
# The Ansible user should be in the sudo group
%sudo   ALL=(ALL) NOPASSWD:ALL
### ...
```
Der Ansible-User (bei uns `qskills@zielsystem`) muss dann Mitglied der Gruppe `sudo` sein. Ansible muss dann mit der `become`-Option (Schalter: `-b`) aufgerufen werden.
Wir verwenden dieses Szenario. Dieses Szenario ist auch am häufigsten, da kein `root`-User auf dem Ansible-Controller und dem Zielsystem über ssh verwendet wird.

## config
[Ansible Configuration Settings — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/reference_appendices/config.html)
