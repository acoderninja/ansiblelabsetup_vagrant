# Overview

This vagrant based and ansible assisted automation will help us to setup a lab to build a tier3 infrastructure and learn the ansible usage.

The lab will implement the following configuration when completed:

----------------|---------------|----------------------------------------|----------------------------|
| Machine Name  | Role          | Network Configuration  Details         | Linux OS                   |
|---------------|---------------|----------------------------------------|----------------------------|
| control       | Ansible Host  | private_network, ip: 192.168.135.101   | Ubuntu bionic64 (18 LTS)   |
| lb01          | Load Balancer | private_network, ip: 192.168.135.102   | Ubuntu bionic64 (18 LTS)   |
| app01         | Web server 1  | private_network, ip: 192.168.135.103   | Ubuntu bionic64 (18 LTS)   |
| app02         | Web server 2  | private_network, ip: 192.168.135.104   | Ubuntu bionic64 (18 LTS)   |
| db01          | MySql DB      | private_network, ip: 192.168.135.105   | Ubuntu bionic64 (18 LTS)   |
----------------|---------------|----------------------------------------|----------------------------|
# Quick Start

* Clone this repo

* Install the hostmanager Vagrant plugin if ytou haven't already `vagrant plugin install vagrant-hostmanager`
  * on a Mac Vagrant host, `brew install libffi` was required prior to installing the plugin successfully

* run `vagrant up` from the root of the synced repo (the folder with Vagrantfile in it)
* Once the VMs are built, type `vagrant ssh control` to logon to the ansible controller from within your vagrant project folder

* Change directories `cd /vagrant/ansible` which is the ansible subfolder of your vagrant project for this lab (the vagrant project folder is mounted within the VMs as /vagrant during provisioning)

* run `ansible-playbook playbooks/site.yml` to build the entire 3 tier configuration (load balancer, sample web app, and database connection)

* run `ansible-playbook playbooks/stack_status.yml` to validate status of each component

* test that the application stack is working properly end to end by running `curl http://lb01/db`


#Exploring the details
Inventory
* ./hosts
  * file defining the servers to be orchestrated
Ansible Configuration
* ./ansible.cfg
  * defines hosts file location and ansible vault password file location
Group vars
* ./group_vars/all/vars
  * global variables file, includes database name and login user info
* ./group_vars/all/vault
  * file encrypted containing database user password (vault_db_pass) 
  * you can view and edit the file with `ansible-vault edit group_vars/all/vault` - your decryption key specified in ansible.cfg will be used to decrypt and reencrypt the file transparently

Ansible Playbooks:

* ./site.yml - main playbook wrapper, which includes playbooks for the various server types, such as loadbalancer.yml, for building the front end load balancer
*  ./loadbalancer.yml
  *  defines affected hosts ("loadbalancer", which in turn is a group defined in ./hosts file)
  *  references the role of type nginx, which means to include running the playbook in playbooks/roles/tasks/main.yml
  * (same idea for database.yml, control.yml, webserver.yml)

* ./playbooks/roles/nginx (same concept for each role in playbooks/roles/*)
  * ./playbooks/roles/nginx/defaults/main.yml variable defaults
  * ./playbooks/roles/nginx/vars/main.yml variables, if defined here would override the ones in defaults folder
  * ./playbooks/roles/nginx/handlers/main.yml handler definitions for "notify" actions in tasks
  * ./playbooks/roles/nginx/tasks/main.yml task step definitions
  * ./playbooks/roles/nginx/templates/nginx.conf.j2 config file template, used in the "configure nginx site" step in tasks/main.yml, using the 
  "template:" module to customize the .j2 file template into nginx.conf

# Ansible syntax samples 

##Selecting and filtering hosts to work with

#####selecting hosts
ansible --list-hosts all
ansible --list-hosts "*"

####name of group in hosts file
ansible --list-hosts loadbalancer

####wildcard filter
ansible --list-hosts "app*"

####multiple groups separated by a colon (deprecated)
ansible --list-hosts database:control

####multiple groups separated by a comma
ansible --list-hosts database,control

####select first node in webserver group
ansible --list-hosts webserver[0]

####everything not in control group, bang escaped in bash
ansible --list-hosts \!control


##simple commands

####internal ping module
ansible -m ping all

####command module (-m) executing "hostname" command (-a)
ansible -m command -a "hostname" all

####command with just -a is equiv, as command module is default module
ansible -a "hostname" all

####show what hosts are involved in this playbook
ansible-playbook playbooks/site.yml --list-hosts

####show what tags are involved in this playbook
ansible-playbook playbooks/site.yml --list-tags


## using playbooks

####simple playbook that executes "hostame" command
ansible-playbook /vagrant/playbooks/hostname.yml

####run only steps in a playbook that have a tag called "packages" defined
ansible-playbook playbooks/site.yml --tags "packages"

####run only steps in a playbook that DON'T have a tag called "packages" defined
ansible-playbook playbooks/site.yml --skip-tags "packages"

####step through tasks and be prompted whether to run each step or not
ansible-playbook playbooks/site.yml --step

####show all tasks that will be executed by the playbook
ansible-playbook playbooks/site.yml --list-tasks

####skip over steps in a playbook and start at a specific task
ansible-playbook playbooks/stack_status.yml --start-at-task "verify end-to-end response"

####verify syntax
ansible-playbook --syntax-check playbooks/site.yml

####do a simulated run of the playbook
ansible-playbook --check playbooks/site.yml

##Utilities

#### gather facts available
ansible -m setup all

