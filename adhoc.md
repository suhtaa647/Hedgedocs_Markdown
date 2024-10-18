---
title: adhoc
tags: 2, 3days, 4days, 5days
---

# ad-hoc
## Erste Schritte unter Ansible
Die Befehle werden als User `qskills` ausgeführt.


### Ad-Hoc Befehle
ACHTUNG: Fehler sind erwünscht!

Ansible kann Befehle sofort auf Zielsystemen ausführen.


```bash=
ansible
### ansible: error: the following arguments are required: pattern
### positional argument pattern: host pattern

ansible ubuntu1
### ERROR! No argument passed to command module

ansible ubuntu1 -m ansible.builtin.ping
### [WARNING]: No inventory was parsed, only implicit localhost is available
### [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'
### [WARNING]: Could not match supplied host pattern, ignoring: ubuntu1

ansible localhost -m ansible.builtin.ping

### create inventory
mkdir /home/qskills/code
echo ubuntu1 > /home/qskills/code/inventory


ansible ubuntu1 -m ansible.builtin.ping -i /home/qskills/code/inventory

### BE LAZY
# export ANSIBLE_INVENTORY=/home/qskills/code/inventory
echo -e "[defaults]\ninventory = /home/qskills/code/inventory" > /home/qskills/.ansible.cfg       ### will be used if no `ansible.cfg` is found in the current working directory
echo -e "[defaults]\ninventory = /home/qskills/code/inventory" > /home/qskills/code/ansible.cfg ### *

### REASURE EVERYTHING IS STILL FINE
ansible localhost -m ansible.builtin.ping

ansible all -m ansible.builtin.ping

ansible all,localhost -m ansible.builtin.ping


ansible ubuntu1 -m ansible.builtin.ping
### ubuntu1 | UNREACHABLE! => {
###     "changed": false,
###     "msg": "Failed to connect to the host via ssh: qskills@ubuntu1: Permission denied (publickey,password).",
###     "unreachable": true
### }


ansible ubuntu1 -m ansible.builtin.ping -k
### ubuntu1 | FAILED! => {
###     "msg": "to use the 'ssh' connection type with passwords or pkcs11_provider, you must install the sshpass program"
### }


sudo apt install sshpass

ansible ubuntu1 -m ansible.builtin.ping -k

### Datei ansible.cfg unter /home/qskills/code anlegen - Inhalt:
[defaults]
host_key_checking = False

### Umgebungsvariable exportieren
export ANSIBLE_CONFIG=/home/qskills/code/ansible.cfg


### with ssh-keys
ssh-keygen -t ed25519 -C qskills@controller
ssh-copy-id ubuntu1


### other module
ansible ubuntu1 -m ansible.builtin.setup


### default module with arguments
###                ┌───────────────── namespace
###                │       ┌───────── collection
###                │       │       ┌─ module
ansible ubuntu1 -m ansible.builtin.command -a "who"
ansible ubuntu1 -a "who"
ansible ubuntu1 -a "who -a"

```
<!--
Naiv nutzen wir das Modul `ansible.builtin.ping`wie folgt:
`ansible ubuntu1 -m ansible.builtin.ping`
Hier fehlt die Autorisierung, mit dem Schalter `-k`  schalten wir die PW-Abfrage von Ansible ein:
`ansible ubuntu1 -k -m ansible.builtin.ping`; das PW ist `b1s`.
Jetzt moniert der Ansible-Controller, dass `sshpass` zum Weiterreichen des PW fehlt. Also, flugs nachinstalliert: `sudo apt install sshpass`.
`ansible ubuntu1 -k -m ansible.builtin.ping`
Jetzt geht's!
Mit dieser Art Befehl kann mensch Befehle direkt einmalig vom Controller aus auf allen Zielsystemen ausführen lassen. Dazu gibt es das CLI `ansible`. `ansible` verwendet mit dem Schalter `-m` die gleichen Ansible-Module (siehe Definition Ansible-Modul) wie die Playbooks. Gegebenenfalls werden die Argumente mit dem Schalter `-a` für das Modul mit übergeben. Sinnervollerweise werden die Argumente mit `"` oder `'` vor der bash versteckt.
Fast immer werden Ad-hoc-Befehle nur mit dem Modul `ansible.builtin.setup` zur Informationsgewinnung ausgeführt: `ansible all -i code/inventory -m ansible.builtin.setup`.
In der Regel wird auch der Schalter `-b` verwendet, um gleich mit den erforderlichen Rechten aus der `/etc/sudoers` zu arbeiten.
-->

