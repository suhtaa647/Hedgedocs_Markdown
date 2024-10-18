---
title: roles
tags: 9, 3days, 4days, 5days
---
# roles

Eine Ansible-Rolle beruht auf einer festgelegten Verzeichnisstruktur, in dem die einzelnen YAMLs mit allen Einstellungen und Beschreibungen zur Rolle liegen.
Eine Rolle lässt sich einfach mit dem Befehl `ansible-galaxy init <name der rolle>` anlegen.
Vorbereitend kann die Utility `tree` mit `apt install tree`  installiert werden.

`ansible-galaxy role init /home/qskills/code/roles/Example`

[Roles — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html#role-directory-structure)

    
## Ordnerstruktur
    
## roleVariables

### defaults
- variables the user will have to modify
- are used for not being forced to set defaults within jinja
  - ```jinja
    {{ variable|default("there", true) }}
    ```
  - otherwise ansible would fail

### vars
- role programming variables
- not userFacing
- avoid hardcoding

###  example
An initial admin username would be in defaults,
but dependency versions would be in vars.
    


## Jinja2
[Jinja — Jinja Documentation (3.1.x)](https://jinja.palletsprojects.com/en/3.1.x/)

[Template Designer Documentation — Jinja Documentation (3.1.x)](https://jinja.palletsprojects.com/en/3.1.x/templates/)


Die Templating-Sprache Jinja2 kann aus einer Template-Datei (`*.j2`) Textdateien/Konfigurationsdateien für die Zielsysteme erzeugen.
Im Template finden sich Variablen als Platzhalter, die das `template`-Modul bei der Ausführung durch ihre Werte ersetzt und somit die reinen Textdatei/Konfigurationsdatei auf dem Zielsystem erzeugt.
Die Syntax von Jinja2 ist gewöhnungsbedürftig, hält aber nichtsdestotrotzdem die Möglichkeiten von Schleifen, Bedingungen/If-Abfragen und Filter bereit.
Filter können die Inhalte von Variablen vor dem Verwenden "bearbeiten". Sie arbeiten wie die Möglichkeit in der Bash beispielsweise die Länge einer Variable `$variable` über `${#variable}` zu erhalten. Auch eine Kommandosubstitution in der Bash erinnert daran: `$(irgendein Programm mit Zeichenkettenausgabe)`setzt das Ergebnis des Programms an die Stelle von `$(irgendein Programm mit Zeichenketteausgabe)`.
Beispiele für Jinja2-Filter ist `dict2items`, welches ein Dictionary in eine Liste umwandelt. Andere Beispiele sind `capitalize` (die Bash hat das als `${variable^^}`) oder `join('ein oder mehrere Trennzeichen')` welches eine Liste in eine Trennzeichen-getrennte Zeichenkette verwandeln kann

### cf. Unterlage

### dotted notation
[Frequently Asked Questions — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/reference_appendices/faq.html#ansible-allows-dot-notation-and-array-notation-for-variables-which-notation-should-i-use)

### filter debugging
```ansible
"{{ ([ 'jumphost' ] + groups['all']) | product([':9100']) | map('join') | list }}"
```
```shell
ansible redhat1 -m debug -a "msg='{{ ([ \'jumphost\' ] + groups[\'all\']) | product([\':9100\']) | map(\'join\') | list }}'"
ansible redhat1 -m debug -a "msg='{{ ([ \'jumphost\' ] + groups[\'all\']) | product([\':9100\']) | map(\'join\') }}'"
ansible redhat1 -m debug -a "msg='{{ ([ \'jumphost\' ] + groups[\'all\']) | product([\':9100\'])}}'"
ansible redhat1 -m debug -a "msg='{{ ([ \'jumphost\' ] + groups[\'all\'])}}'"
ansible redhat1 -m debug -a "msg='{{  [ \'jumphost\' ]  }}'"
ansible redhat1 -m debug -a "msg='{{ ([ \'jumphost\' ] + groups[\'all\']) | product([\':9100\']) | map(\'join\') | list }}'"
```



## Schritt 1 für unsere Rolle: Anlegen der Rollenverzeichnisstruktur
Alle Rollen leben in dem Verzeichnis `code/roles/`. Es ist eine gute Idee, eigene Rollennamen mit Großbuchstaben anfangen zu lassen. Das dient der Übersichtlichkeit.
Legt die Rolle `Webserver` an!
Lösung:
```bash
mkdir -p ~/code/roles
cd ~/code/roles ### wg. Kommentaren
ansible-galaxy role init Webserver
cd ~/code
```
Die Rolle soll das Zielsystem in einen Webserver verwandeln. Dazu muss das passende Software-Paket installiert werden.

## Schritt 2 für die Rolle: `tasks/main.yml` schreiben/editieren
`tasks/main.yml`
Bitte folgende Datei als `tasks/main.yml`  in der Rolle anlegen oder die von `ansible-galaxy init` bereits erstellte Datei verwenden.
(Der vollständige Name ist:
`/home/qskills/code/roles/Webserver/tasks/main.yml`
)

## Schritt 3 für die Rolle: Playbook zum Aufruf schreiben
### `~/code/playbooks/RoleWebserver.yml`
```yaml=
---
# Implement Role Webserver
- name: Rollenaufruf
  hosts: all
  roles:
    - Webserver
    # - ../roles/Webserver
  become: yes
```
## Schritt 4: Testen der Rolle

ACHTUNG: Fehler sind erwünscht!

`ansible-playbook -i ~/code/inventory ~/code/playbooks/RoleWebserver.yml --limit suse1`
`ansible-playbook  ~/code/playbooks/RoleWebserver.yml -l suse1`

[Ansible Configuration Settings — Ansible Documentation - roles_path](https://docs.ansible.com/ansible/2.9/reference_appendices/config.html#envvar-ANSIBLE_ROLES_PATH)
[Ansible Configuration Settings — Ansible Community Documentation - confFile](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#the-configuration-file)
### XOR
`echo 'roles_path = code/roles' >> ~/.ansible.cfg`
`echo 'roles_path = roles' >> ~/code/ansible.cfg  ### *`



## Schritt 5: Sicherstellt, dass das Paket `apache2` installiert ist.
### `~/code/roles/Webserver/tasks/main.yml`
```yaml=
---
# tasks file for Webserver

- name: 'Ensure the webserver is available (main)'
  ansible.builtin.include_tasks:
    file: ./server.yml

```
### `~/code/roles/Webserver/tasks/server.yml`
```yaml=
---
- name: Make sure that the webserver package is installed
  ansible.builtin.package:
    name: apache2
    state: present
```
TODO
Test, ob der Webserver erfolgreich installiert wurde: Auf das `suse1`-System gehen und mit `which httpd` testen.
Test, ob der Webserver erfolgreich gestartet wurde: `curl suse1` vom Controller aus durchführen.
Bei Suse geht's nicht :-( -> Der Service muss noch gestartet werden.

## Schritt 6: Der Webserver soll laufend sein
### `~/code/roles/Webserver/tasks/server.yml`
```yaml=
---
- name: Make sure that the webserver package is installed
  ansible.builtin.package:
    name: apache2
    state: present
      
- name: Make sure that the webserver is running
  ansible.builtin.service:
    name: apache2
    state: started
```

## Schritt 7: Name des Package, Zustand des Package und des Services vom Webserver in Variablen ablegen
Die Variablen-Standardbelegungen werden im Verzeichnis `defaults` aufgeschrieben. 
Die dort hinterlegten Werte werden oft von Neubelegungen der Variablen an anderen Stellen der Rolle überschrieben. Im Prinzip dienen die Einträge in `/home/qskills/code/roles/Webserver/defaults/main.yml` der initialen Deklaration von Variablen.
Die Variablen-Namen sollen sein: `webserver_service`, `webserver_package` und `webserver_package_state`.
### `~/code/roles/Webserver/defaults/main.yml`
```yaml=
---
# defaults file for Webserver
webserver_service: apache2
webserver_package: apache2
webserver_package_state: present
```

vars: role-facing
defaults: defaults or user-facing - should be overridden in playbook ...; lowest precedence
## Schritt 8: Ersetzen Sie die Variablen 
`webserver_service`, 
`webserver_package` und `webserver_package_state`
in `defaults` in der Rolle und binden Sie sie entsprechend in die 
`~/code/roles/Webserver/tasks/server.yml` ein!
### `~/code/roles/Webserver/tasks/server.yml`
```yaml=
---
- name: Make sure that the webserver package is installed
  ansible.builtin.package:
    name: "{{ webserver_package }}"
    state: "{{ webserver_package_state }}"
    
- name: Make sure that the webserver is running
  ansible.builtin.service:
    name: "{{ webserver_service }}"
    state: started
```
## Schritt 9: Einarbeiten von Handlern
Der Webserver soll neu gestartet werden, wenn er upgedatet oder frisch installiert wurde (1. Handler).
Der Webserver soll seine Konfigurationsdateien neu laden, wenn sie verändert wurden (2. Handler).
Hinweis: Das Modul `ansible.builtin.service` kennt die Werte `restarted` und `reloaded` für das Argument `state`.

Editiert die Datei `~/code/roles/Webserver/handlers/main.yml`!
### `~/code/roles/Webserver/handlers/main.yml`
```yaml=
---
# handlers file for Webserver
# Falls der Webserver neu installiert wurde oder aktualisiert wurde
- name: Restart Webserver
  ansible.builtin.service:
    name: "{{ webserver_service }}"
    state: restarted

# Falls nur die Konfiguration des Webservers verändert wurde
- name: Reload Webserver
  ansible.builtin.service:
    name: "{{ webserver_service }}"
    state: reloaded
```

Die Datei `/home/qskills/code/roles/Webserver/tasks/main.yml` muss dann entsprechend modifiziert werden:
### `~/code/roles/Webserver/tasks/server.yml`
```yaml=
---
# tasks file for Webserver
- name: Make sure that the webserver package is installed
  ansible.builtin.package:
    name: "{{ webserver_package }}"
    state: "{{ webserver_package_state }}"
  # Falls die Webserver-Package neu installiert wurde, oder aktualisiert wurde,
  # soll der Webserver neu gestartet werden
  notify: Restart Webserver
  
- name: Make sure that the webserver is running
  # Falls der Webserver zwar installiert ist, aber nicht läuft, wird hier nochmal sicher
  # gestellt, dass er auch wirklich läuft.
  ansible.builtin.service:
    name: "{{ webserver_service }}"
    state: started
```

## Schritt 10: Einbinden distributionsspezifischer Variablen-Belegungen
RedHat hat entschieden, die Apache2-Package, anders als alle anderen Distributionen, `httpd` zu nennen.
Wir können das in unserem Beispiel abbilden mit einem anderen Wert für die Variablen `weberserver_package`  und `webserver_service` für RedHat-Systeme.
Es gibt das Ansible-"Modul" `include_vars <Variablen-YAML>`. Wenn wir diesen Befehl in `tasks/main.yml` verwenden, werden die zu ersetzenden Variablen-Belegungen aus beispielsweise `vars/RedHat.yml` eingebunden.

### Tipp:
> Macht für jede Variable, die ihr in der ganzen Rolle verwenden wollt, einen Eintrag in `defaults/main.yml`. Dann habt ihr dort eine übersichtliche Liste aller in der Rolle verwendeten Variablen.

Zur Bestimmung des richtigen Namens für die Webserver-Package und den Webserver-Service werden die Variablen-YAMLs geschrieben, die distributionsabhängig verwendet werden sollen.

### `~/code/roles/Webserver/vars/RedHat.yml`
```yaml
---
# Variables for the Red Hat Family
webserver_service: httpd
webserver_package: httpd

```
### `~/code/roles/Webserver/vars/Debian.yml`
```yaml
---
# Variables for the Debian family

```
### `~/code/roles/Webserver/vars/Suse.yml`
```yaml
---
# Variables for the Suse family
webserver_package_state: latest

```

## Schritt 11: Einbinden Distributions-spezifischer Variablen-Belegungen
#### Listing Teil-Code zum Einbinden von zusätzlichen Variablen-YAMLs aus `~/code/roles/Webserver/vars`
```yaml
.
# Variablen werden distributionsspezifisch gesetzt
- include_vars: <Name vom Variablen.yml>
.
```
Um die Variablen Distributions-spezifisch zu setzen, kann ich distributionsspezische Variablen-Dateien einbinden. Der Trick ist, dass die Namen der Variablen-Dateien einem Ansible Fact entsprechen. Die entsprechenden Zeilen in `tasks/server.yml` müssen _vor_ den Tasks stehen, die die Variablen verwenden:
### Listing Teil-Code Einbinden von distributionsspezifichen Variablen-YAMLs aus `~/code/roles/Webserver/tasks/`
```yaml
# ...
### Variablen werden distributionsspezifisch gesetzt
include_vars: "{{ ansible_os_family }}.yml"
# ...
```

Also müssen wir unsere `tasks/server.yml` entsprechend modifizieren.

### `~/code/roles/Webserver/tasks/server.yml`

ACHTUNG: Linting-Fehler sind erwünscht!

```yaml=
--- 
# tasks file for Webserver
# Variablen werden distributionsspezifisch gesetzt
include_vars: "{{ ansible_os_family }}.yml"

- name: Make sure that the webserver package is installed
  ansible.builtin.package:
    name: "{{ webserver_package }}"
    state: "{{ webserver_package_state }}"
  # Falls die Webserver-Package neu installiert wurde, oder upgedatet wurde,
  # soll der Webserver neu gestartet werden
  notify: Restart Webserver
  
- name: Make sure webserver is running
  ansible.builtin.service:
    name: "{{ webserver_service }}"
    state: started
```

```yaml=
---
# tasks file for Webserver

- name: 'Include distribution-specific variables.'
  ansible.builtin.include_vars:
    file: "{{ ansible_os_family }}.yml"

- name: Make sure that the webserver package is installed
  ansible.builtin.package:
    name: "{{ webserver_package }}"
    state: "{{ webserver_package_state }}"
    # Falls die Webserver-Package neu installiert wurde, oder upgedatet wurde,
    # soll der Webserver neu gestartet werden
  notify: Restart Webserver

- name: Make sure webserver is running
  ansible.builtin.service:
    name: "{{ webserver_service }}"
    state: started
```
### Debugging

`ansible-playbook ~/code/playbooks/RoleWebserver.yml --check -v -l debian1`
`ansible-playbook ~/code/playbooks/RoleWebserver.yml --check -v -l redhat1`

## Schritt 12: Vorbereiten verschiedener Virtual Hosts für den Webservice
Ein Apache-Server kann mehrere Websites parallel bedienen. Diese heißen bei Apache "Virtual Hosts". Jeder Virtual Host hat einen kleinen Definitionstext mit Server-Namen, www.Adresse und DocumentRoot (dort, wo die HTML-, PHP-, CSS- und sonstige Dateien für die Website liegen).

### Listing: Virtual-Host-Konfigurationsdatei
```htmlmixed
<VirtualHost *:80>
ServerName bupfi
DocumentRoot /srv/www/bupfi
ServerAlias www.
        
<Directory /srv/www/bupfi>
  require all granted
</Directory>

</VirtualHost>
```

Wir legen, im ersten Schritt, die Daten für alle Virtual Hosts in einer einzigen Listen-Variablen (ein Listen-Eintrag pro Virtual Host) ab. Im zweiten Schritt nutzen wir diese Liste mit einem Jinja2-Template, welches dann die eigentlichen VHost-Konfigurationsdateien für jedes Zielsystem erzeugt.
Zunächst tragen wir die Listen-Variablen `apache_vhosts` für 3 Virtual Hosts in `vars/main.yml` ein. Jeder Listeneintrag ist ein Dictionary, welches die beiden erforderlichen Werte pro Virtual Host enthält.
 
### `~/code/roles/Webserver/vars/main.yml`
```yaml=
---
# vars file for Webserver
apache_vhosts:
  - name: websrv1
    docroot: /srv/www/websrv1
  - name: gogle.com
    docroot: /srv/www/gogle
  - name: hooogle.com
    docroot: /srv/www/neuecoolewebsite
```
## Schritt 13: Template-Datei erstellen
Die Platzhalter-Variable heißt schon `item`, da die 3 erforderlichen VHost-Konfigurationsdateien in einem `loop`  erzeugt werden.
### `~/code/roles/Webserver/templates/vhost.conf.j2`
```htmlmixed=
<VirtualHost *:80>
ServerName {{ item['name'] }}
DocumentRoot {{ item['docroot'] }}
ServerAlias www.{{ item['name'] }}
    
<Directory {{ item['docroot'] }}>
  require all granted
</Directory>

</VirtualHost>
```
## Schritt 14: Distributionsspezifisches Konfigurationsverzeichnis für die Virtual Hosts von Apache festlegen
Bevor wir das Templating durchführen können, müssen wir noch distributionsspezifische Speicherorte für die VHost-Dateien berücksichtigen. Dazu arbeiten wir mit der Variablen `apache_vhost_dir`, die wir über den bereits verwendeten Mechanismus (s. o.) distributionsspezifisch setzen.
Dafür können wir die schon vorhandenen Dateien unter `vars` ergänzen:

### `~/code/roles/Webserver/vars/RedHat.yml`
```yaml
---
# Variables for Red Hat Family
webserver_service: httpd
webserver_package: httpd
apache_vhost_dir: /etc/httpd/conf.d

```

### `~/code/roles/Webserver/vars/Debian.yml`
```yaml
---
# Variables for Debian family
apache_vhost_dir: /etc/apache2/sites-enabled

```

### `~/code/roles/Webserver/vars/Suse.yml`
```yaml
---
# Variables for Suse family
webserver_package_state: latest
apache_vhost_dir: /etc/apache2/vhosts.d

```
###  `~/code/roles/Webserver/templates/index.html.j2`
```=
Hello from {{ ansible_hostname }} via {{ item['name'] }}

```
## Schritt 16: Template-Datei mit dem `template`-Modul nutzen

### `~/code/roles/Webserver/tasks/vhosts.yml`
```yaml=
---
# Variablen werden distributionsspezifisch gesetzt
- name: 'Include distribution-specific variables.'
  ansible.builtin.include_vars:
    file: "{{ ansible_os_family }}.yml"

- name: Ensure existence of Docroot
  ansible.builtin.file:
    name: "{{ item['docroot'] }}"
    state: directory
    mode: "0755"
  loop: "{{ apache_vhosts }}"

- name: Make sure homepage is present
  ansible.builtin.template:
    src: index.html.j2
    dest: "{{ item['docroot'] }}/index.html"
    mode: "0644"
  loop: "{{ apache_vhosts }}"

- name: Have Apache VHost Configuration installed
  ansible.builtin.template:
    src: vhost.conf.j2
    dest: "{{ apache_vhost_dir }}/{{ item['name'] }}.conf"
    mode: "0644"
  loop: "{{ apache_vhosts }}"
  notify: Reload Webserver
  
```


### update the tasksMainFile
`~/code/roles/Webserver/tasks/main.yml`
```yaml=
# tasks file for Webserver

- name: 'Ensure the webserver is available'
  ansible.builtin.include_tasks:
    file: ./server.yml

- name: 'Ensure vhosts are created'
  ansible.builtin.include_tasks:
    file: ./vhosts.yml
    
```
## apply
```bash
### first single host
ansible-playbook ~/code/playbooks/RoleWebserver.yml -l suse1
### then all
ansible-playbook ~/code/playbooks/RoleWebserver.yml 
```
## check

### curl
#### everywhere
##### single
```
curl --connect-to websrv1:80:debian1:80 http://websrv1
```

##### all
```bash=
for HOST in {redhat,debian,suse,ubuntu}{1,2}
    do for SITE in {websrv1,hooogle.com,gogle.com}
        do curl --connect-to $SITE:80:$HOST:80 http://$SITE
    done
done

```


#### each on own host, serving the site

```
sudo vim /etc/hosts

127.0.0.1 websrv1
```
```
curl websrv1
```

### browser
on jumphost
```bash
### sudo vim /etc/hosts
100.80.1.225    debian1.user2.training.lab      debian1 hooogle.com gogle.com websrv1

sudo systemctl restart dnsmasq

firefox http://websrv1
```