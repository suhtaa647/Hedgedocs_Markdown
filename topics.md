---
title: topics
tags: 15, 3days-optional, 4days-optional, 5days
---

# topics

## INFO
[geerlingguy (Jeff Geerling)](https://github.com/geerlingguy)
[Jeff Geerling - YouTube](https://www.youtube.com/c/JeffGeerling)

[ansible-community/awesome-ansible: Awesome Ansible List](https://github.com/ansible-community/awesome-ansible)

## airgapped
Ansible Collections sind Zusammenstellungen von Ansible Modulen, die von Ansible selbst oder der Community bereitgestellt werden.

### Nutzung von Ansible Collections ohne Internet-Zugang
https://www.redhat.com/sysadmin/install-ansible-disconnected-node

[Ansible CLI cheatsheet — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/command_guide/cheatsheet.html)


## special chars - local
[Learn yaml in Y Minutes](https://learnxinyminutes.com/docs/yaml/)
[YAML Multiline Strings](https://yaml-multiline.info/)
[community.general.locale_gen module – Creates or removes locales — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/collections/community/general/locale_gen_module.html)

---

## Tagging
Tags dienen zum direkten Ansprechen einer oder einer Gruppe von Tasks.
Dabei kann ein Benamungsschema für die Tags nützlich sein. Hier im Beispiel fangen alle _eigenen_ Tag-Namen mit "`tag_`" an.

### Listing `Playbook_mit_Tags.yml`:
```yaml=
---
- name: a playbook with tags
  hosts: all
  tasks:
    - name: a task for ubuntu1-0
      ansible.builtin.debug:
        msg: "{{ ansible_hostname }}"
      when: ansible_hostname == "ubuntu1-0"
      tags: tag_ubuntu1-0
      
    - name: a task for suse1-0
      ansible.builtin.debug:
        msg: "{{ ansible_hostname }}"
      when: ansible_hostname == "suse1-0"
      tags: tag_suse1-0
      
    - name: a task for redhat2-0
      ansible.builtin.debug:
        msg: "{{ ansible_hostname }}"
      when: ansible_hostname == "redhat2-0"
      tags: tag_redhat2-0
      
    - name: a task for all hosts
      ansible.builtin.debug:
        msg: "{{ ansible_hostname }}"
      tags: ['never', 'allofthem']

```
### Aufrufe zum Testen
1. `ansible-playbook -i code/inventory code/TagsUsage.yml --list-tasks`
2. `ansible-playbook -i code/inventory code/TagsUsage.yml --list-tags`
3. `ansible-playbook -i code/inventory code/TagsUsage.yml`
4. `ansible-playbook -i code/inventory code/TagsUsage.yml --tags "never"`
5. `ansible-playbook -i code/inventory code/TagsUsage.yml --list-tasks --tags "ubuntu1-0,never"`
6. `ansible-playbook -i code/inventory code/TagsUsage.yml --tags "never"`

---

## Ansible Vault
Ansible Vault bietet die Möglichkeit Ansible-Code ganz oder teilweise (!) zu verschlüsseln.
### Beispiel: Schritte zum Verschlüsseln und Entschlüsseln eines Playbooks
1. Kopieren eines Playbooks zum Testen: 
   `cp -av /home/tux/code/playbooks/RoleWebserver.yml /home/tux/code/playbooks/Vault-Test.yml`
3. Verschlüsseln: 
   `ansible-vault encrypt /home/tux/code/playbooks/Vault-Test.yml`
5. Aufruf des verschlüsselten Playbooks mit `--ask-vault-pass` zum interaktiven Entschlüsseln:
   `ansible-playbook -i /home/tux/code/inventory --ask-vault-pass /home/tux/code/playbooks/Vault-Test.yml --check`
7. Entschlüsseln: 
   `ansible-vault decrypt /home/tux/code/playbooks/Vault-Test.yml`

### Beispiel: Schritte zum Verschlüsseln einer einzelnen Zeichenkette zum Einsetzen in eine YAML-Datei
1. Verschlüsseln einer Variable: 
   `ansible-vault encrypt_string 'An encrypted string' --name 'vault_variable'`
3. Einsetzen in eine YAML-Datei `Playbook_mit_verschlüsselter_Zeichenkette.yml`
   Beispielsweise könnten wir die erzeugte Zeichenkette aus 1. so an das Playbook anhängen 
`ansible-vault encrypt_string 'An encrypted string' --name 'vault_variable' >> /home/tux/code/playbooks/Playbook_mit_verschlüsselter_Zeichenkette.yml` 
   und diese Zeile anschließend im Editor innerhalb des Playbooks an die richtige Stelle schieben

#### Listing `Playbook_mit_verschlüsselter_Zeichenkette.yml`:
```yaml=
---
- name: 'My playbook with an encrypted variable'
  hosts: suse2
  ### password: password ### NEVER DO THIS IN REAL
  vars:
    vault_variable: !vault |
              $ANSIBLE_VAULT;1.1;AES256
              35636638306535373436353062313130373036326134346431383132393537303261633737333264
              6430356630363430323130633535393939636264633165370a323265393733656433383564343239
              30316630626637373335333337373861646539353665393132383639616366396534323861356138
              6532323663663437370a383263643432643466353261333131613033326262653237373930376163
              63376239383231616436316664373036653239393936333662366139333365316633
  tasks:
    - name: 'Print encrypted value'
      ansible.builtin.debug:
        msg: "{{ vault_variable }}"
```
3. Aufruf des Playbooks: 
  `ansible-playbook --ask-vault-pass /home/tux/code/playbooks/Playbook_mit_verschlüsselter_Zeichenkette.yml`

### Alternativen

[cyberark/summon: CLI that provides on-demand secrets access for common DevOps tools](https://github.com/cyberark/summon)

[Community.Hashi_Vault — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/collections/community/hashi_vault/index.html)

[Community.Sops — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/collections/community/sops/index.html)

[Ansible | CyberArk Docs](https://docs.cyberark.com/conjur-enterprise/13.0/en/Content/Integrations/ansible.html)

### lookups for credentials
```shell
ansible-doc -t lookup
ansible-doc -t lookup -l
```
[christian-heusel/ansible-lookup-plugin-gopass: Ansible lookup plugin for gopass ](https://github.com/christian-heusel/ansible-lookup-plugin-gopass)

---

## Jinja2, cont.

## Verschachtelte Schleifen
Der Jinja2-Filter `{{ <liste> | product(<liste> }}` bildet ein Kartesisches Produkt ("Jeder mit jedem").
Das Ergebnis des Jinja2-Filter Ausdrucks `{{ ['user1', 'user2']|product(['.vimrc', '.bashrc'])|list }}` ist eine Liste:
```
- ['user1', '.vimrc']
- ['user1', '.bashrc']
- ['user2', '.vimrc']
- ['user2,' '.bashrc']
```
Diese Liste wird im Beispiel der Folie 163 an `loop` verfüttert. Innerhalb des ersten Schleifendurchlaufs wird die Schleifenvariable `item` zu einer Listen-Variable mit dem Inhalt ['user1', '.vimrc'], referenziert ergibt das dann folgende Einsetzungen: `item[0]` wird zu `user1`, `item[1]` wird zu `.vimrc`. Somit kann jeder User aus der Userliste jede Datei aus der Dateiliste kopiert bekommen.
