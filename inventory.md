---
title: inventory
tags: 3, 3days, 4days, 5days
---

# inventory

## Inventory erstellen
Alle Systeme müssen im Inventory, einer einfachen Text-Datei im INI-Format, eingetragen sein.
Die Datei soll als `code/inventory` erstellt werden.
### Listing `/home/qskills/code/inventory`:
```ini=
ubuntu1
```

## `ssh`-Schlüsselpaar erstellen und testen
1. ssh-Schlüsselpaar für `qskills@ansible-X` erzeugen:
`ssh-keygen -t ed25519`
2. Der öffentliche Schlüsselteil kommt auf ein Zielsystem:
`ssh-copy-id qskills@ubuntu1`
4. Test mit Passwort-losem Einloggen auf dem Zielsystem:
`ssh qskills@ubuntu1`
5. Testen mit dem Ansible-Befehl 
`ansible ubuntu1 -i ~/code/inventory -m ansible.builtin.ping`

## `ssh`-Login für alle Zielsysteme schaffen
1. Alle Systeme dem Ansible-Controller bekannt machen: 
   `for i in {redhat,debian,suse,ubuntu}{1,2} ; do ssh-keyscan -H $i >> ~/.ssh/known_hosts ; done`
3. Den im vorigen Schritt erzeugten Schlüssel verteilen: 
   `for i in {redhat,debian,suse,ubuntu}{1,2} ; do ssh-copy-id $i ; done`

   Hier muss ein letztes Mal das Passwort vom Ziel-Account `qskills@zielrechner` angegeben werden.
   
      w/sshpass:
      `for i in {redhat,debian,suse,ubuntu}{1,2} ; do SSHPASS=user sshpass -e ssh-copy-id $i ; done`
      
      Hier nicht...
5. Testen mit `ssh suse2`, damit sollte jetzt Passwort-loses Einloggen möglich sein

Der Schritt 2. stellt sicher, dass _alle_ öffentlichen Host-Schlüssel abgescannt werden. Das kann durchaus mehr als einer sein (beispielsweise Schlüssel auf Basis von unterschiedlichen Algorithmen).
Oft wird bei der Ersteinrichtung des Zielrechners bereits der Schlüssel vom Ansible-User vorinstalliert, sodass die Schlüsselverteilung entfallen kann.

## Alle Systeme im Inventory eintragen
`for i in {debian,redhat,suse,ubuntu}{1,2} ; do echo $i ; done > ~/code/inventory`

## Gruppieren nach [ansible.builtin.command module – Execute commands on targets — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/command_module.html) Distribution im Inventory
### Listing: `code/inventory`:
```ini=
# Gruppendefinitionen

[RedHat]
redhat1
redhat2

[Debian]
debian1
debian2

[Suse]
suse1
suse2

[Ubuntu]
ubuntu1
ubuntu2
```

<!--  ------------------------------------------------------------------------------------- -->
## Debug
### make the output more readable
```shell
export ANSIBLE_LOAD_CALLBACK_PLUGINS=1
export ANSIBLE_STDOUT_CALLBACK=yaml
```
-> wir im Kapitel configuration erklärt

## Aufgaben zum Inventory und der Variablen-Deklaration darin:
1. Deklariert eine Variable im Inventory, mit der das System `redhat2` auch unter dem Hostnamen `b1` mit dem Port `2222` erreichbar ist
2. Erstellt eine neue Gruppe namens `Webserver`, die die Systeme `ubuntu1` und `suse1` beinhaltet
3. Erstellt ferner eine Gruppe `Deployment`, die die Systeme `redhat1` und `suse2` beinhaltet
4. Erstellt nun eine Gruppe `Productive`, in der die beiden Gruppen `Webserver` und `Deployment` zusammengefasst sind

## Aufgabe: Lasst die Warnmeldung bei den Suse-Systemen verschwinden
Hilfestellung: 
Der verwendete Ansible-Interpreter (=Python-Interpreter) kann über die Variable `ansible_python_interpreter`  gesetzt werden.

[How to build your inventory — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html)
### Listing Inventory `/home/qskills/code/inventory`:
```ini=
### Einzelhosts

# b1 ansible_host=redhat2 ansible_port=2222


### Gruppendefinitionen

[RedHat]
redhat1
redhat2

[Debian]
debian1
debian2

[Suse]
suse1
suse2

[Ubuntu]
ubuntu1
ubuntu2

[Webserver]
ubuntu1
suse1

[Deployment]
redhat1
suse2


### Gruppen aus Gruppen

[Productive:children]
Webserver
Deployment


### Gruppenvariablen

### Variable für alle Rechner der Gruppe Suse
### Die Definition der Variable ansible_python_interpreter lässt die Warnmeldung
### bei Suse-Systemen verschwinden mit folgender Einstellung:
[Suse:vars]
ansible_python_interpreter=/usr/bin/python3.12
[all:vars]
ansible_python_interpreter=/usr/bin/python3
```


## Python

Sollte beispielsweise unter `suse` Python nicht in einer ausreichend neuen Version verfügbar sein,
(bspw. `The error was: SyntaxError: future feature annotations is not defined`)
muss Python in einer ausreichend neuen Version nachinstalliert werden:

### XOR
#### on target system
```bash
python3 --version
sudo zypper update

sudo zypper install python312
python3 --version

whereis python3.12
```
#### on controller
zwei ansible Module benötigen keinen python-Interpreter:
[ansible.builtin.raw module – Executes a low-down and dirty command — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/raw_module.html)
[ansible.builtin.script module – Runs a local script on a remote node after transferring it — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/script_module.html)
Befehle dürfen dann aber nicht interaktiv verwendet werden.

``` bash
ansible Suse -m raw -a 'sudo zypper update -y; sudo zypper install -y python312'
```
Der Interpreter muss dann in den Variablen der Gruppe direkt ausgewählt werden:
   `ansible_python_interpreter=/usr/bin/python3.12`
### check 
```bash
ansible -a'python3    --version' suse1 -v
ansible -a'python3.12 --version' suse1 -v
```


## Voraussetzungen für das Funktionieren von Ansible
1. Die beteiligten Systeme müssen über TCP/IP erreichbar sein, ggf. mit DNS-Auflösung
2. Der Ansible-Account auf dem Controller muss ein Schlüsselpaar haben (Erzeugen mit `ssh-keygen -t ed25519`)
3. Der Public Key muss auf die Zielsysteme zum Passwort-losen Einloggen kopiert werden
4. Die Zielsysteme müssen unter einem erreichbaren Namen oder IP-Adresse in einem Inventory (bei uns: `/home/qskills/code/inventory`) eingetragen sein
5. Der Ansible-User auf dem Zielsystem muss `root`-Rechte über einen Eintrag in `/etc/sudoers` erlangen können
6. Auf dem Controller muss zumindest das Software-Paket `ansible` (bei Red Hat: `ansible-core`) installiert sein (mit Python als Abhängigkeit)
7. Auf den Zielsystemen muss Python installiert sein

9. Windows-Zielsysteme brauchen die PowerShell 3.0 oder neuer, .NET 4.0 oder neuer und den WinRM Listener (https://docs.ansible.com/ansible/latest/os_guide/windows_setup.html)
