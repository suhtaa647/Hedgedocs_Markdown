---
title: collections
tags: 10, 3days, 4days, 5days
---

# collections

## create a collection
### create a collection from the Webserver-Role (optional)

```bash
cd ~/code
ansible-galaxy collection init b1.ansible
```

### `git init`

in the collection not in the namespace
```bash
cd ~/code/b1/ansible
git init
```

### transfer role (optional)
put the role you have written into the folder `roles`
```bash
cp -avr ~/code/roles/Webserver ~/code/b1/ansible/roles ### optional
```

### playbooks
create the folder `playbooks`
```bash
mkdir ~/code/b1/ansible/playbooks
```

#### RoleWebserver (optional)
put the `RoleWebserver.yml` into `playbooks`
```bash
cp -avr ~/code/playbooks/RoleWebserver.yml ~/code/b1/ansible/playbooks ### optional
```


### add a `ansible.cfg`
`ansible.cfg` is used only as a single file, this file will only be usefull when developing the role
`~/code/b1/ansible/ansible.cfg`
``` shell
### ansible-config-file - only used for the development of this collection
[defaults]
roles_path = roles
library = plugins
filter_plugins = plugins
### Use the YAML callback plugin.
stdout_callback = yaml
### Use the stdout_callback when running ad-hoc commands.
bin_ansible_callbacks = True
```

### inventory
inventories are not used inside collections because they are not reusable

## reusing
[Installing collections — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/collections_guide/collections_installing.html#installing-a-collection-from-source-files)
### create project
```bash
cd /home/qskills/code
ansible-galaxy collection init b1.project
cd ./b1/project
```
### reuse the previous collection
`touch ~/code/b1/project/requirements.yml`
#### XOR
##### use local directory
```yaml=
  # requirements.yml
  collections:
    - name: b1.ansible
      version: 1.0.0
      source: ../../b1/ansible
      type: dir
```
##### use repository
1. create a repository on the local (inner) gitea
2. push the collection to the local (inner) gitea

`git remote add origin git@git.training.lab:b1/ansible.git`

```yaml=
  # requirements.yml
  collections:
    - name: http://git.training.lab/b1/ansible.git
      version: main
      type: git

```
##### use galaxy
our collection will not be published to the galaxy - but this is the default way consuming collections.
```yaml=
  # requirements.yml
  collections:
    - name: b1.ansible
      version: 1.0.0
      type: galaxy
```

### check installed collections
`ls ~/.ansible/collections/ansible_collections/`

### install collection
`ansible-galaxy install -r requirements.yml`

### check installed collections again
`ls ~/.ansible/collections/ansible_collections/`
`exa -T ~/.ansible/collections/ansible_collections/`

### playbook
[Re-using Ansible artifacts — Ansible Community Documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse.html)

`mkdir ~/code/b1/project/playbooks`

#### `~/code/b1/project/playbooks/CollectionTest.yml`

```yaml=    
---
- name: Local play
  hosts: all
  vars:
    messages: "From local play"
  tasks:
    - name: Debug-Message
      ansible.builtin.debug:
        msg: "{{ messages }}"
      tags: local
    ### not available yet:
    # - name: Debug-Message with custom filter
    #   ansible.builtin.debug:
    #     msg: "{{ messages | b1.ansible.uppercase_input }}"
    #   tags: [local,custom]
    # - name: Use Custom module
    #   b1.ansible.hello_world:
    #     content: "{{ messages }}"
    #   tags: [local,custom]
- name: Reuse playbook from other collection
  ansible.builtin.import_playbook: b1.ansible.RoleWebserver
  tags: imported

```

##### usage
```bash

ansible-playbook ~/code/b1/project/playbooks/CollectionTest.yml -i ~/code/inventory -l debian1 -t local
ansible-playbook ~/code/b1/project/playbooks/CollectionTest.yml -i ~/code/inventory -l debian1 -t imported


### after further development and deployment of the role
ansible-playbook ~/code/b1/project/playbooks/CollectionTest.yml -i ~/code/inventory -l debian1 -t 'local' --skip-tags custom
ansible-playbook ~/code/b1/project/playbooks/CollectionTest.yml -i ~/code/inventory -l debian1 -t local

```