---
title: AWX
tags: 12, 4days, 5days
---

# AWX

https://awx.training.lab/

runs on infra in a MicroK8s-Cluster - requires k8s because of operator

users are already created - known credentials

upstream for the **Ansible Automation Controller** (former **Tower**) part of the **Ansible Automation Platform**

## WORKFLOW-OVERVIEW

### prepare inventory for use on AWX

### create repositories

####  collection

####  inventory

####  EE

###  add Organisation

###  add Credentials

####  vcs

####  inventory

###  add Project

####  collection

####  inventory

###  add Inventory

###  generate Template

###  run Job

###  fail

###  generate EE

####  enable registry

####  install docker

####  prepare venv

####  build EE

####  push EE

###  create Execution Environment

### use Execution Environment in Template

### run Job

### check result


## prepare the inventory
`~/code/inventory`
```ini
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
# Variable für alle Rechner der Gruppe Suse
# Die Definition der Variable ansible_python_interpreter lässt die Warnmeldung
# bei Suse-Systemen verschwinden mit folgender Einstellung
[Suse:vars]
ansible_python_interpreter=/usr/bin/python3.12


### AWX - will be executed on an host with no search domain set - 
### use your own userNumber!
[all:vars]
ansible_python_interpreter=/usr/bin/python3
host_domain=user0.training.lab
ansible_host="{{inventory_hostname}}.{{host_domain}}"
```

## DEPENDENCIES

1.  needs code for the `b1.ansible` from gitea

2.  needs code for the `inventory` from gitea

## WORKFLOW

1.  check `/etc/hosts`

2.  GITEA

    1.  create repo on inner `gitea` (outer works, too)

        via jumphost http://git.training.lab

        1.  create organisation

            b1

        2.  create repository

            ansible

        3.  setup sshKey

            via user http://git.training.lab/user/settings/keys

            ```shell
            cat ~/.ssh/id_ed25519.pub

            ```

    2.  push code to repo via ssh

        ```shell
        git checkout -b main
        git remote add origin git@git.training.lab:b1/ansible.git
        git push -u origin main
        ```

3.  AWX

    1.  credentials

        user0 with the provided password
        `kubectl get secret awx-admin-password -ojsonpath='{.data.password}' -n awx | base64 -d`

    2.  create organisation

        http://awx.training.lab/#/organizations
        
        b1s

    3.  create user

        http://awx.training.lab/#/users 
        
        already created

    4.  create credentials

        http://awx.training.lab/#/credentials

        1.  key for gitea

            can be a newly generated with access to the git repo 
            - Name: gitea
            - Credential Type: Source Control
            - key:

            ```shell
            cat ~/.ssh/id_ed25519
            ```

        2.  key for machines

            ansible deploy to machines like qskills on the controler
            - Name: inventory
            - Credential Type: Machine
            - username: qskills
            - key:

            ```shell
            cat /home/qskills/.ssh/id_ed25519
            ```

    5.  create project

        http://awx.training.lab/#/projects
        
        - Name: ansible
        - use ssh-link as copied from gitea
        - use SSH-key for gitea
        - Source Control Type: Git
        - Source Control URL: git@git.training.lab:b1/ansible.git XOR
        - Source Control URL: https://94360.training.b1-systems.de/git/b1/ansible.git
        - Source Control Credential: gitea

    6.  jobs

        watch syncing or restart Sync has to be successful

    7.  create project for inventory

        create a repository on gitea containing an inventory
        - Name: inventory
        - Source Control Type: Git
        - Source Control URL: https://94360.training.b1-systems.de/git/training/user0.git
        - Source Control Credential: gitea

    8.  inventory

        - Name: inventory
        - Organisation: b1s
        - Variables: none
        - create the inventory

        1.  edit the inventory
            - Tab Sources:
            - XOR
            1. from sources
                - Name: inventory
                - Source: Sourced from a project
                - Project: inventory
                - Inventory file: inventory

            2.  create solitary repository

                ```shell
                ansible-inventory -i ~/code/inventory --list --yaml
                ```
                
                ```yaml
                all:
                 children:
                   Debian:
                     hosts:
                       debian1:
                         ansible_host: '{{inventory_hostname}}.{{host_domain}}'
                         ansible_python_interpreter: /usr/bin/python3
                         host_domain: user0.training.lab
                       debian2:
                         ansible_host: '{{inventory_hostname}}.{{host_domain}}'
                         ansible_python_interpreter: /usr/bin/python3
                         host_domain: user0.training.lab
                   Productive:
                     children:
                       Deployment:
                         hosts:
                           redhat1: {}
                           suse2: {}
                       Webserver:
                         hosts:
                           suse1: {}
                           ubuntu1: {}
                   RedHat:
                     hosts:
                       redhat1:
                         ansible_host: '{{inventory_hostname}}.{{host_domain}}'
                         ansible_python_interpreter: /usr/bin/python3
                         host_domain: user0.training.lab
                       redhat2:
                         ansible_host: '{{inventory_hostname}}.{{host_domain}}'
                         ansible_python_interpreter: /usr/bin/python3
                         host_domain: user0.training.lab
                   Suse:
                     hosts:
                       suse1:
                         ansible_host: '{{inventory_hostname}}.{{host_domain}}'
                         ansible_python_interpreter: /usr/bin/python3.12
                         host_domain: user0.training.lab
                       suse2:
                         ansible_host: '{{inventory_hostname}}.{{host_domain}}'
                         ansible_python_interpreter: /usr/bin/python3.12
                         host_domain: user0.training.lab
                   Ubuntu:
                     hosts:
                       ubuntu1:
                         ansible_host: '{{inventory_hostname}}.{{host_domain}}'
                         ansible_python_interpreter: /usr/bin/python3
                         host_domain: user0.training.lab
                       ubuntu2:
                         ansible_host: '{{inventory_hostname}}.{{host_domain}}'
                         ansible_python_interpreter: /usr/bin/python3
                         host_domain: user0.training.lab

                ```
    9.  template

        http://awx.training.lab/#/templates/job_template/add

        1.  new job template
            - Name: Webserver
            - Job Type: Run
            - Inventory: inventory
            - Project: ansible
            - Credentials: inventory
            - Playbook: playbooks/RoleWebserver.yml
            - `Save` `Launch`
## INFO
### debugging
increase job\'s verbosity

### changes on inventory have to be synced manually

## PROBLEM: zypper missing

### MESSAGE

``` example
TASK [Webserver : Make sure that the webserver package is installed] ***********
task path: /runner/project/roles/Webserver/tasks/server.yml:7
redirecting (type: modules) ansible.builtin.zypper to community.general.zypper
The full traceback is:
NoneType: None
fatal: [suse2]: FAILED! => {
    "changed": false,
    "msg": "Could not find a module for zypper."
}
```


### job

will launch a pod in the awx namespace

```shell
ssh infra
sudo su -
kubectl config set-context --current --namespace awx
alias k=kubectl
k logs automation-job-NN-AAAAA
k get pod -w
k exec -it POD -- bash
ansible-galaxy collection list
```

## build EE with community.general installed

adding a requirements.yml will not work


###  generate EE

####  enable registry

on infra as root
```shell
microk8s enable registry
```

####  install docker
on ansible-N as root
```shell
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo docker run hello-world
sudo usermod -aG docker $USER
newgrp docker
```
#### enable registry
```shell
sudo su -
cat >> /etc/docker/daemon.json  <<'EOF'
{
  "insecure-registries" : ["infra:32000"]
}
EOF
systemctl reload docker
```

#### prepare venv
copy awx-EE-master.tar.gz to ansible-0
```shell
tar -xzf awx-EE-master.tar.gz
cd awx-ee
python -m venv venv
source venv/bin/activate
python3 -m pip install -r requirements.txt

```

#### build EE
```shell
ansible-builder build --tag=infra:32000/custom-ee:1.0.0 --container-runtime=docker --verbosity=3
````

#### push EE
```shell
docker push infra:32000/custom-ee:1.0.0
```
#### check EE
```shell
curl infra:32000/v2/_catalog
sudo apt install -y skopeo
skopeo --tls-verify=false  list-tags docker://infra:32000/custom-ee
```
### AWX: create Execution Environment
https://awx.training.lab/#/execution_environments
Name: custom-ee
Image: localhost:32000/custom-ee:1.0.0


### use Execution Environment in Template
https://awx.training.lab/#/templates
on Webserver
Execution Environment: custom-ee

### run Job
Launch template
### check result