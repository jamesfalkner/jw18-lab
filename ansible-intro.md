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
quickly and easily from your playbooks.

### Run a Playbook that the Lab Needs

Later on in this lab, when you're working as a developer you'll create and deploy microservices
to OpenShift using a template within OpenShift. Let's deploy that template now with Ansible. The
playbook will use our previously installed dependency `openshift_common_facts`.

~~~sh
ansible-playbook init_dev_template.yml
~~~

And the final output should be

~~~
PLAY RECAP *********************************************************************************  
localhost                  : ok=18   changed=6    unreachable=0    failed=0  
~~~

Inspect the contents of `init_dev_template.yml` and notice the use of `openshift_common_facts`:

~~~yaml
  tasks:
    - include_role:
        name: openshift_common_facts
      tags: always
~~~

{% raw  %}
This role sets several variables related to the OpenShift cluster you have been provided, namely
the `openshift_cli` fact which can be referenced as `{{ openshift_cli }}` in the playbook.
{% endraw %}

### Install a Text Editor

We need a simple text editor to write or edit playbooks. Let's install one using an _ad-hoc_ Ansible command.

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

### Deploy HTTPD to OpenShift using Ansible

There is a sample playbook in on your machine named `init_httpd.yml`. It's a relatively simple
playbook which installs Apache HTTPD into OpenShift and waits for it to be up and running. Inspect
the playbook:

~~~sh
cat init_httpd.yml
~~~

This Playbook also demonstrates (albeit rather naievely) the characteristic of _idempotency_ - a core tenant of repeatable automated infrastructure.
Idempotent tasks can be run multiple times and will achieve the same effect without side-effect. Here we can run our playbook as many
times as we want, as fast as we want, and the end result will always be the same (HTTPD will be installed).

### Run the HTTPD playbook

Run the playbook:

~~~sh
ansible-playbook init_httpd.yml
~~~


This playbook uses the `command` and a few other modules and tasks to deploy Apache HTTPD (a web server) to your lab machine.
The same kinds of commands are used later on in this lab.

Did it work? Probably not. There is a bug in the playbook! The problem is that out of the box,
the Apache HTTPD server has a test page that can be accessed to make sure the server is working,
however this test page returns an HTTP `403 Forbidden` error by design, not a `200 OK`. Our httpd
server is working, but the playbook isn't right. Let's fix it with our handy nano text editor:

~~~sh
nano init_httpd.yml
~~~

In the editor, use the arrow keys to navigate to the bottom of the Playbook where you see two places
that need to change from `200` to `403`. Make the changes by backspacing over the `200`'s and replace
with `403` so that the end looks like:

~~~yaml
  - name: wait for httpd to be running
    uri:
      url: "http://httpd-webserver.{{ apps_hostname_suffix }}"
      status_code: 403
    register: result
    until: result.status == 403
    retries: 10
    delay: 2
~~~

Press `CTRL-X` and save the file when prompted. You should end up back at the Bash shell prompt.

Re-run the playbook (it's idempotent, right?) to test our fix:

~~~sh
ansible-playbook init_httpd.yml
~~~

You may see a few things that look like errors or failures as Ansible retries a few times, but eventually
you'll see success:

~~~
PLAY RECAP *******************************************************************************************************************************************
localhost                  : ok=19   changed=8    unreachable=0    failed=0
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

