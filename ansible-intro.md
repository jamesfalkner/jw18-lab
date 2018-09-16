## Ansible Intro

Ansible is a radically simple IT automation engine that automates cloud provisioning, configuration management,
application deployment, intra-service orchestration, and many other IT needs.

In this first step you will...

## Access Terminal

You will run a number of Ansible commands using a web-based terminal on your lab environment.

[Access the terminal]({{ TERMINAL_URL }}){:target="_blank"} and get started!

## Log-in to the Terminal

Username: cloud-user
Password: r3dh4t1!

## Switch to the correct user (root)

```
sudo -s -H
```

## Change to the Working Directory

```
cd $JW18_ANSIBLE
```

## Run a Playbook that the Lab Needs

```
ansible-playbook init_dev_template.yml
```

And the final output should be
```
PLAY RECAP *********************************************************************************  
localhost                  : ok=18   changed=6    unreachable=0    failed=0  
```

## Install a Text Editor

This is an ad-hoc Ansible command than runs against localhost (or any host you want) which uses the yum module to install the latest version of the nano package.

```
ansible localhost -m yum -a "name=nano state=latest"
```

This could be done as a playbook:
```
---
- hosts: localhost
  tasks:
  - yum:
      name: nano
      state: latest
```

## Install Prerequsite Roles from Galaxy

The playbook needs a few roles to run successfully. First let's find the full name by searching Ansible Galaxy.

```
ansible-galaxy search openshift_common_facts
```

Then install the role with the fully qualified name.

```
ansible-galaxy install siamaksade.openshift_common_facts
```

## Now Let's Install Something

Create a new playbook using our text editor

```
nano my-playbook.yml
```

With the contents
```
---
- name: Install nginx container
  hosts: localhost
  tasks:
  - command: oc new-project nginx-lab
  - command: oc new-app nginx
  - command: oc get routes
    register: routes
  - debug: var=routes
```

## Run your playbook

```
ansible-playbook my-playbook.yml
```

And the final output there should be a HOST/PORT like `https://master-3e6a.rhpds.opentlc.com`
```
PLAY RECAP *********************************************************************************  
localhost                  : ok=18   changed=6    unreachable=0    failed=0  
```

## Voila!

Well done! You are now ready to proceed to the next lab and put your developer hat on and write some code!

