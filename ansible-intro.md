## Ansible Intro

Ansible is a radically simple IT automation engine that automates cloud provisioning, configuration management,
application deployment, intra-service orchestration, and many other IT needs.

In this step you will run a few Ansible commands on your lab machine to learn how to automatically
deploy software to OpenShift using easy-to-create Ansible Playbooks.

### Access Terminal

We'll use the Ansible CLI (Command Line Interface) to perform basic Ansible tasks with a Linux shell. To run these
commands, access your lab machine using a web-based terminal on your lab environment.

|**NOTE**: The first time you access the terminal, you will get a “Your connection is not private” or other security warning in your browser due to using self-signed certificates. Please click on Advanced and then Proceeed to accept the untrusted certificate (other browsers may have a slightly different process to accept this warning).

[Access the terminal]({{ TERMINAL_URL }}){:target="_blank"} and get started!

### Log-in to the Terminal

When you access the web terminal, you'll need to login. Use these credentials:

* Username: `cloud-user`
* Password: `r3dh4t1!`

![Terminal]({% image_path ansible-terminal.png %}){:width="900px"}

### Switch to the correct user (`root`)

Ansible can run with many different privilege polices/setups. For this lab we will simply run
as the `root` user. See the [Ansible Docs](https://docs.ansible.com/ansible/latest){:target="_blank"} for more information
on Ansible and privileges.

Run the `sudo` command to become the `root` user:

~~~sh
sudo -s -H
~~~

### Change to the Working Directory

This directory contains some files used by the lab (most notably the lab infrastructure itself has
been deployed using Ansible, and files within this directory are a good example of an Ansible
Playbook for automating deployments to multiple machines!).

Switch to the right directory:

~~~sh
cd $JW18_ANSIBLE
~~~

### Run a Playbook that the Lab Needs

Later on in this lab, when you're working as a developer you'll create and deploy microservices
to OpenShift using a template. Let's deploy that template now with Ansible:

~~~sh
ansible-playbook init_dev_template.yml
~~~

And the final output should be

~~~
PLAY RECAP *********************************************************************************  
localhost                  : ok=18   changed=6    unreachable=0    failed=0  
~~~

### Install a Text Editor

This is an [ad-hoc](https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html){:target="_blank"} Ansible command than runs against localhost (or any host you want) which uses the _yum_ module to install the latest version of the _nano_ package (nano is a basic text editor, like `vi` but for ordinary humans).

~~~sh
ansible localhost -m yum -a "name=nano state=latest"
~~~

This could also be done as a playbook:

~~~yaml
---
- hosts: localhost
  tasks:
  - yum:
      name: nano
      state: latest
~~~

### Install Prerequisite Roles from Galaxy

[Ansilble Galaxy](https://galaxy.ansible.com/){:target="_blank"} has thousands of open source pre-created roles you can use in your projects (very much like
Maven Central, NPM Registry, Docker Hub, etc). Let's find some helpful roles related to OpenShift
that we can use later on. These roles do things like query OpenShift for its configuration and then
set _facts_. Facts are bits of information derived from examining host systems that are stored as variables for later use in a play.

First let's find the full name of our desired role by searching Ansible Galaxy.

~~~sh
ansible-galaxy search openshift_common_facts
~~~

The `search` command returns the list of roles that match the query term.

Then install the role with the fully qualified name.

~~~sh
ansible-galaxy install siamaksade.openshift_common_facts
~~~

This role will be used later on to inspect the OpenShift cluster you're using and discover facts
about it (such as login names, URLs, configuration, etc) and can be used to deploy things to OpenShift
quickly and easily.

### Create a Playbook to deploy an app to OpenShift

Create a new playbook using our _nano_ text editor.

~~~sh
nano my-playbook.yml
~~~

|**NOTE**: Note that Playbooks are written in [YAML](https://en.wikipedia.org/wiki/YAML){:target="_blank"}, where indenting is significant. So be sure to maintain the same indenting for each line while copy/pasting!

Copy/paste this content into the file:

~~~yaml
---
- name: Install Apache HTTPD container
  hosts: localhost
  tasks:
  - command: oc new-project webserver
  - command: oc new-app httpd
  - command: oc expose svc/httpd
  - command: oc get route/httpd -n webserver -o jsonpath='{.spec.host}'
    register: route
  - debug: var=route
~~~

This playbook uses the `command`, `register` and `debug` tasks to deploy Apache HTTPD (a web server) to your lab machine.
The same kinds of commands are used later on in this lab.

Press `CTRL-X` to exit, answering `Y` to save the file with the same filename as above and return to the Bash shell prompt.

### Run your playbook

~~~sh
ansible-playbook my-playbook.yml
~~~

One of the `command` tasks used the `register` task to save the output from a command (`oc get route/httpd`)
as a variable named `route` that can be referenced in the playbook. We output this using the `debug` task on the
last line of the playbook. This will show you a URL you can use to access the app once it's deployed:

~~~
TASK [debug] *****************************************************************************************************************************************
ok: [localhost] => {
    "route": {
        ...
        "stdout_lines": [
            "httpd-webserver.apps-GUID.generic.opentlc.com"
        ]
    }
}
~~~

### Test the app

Now that you've deployed something to OpenShift with Ansible, you should be able to access it
through its route URL. [Try the route URL in your browser](http://httpd-webserver{{ PROJECT_SUFFIX }}.{{ APPS_HOSTNAME_SUFFIX }}){:target="_blank"}.
You should get the HTTPD "It works!" screen:

![HTTPD Works]({% image_path ansible-webserver-test.png %}){:width="900px"}

### Voila!

Well done! You've deployed the remaining lab infrastructure using Ansible and repeatable Playbooks.
These playbooks can be stored in your configuration management database (typically a SCM like Git,
Subversion, or others.)

You are now ready to proceed and put your developer hat on and write some code with the lab you
just deployed!

