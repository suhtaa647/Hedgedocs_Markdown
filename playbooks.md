---
title: playbooks
tags: 8, 3days, 4days, 5days
---
# playbooks


## quickest copy & paste method for all the listings

1. copy the content
2. select the path
3. on alacritty with ssh-connection to controller
    a). open file:
        `middleClick` `C-a` `nvim` `SPACE` `ENTER`
    b). insert content:
        `C-S-v`
    c). save/create:
        `C-s :q` or even faster `:x`


## Ansible Playbooks
Ein Playbook kann 1-n Plays enthalten. Ein Play vereint 1-n Tasks (Abschnitt `tasks: `), eine Zieldefinition (Abschnitt `hosts: `) und optional Variablen-Setzungen (Abschnitt `vars: `) in einer YAML-Struktur.
Für die Übersichtlichkeit macht es Sinn, nur ein Play pro Playbook zu haben. Für komplexere Szenarien wird statt Playbooks eine Ansible-Rolle verwendet.
Das Playbook des Beispiels wird mit dem Befehl 
`ansible-playbook -i code/inventory code/playbooks/00_Verzeichnis_anlegen.yml`
ausgeführt. Playbooks werden wir immer im Ordner `/home/qskills/code/playbooks` anlegen, da `ansiblels` zum Starten das directory playbooks benötigt.)

### Syntax Checks, Dry Runs und Diffs
`ansible-playbook` kennt den Schalter `--check` / `-C`, damit lässt sich ein Dry Run durchführen.
Achtung: Bei komplexeren Playbooks, beispielsweise mit ausgewerteten `register`-Variablen, testet der Dry Run nicht die komplette Funktionalität.

Die Veränderungen die durch eine Task herbeigeführt wird lässt sich mit `--diff` / `-D` visualisieren. 

Die Syntax von einem Playbook lässt sich mit dem Schalter `--syntax-check` prüfen.

### Aufgabe
Schreiben Sie ein Playbook, in dem ein Task mit dem Modul `ansible.builtin.file`
sicherstellt, dass das Verzeichnis `/home/qskills/foo/` auf allen Zielsystemen vorhanden ist.

Hinweis: Speichern sie das Playbook im Verzeichnis `playbooks` ab, das direkt neben dem `inventory` liegt (`/home/qskills/code`) und geben sie der Datei die Endung `.yml`
`mkdir ~/code/playbooks`

#### `~/code/playbooks/00_Verzeichnis_anlegen.yml`
```yaml=
---
- name: Verzeichnis anlegen

  hosts: all
  
  tasks:
    - name: Stelle sicher, dass das der Ordner angelegt ist.
      # Diese Task wird mit dem angemeldeten Benutzer
      # ausgeführt. Wenn wir das Playbook oder die Task
      # nicht mit "become" ausgeführt haben, ist das
      # bei uns qskills. Das hat zur Folge, dass Eigentümer-
      # und Gruppenzuordnung automatisch richtig werden
      ansible.builtin.file:
        path: /home/qskills/foo/
        state: directory 
```
Vergleiche die Benennungen.

### mehrere Tasks in einem Playbook

#### `~/code/playbooks/01_Controller_kopieren.yml`
```yaml=
---
- name: Kopiert das Controller-System

  hosts: 
    - ubuntu1
    # - ubuntu2
    # - Ubuntu
  vars:
    variablenname: WertderVariable
  
  tasks:
    - name: User qskills soll da sein
      ansible.builtin.user:
        name: qskills
        state: present
      become: yes

    - name: Stelle das Verzeichnis /home/qskills/foo sicher
      ansible.builtin.file:
        name: /home/qskills/foo
        state: directory
        owner: qskills

    - name: Verteile das Inventory
      ansible.builtin.copy:
        src: /home/qskills/inventory
        dest: /home/qskills/foo/inventory
    
    - name: Sicherstellen, dass Ansible installiert ist
      ansible.builtin.package:
        name: ansible
        state: present
      become: yes
```
Anmerkung: Dieses Playbook kümmert sich noch nicht um die ganzen SSH-Schlüsselaustausche, die das Vertrauen der Zielsysteme in den neugeborenen Controller erforderte.

ACHTUNG: Fehler sind erwünscht!


### Ansible Facts
Die Ansible Facts werden durch das Modul `ansible.builtin.setup` geliefert. Sie lassen sich als "Quasi-Variable" verwenden.

ACHTUNG: Fehler sind erwünscht!

#### `~/code/playbooks/02_Get_Host_Infos.yml`
```yaml=
---
- name: Get Infos from hosts
  hosts: all
  # gather_facts: no
  tasks:
    - name: Print infos of host
      ansible.builtin.debug:
        msg:
          - "Der Hostname ist: {{ ansible_hostname }}"
          - "Das Betriebssystem ist: {{ ansible_distribution }}"
          - "Das genaue Release ist: {{ ansible_distribution_release }}"
          - "Version: {{ ansible_distribution_version }}"
          - "Die IP-Adresse lautet: {{ ansible_default_ipv4.address }}"
          - "Die Ethernet MAC-Adresse ist: {{ ansible_default_ipv4.macaddress }}"
          - "Die Maschine hat ingesamt {{ ansible_memtotal_mb }} MB RAM"
          - "Die Maschine hat zur Zeit noch {{ ansible_memfree_mb }} MB RAM frei"
          - "Die VM ist virtualisiert mit: {{ ansible_virtualization_tech_guest }}"
          - "Die Python Version ist: {{ ansible_python_version }}"
```
<!-- ---------------------------------------------- -->


## Ansible-Module

### Definition Ansible-Modul
> Ein Ansible-Modul abstrahiert eine Änderung auf dem Zielsystem von den zielsystemspezifischen Befehlen.
> Es ist ein Python-Skript, welches die idempotenten Eigenschaften von Ansible realisiert. 
> Dazu vergleicht es den Ist-Zustand im Zielsystem mit dem Soll-Zustand aus dem Ad-hoc-Befehl, dem Ansible Playbook oder der Ansible Role, bestimmt die notwendigen Aktionen und führt sie aus.
> Es liefert ein festgelegtes Set von Rückgabe-Werten:
> [Return Values — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/reference_appendices/common_return_values.html#common-return-values)


### Definition Playbook
> Ein Playbook beschreibt gewünschten Zustand einer ganzen Umgebung. 
> Die Umgebung steht im Inventory. 
> Mit der Auswahl von verschiedenen Inventorys lassen sich darüber mehrere Umgebungen mit den gleichen Playbooks bearbeiten.
> Die Sprache des Playbooks ist YAML (bitte an die erste Zeile mit `---` denken!). 
> Die Abschnitte des Playbooks sind als Liste, mit den Elementen 
> `hosts:`, 
> `vars:`(fakultativ) und dem Hauptabschnitt 
> `tasks:` organisiert. 
> Die einzelnen Abschnitte sind wiederum Listen, so besteht `tasks:` aus einer Liste von Tasks.
> Weiterhin werden eine oder mehrere Rollen immer aus einem Playbook heraus aufgerufen.


## Variablen bei Ansible
Variablen können bei Ansible an vielen Stellen definiert werden.
- role defaults
- inventory
- playbooks
- role vars
- tasks
- registered vars (facts)
- command line (-e "NAME=VALUE")

Die Variablen-Präzedenz ist hier allgemein dokumentiert:
[Controlling how Ansible behaves: precedence rules — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/reference_appendices/general_precedence.html)
Konkrete Liste mit der Präzedenz bei Variablen:
[Using Variables — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#ansible-variable-precedence)

### Arten
[Special Variables — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html#magic)
### Aufgabe:
Deklarieren Sie in einem Playbook namens `Variablen_Demo.yml` 
1. drei Variablen und lassen Sie diese in einem Task mit dem
`ansible.builtin.debug`-Modul und dem Parameter `msg` ausgeben.
2. eine Liste von Verkehrsmitteln und lassen Sie eines davon in einem Task mit dem
`ansible.builtin.debug`-Modul und dem Parameter `msg` ausgeben.

ACHTUNG: Fehler sind erwünscht!
#### `~/code/playbooks/03_Variablen_Demo.yml`
```yaml=
---
- name: Variablen-Demo
  hosts: all
  vars:
    var1: 1
    var2: 2
    var3: 3
    verkehrsmittel:
      - Dreirad
      - Fahrrad
      - Auto
      - Bus
  tasks:
    - name: Zeige Variablen aus den Playvars
      ansible.builtin.debug:
        msg: "{{ var1 }} {{ var2 }} {{ var3 }} {{ var4 }}"
    - name: Zeige Variablen aus den Taskvars
      vars:
        var1: 1111
      ansible.builtin.debug:
        msg: "{{ var1 }} {{ var2 }} {{ var3 }}"
    - name: Zeige einen Listen-Wert an
      ansible.builtin.debug:
        msg: "Die fortgeschrittene Form der Fortbewegung ist {{ verkehrsmittel[3] }}"
```
```shell
ansible-playbook playbooks/Variablen_Demo.yml -e var1=9999
```

### Registrierte Variablen
Registrierte Variablen nehmen das Ergebnis einer Task in einem Dictionary auf. Diese Dictionary-Variable kann dann weiterverarbeitet oder für eine weitere Task ausgewertet werden.
#### `~/code/playbooks/04_Registered_Variable_Demo.yml`
```yaml=
---
- name: A task where the result goes into a registered variable
  
  hosts: ubuntu2
  
  tasks:
    - name: Ausgabe aus id in eine registrierte Variable
      ansible.builtin.command: "id"
      register: registrierte_variable
      
    - name: Ausgabe der Registrierten Variablen
      ansible.builtin.debug:
        msg: "Hier der vollständige Inhalt des Dictionarys: {{ registrierte_variable }}"
        
    - name: Ausgabe von STDOUT des Befehls id von oben
      ansible.builtin.debug:
        msg: "Hier der Wert vom Key stdout des Dictionarys: {{ registrierte_variable['stdout'] }}"
```


### Sonderlocken `ansible.builtin.command` und `ansible.builtin.shell`
Das Modul `ansible.builtin.command` dient dazu, einen Befehl direkt auf dem Zielsystem auszuführen. Es ist eher als Notnagel zu sehen, da es nach Aufruf, unabhängig vom Ergebnis, immer den Status "OK" meldet. Das idempotente Verhalten eines gesunden Moduls muss hier mühsam individuell mit `changed_when` und `failed_when` nachgebaut werden.

## When-Bedingung
Das `when`-Statement kann am Ende eine Task angegeben werden und lässt eine Bedingung für die Ausführung der Task zu.
[Conditionals — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_conditionals.html)

### Aufgabe
1. Schreiben Sie ein Playbook mit einem Task, in dem mit dem Modul
ansible.builtin.package sichergestellt wird, dass das Programm git installiert
ist, sofern die Zielsysteme den ansible_hostname ubuntu1 oder suse2 haben.
2. Fügen Sie dem Playbook einen weiteren Task hinzu, in dem mit dem Modul
ansible.builtin.package sichergestellt wird, dass das Paket curl auf allen
Systemen vorhanden ist, die nicht zur ansible_os_family Suse oder Debian
gehören.
#### `~/code/playbooks/05_Auswerten_von_when.yml`
```yaml=
---
- name: Auswerten einer when-Bedingung

  hosts: all

  tasks:
    - name: Paket vorhanden bei den Hosts ubuntu1 und suse2
      ansible.builtin.package:
        name: git
        state: present
      when: ansible_hostname == "ubuntu1" or ansible_hostname == "suse2"
      # alternativ in einer Liste:
      # when: ansible_hostname in ["ubuntu1", "suse2"]
      become: yes

    - name: curl fuer alle Distributionen ausser Suse- und Debian-Familie
      ansible.builtin.package:
        name: curl
        state: present
      when: ansible_os_family != "Suse" and ansible_os_family != "Debian"
      # alternativ:
      # when: ansible_os_family not in ["Suse", "Debian"]
      become: yes
```

#### `~/code/playbooks/06_Idempotenz_basteln.yml`
```yaml=
---
- name: Idempotenz

  hosts: ubuntu1
  
  tasks:
    - name: Der Befehl id 
      ansible.builtin.command: "id"
      register: register_ergebnis_von_einem_befehl
      changed_when: false

    - name: Gesamte registrierte Variable ausgeben
      ansible.builtin.debug:
        msg: "Der gesamte Inhalt von der Registrierten Variable ist {{ register_ergebnis_von_einem_befehl }}"

    - name: Ergebnis prüfen und ausgeben
      ansible.builtin.debug:
        msg: "Der Befehl id war erfolgreich!"
      when: "register_ergebnis_von_einem_befehl['rc'] == 0"

```
[Error handling in playbooks — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_error_handling.html)

#### `~/code/playbooks/07_Prüfen_ob_Systeme_erreichbar_sind.yml`
```yaml=
---
- name: Prüfen der Erreichbarkeit von Adress-Erreichbarkeit vom Zielsystem aus

  hosts: ubuntu1
  
  vars:
    systeme:
      - host1
      - host2
      - www.b1-systems.de
  
  tasks:
    - name: Ping
      ansible.builtin.command: "ping -c 1 {{ item }}"
      register: register_Ping
      failed_when: "register_Ping['rc'] != 0"
      loop: "{{ systeme }}"
```

Das Modul `ansible.builtin.shell` arbeitet ähnlich, hier wird die eingestellte Shell aufgerufen und ihr der Befehl, beispielsweise ein selbstgeschriebenes Shell-Skript, übergeben. Dann lassen sich auch alle Shell-Funktionalitäten nutzen.
So ist es zum Beispiel sinnvoll, die Lokalisierungsvariable `LANG` für den Befehl auf `C` zu setzen:
```yaml
# ...
  ansible.builtin.shell:
    cmd: "LANG=C <Befehl>"
# ...
```
Das setzt die Sprache der Befehlsausgabe auf eine auf jedem System vorhandenen "C"-Sprache, sprich Standard-Englisch.
Diese Module sind eine süße Versuchung, alte imperative Gewohnheiten zu behalten. Sie widersprechen aber der deklarativen Ansible-Philosophie. Fast immer findet sich ein bereits vorhandenes Ansible-Module.

#### difference  `command`, `shell`
plain - vs /bin/bash
[ansible.builtin.command module – Execute commands on targets — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/command_module.html)

[ansible.builtin.shell module – Execute shell commands on targets — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/shell_module.html)

## Schleifen mit Ansible
### Aufgabe:
Schreibt ein Playbook, welches das Laufen der Dienste `ssh` und `cron` auf dem System `ubuntu2` sicherstellt!
Hinweise: Die Package-Namen sind `openssh-server` und `cron`, die Dienste-Namen sind `ssh` und `cron`.
Ihr braucht die beiden Module `ansible.builtin.package` und `ansible.builtin.service`.

#### `~/code/playbooks/08_Dienste.yml`

ACHTUNG: Fehler sind erwünscht!

```yaml
---

- name: a playbook, that should make sure with a loop that several services are installed and running

  hosts: ubuntu2

  vars:
    package_names:
      - 'openssh-server'
      - 'cron'
      # Folgendes nach dem ersten Lauf ent-kommentieren, um
      # die Schönheit der Trennung von Code und Daten zu genießen
      # - 'apache2'
    service_names:
      - ssh
      - cron
      # Folgendes nach dem ersten Lauf ent-kommentieren, um
      # die Schönheit der Trennung von Code und Daten zu genießen
      # - 'apache2'

  tasks:
    - name:
      ansible.builtin.package:
        name: "{{ item }}"
        state: present
      loop: "{{ package_names }}"

    - name:
      ansible.builtin.service:
        name: "{{ item }}"
        state: started
      loop: "{{ service_names }}"
      
```

##### Execution
`ansible-playbook ~/code/playbooks/08_Dienste.yml`
`ansible-playbook ~/code/playbooks/08_Dienste.yml -e "package_names=['openssh-server','cron','apache2']" -e "service_names=['ssh','cron','apache2']"`
