## Ansible Intro

Ansible is a radically simple IT automation engine that automates cloud provisioning, configuration management,
application deployment, intra-service orchestration, and many other IT needs.

In this step you will run a few Ansible commands on your lab machine to learn how to automatically
deploy software to OpenShift using easy-to-create Ansible Playbooks.

### Access Terminal

We'll use the Ansible CLI (Command Line Interface) to perform basic Ansible tasks with a Linux shell. To run these
commands, access your lab machine using a web-based terminal on your lab environment.

[Access the terminal]({{ TERMINAL_URL }}){:target="_blank"} and get started!

### Log-in to the Terminal

When you access the web terminal, you'll need to login. Use these credentials:

* Username: `cloud-user`
* Password: `r3dh4t1!`

### Switch to the correct user (`root`)

Ansible can run with many different privilege polices/setups. For this lab we will simply run
as the `root` user. See the [Ansible Docs](https://docs.ansible.com/ansible/latest) for more information
on Ansible and privileges.

Run the `sudo` command to become the `root` user:

```bash
sudo -s -H
```

### Change to the Working Directory

This directory contains some files used by the lab (most notably the lab infrastructure itself has
been deployed using Ansible, and files within this directory are a good example of an Ansible
Playbook for automating deployments to multiple machines!).

Switch to the right directory:

```sh
cd $JW18_ANSIBLE
```

### Run a Playbook that the Lab Needs

Later on in this lab, when you're working as a developer you'll create and deploy microservices
to OpenShift using a template. Let's deploy that template now with Ansible:

```sh
ansible-playbook init_dev_template.yml
```

And the final output should be
```console
PLAY RECAP *********************************************************************************  
localhost                  : ok=18   changed=6    unreachable=0    failed=0  
```

### Install a Text Editor

This is an [ad-hoc](https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html) Ansible command than runs against localhost (or any host you want) which uses the _yum_ module to install the latest version of the _nano_ package (nano is a basic text editor, like `vi` but for ordinary humans).

```
ansible localhost -m yum -a "name=nano state=latest"
```

This could also be done as a playbook:

```yaml
---
- hosts: localhost
  tasks:
  - yum:
      name: nano
      state: latest
```

### Install Prerequsite Roles from Galaxy

Ansilble Galaxy has thousands of open source pre-created roles you can use in your projects (very much like
Maven Central, NPM Registry, Docker Hub, etc). Let's find some helpful roles related to OpenShift
that we can us elater on. These roles do things like query OpenShift for its configuration and then
set _facts_ that playbooks can use programmatically.

First let's find the full name by searching Ansible Galaxy.

```bash
ansible-galaxy search openshift_common_facts
```

The `search` command returns the list of roles that match the query term.

Then install the role with the fully qualified name.

```bash
ansible-galaxy install siamaksade.openshift_common_facts
```

### Create a Playbook to deploy an app to OpenShift

Create a new playbook using our _nano_ text editor:

```bash
nano my-playbook.yml
```

With the contents
```yaml
---
- name: Install nginx container
  hosts: localhost
  tasks:
  - command: oc new-project webserver
  - command: oc new-app httpd
  - command: oc expose svc/httpd
  - command: oc get route/httpd -n webserver -o jsonpath='{.spec.host}'
    register: route
  - debug: var=route
```

This playbook uses the `command`, `register` and `debug` tasks to deploy Apache HTTPD (a web server) to your lab machine.
The same kinds of commands are used later on in this lab.

### Run your playbook

```bash
ansible-playbook my-playbook.yml
```

One of the `command` tasks used the `register` task to save the output from a command (`oc get route/httpd`)
as a variable named `route` that can be referenced in the playbook. We output this using the `debug` task on the
last line of the playbook. This will show you a URL you can use to access the app once it's deployed:

```console
TASK [debug] *****************************************************************************************************************************************
ok: [localhost] => {
    "route": {
        ...
        "stdout_lines": [
            "httpd-webserver.apps-GUID.generic.opentlc.com"
        ]
    }
}
```

### Test the app

Now that you've deployed something to OpenShift with Ansible, you should be able to access it
through the route displayed above. Open a new tab in your browser and copy/paste the URL. It
should be something like `http://httpd-webserver.apps-XXXX.generic.opentlc.com`.

You should get the HTTPD "It works!" screen:

![HTTPD Works]({% image_path ansible-webserver-test.png %}){:width="900px"}

### Voila!

Well done! You've deployed the lab infrastructure using Ansible and repeatable Playbooks.
You are now ready to proceed and put your developer hat on and write some code with the lab you
just deployed!

