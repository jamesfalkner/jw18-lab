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

## Checkout the Latest Copy

```
git clone https://github.com/jamesfalkner/jw18-lab.git
```

## Change to the Working Directory

```
cd jw18-lab/ansible
```

## View the Playbook

```
more init_dev_template.yml
```

## Install Prerequsite Roles from Galaxy

```
ansible-galaxy install -r requirements.yml
```

## Run the playbook (as root)

```
sudo ansible-playbook init_dev_template.yml
```

And the final output should be
```
PLAY RECAP *********************************************************************************
localhost                  : ok=18   changed=6    unreachable=0    failed=0 
```

## Voila!

Well done! You are now ready to proceed to the next lab and put your developer hat on and write some code!

## Additional Ansible Information

Would you like to learn Ansible?  
It’s easy to get started: [ansible.com/get-started](http://ansible.com/get-started)

Want to learn more?
Videos, webinars, case studies, whitepapers: [ansible.com/resources](http://ansible.com/resources)

