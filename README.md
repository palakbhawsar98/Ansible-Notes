<a href="https://jenkins.io">
  <picture>
    <source width="300" media="(prefers-color-scheme: dark)" srcset="https://www.vectorlogo.zone/logos/ansible/ansible-ar21.png">
    <img width="300" src="https://www.vectorlogo.zone/logos/ansible/ansible-ar21.png">
  </picture>
</a>

### What is Ansible
Ansible is an open source IT automation tool that automates provisioning, configuration management, application deployment, orchestration, and many other manual IT processes.

### Use Cases of Ansible
- Automation(Any system automation, Server, Database, configuration, start restart services)
- Change Management (Production server changes)
- Provisionning(Setup server from scratch or cloud provisioning)
- Orchestration(Large scale automation framework, can integrate with other tool like jenkins, docker)

**Ansible is simple. No need of any agent to manage various servers. Target machines/Services are accessed by SSH, winrm & API**
![image](https://user-images.githubusercontent.com/69889600/219846350-a0c53ac4-0b2f-4781-ac6c-a5dd512ea649.png)

**Ansible uses the python module, python script to connect to the target machine. It dumps the python script and executed there and return the output.**

![image](https://user-images.githubusercontent.com/69889600/219847528-a54c5417-127c-4e44-9762-72e4d1a33775.png)

### What is Ansible Playbook
Playbooks are the files where the Ansible code is written. Playbooks are written in YAML format. YAML means "Yet Another Markup Language,".
It is basically a blueprint of automation tasks—which are complex IT actions executed with limited or no human involvement.

### What is Ansible Inventory
Inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. The file can be in one of many formats depending on your Ansible environment and plugins. The default inventory located at /etc/ansible/hosts

### What is Ansible Config file
The file that governs the behavior of all interactions performed by the control node. In Ansible’s case that default configuration file is (ansible.cfg) located in /etc/ansible/ansible.cfg. 

- Launch an EC2 instance and install Ansible ( This is the control machine)
- Launch more EC2 instances to manage them 
- Create a inventory file in your control machine in Project directory
- Don't give password instead ive private key 

### Hosts Entry in Inventory file

```
host_name ansible_host=private_addr ansible_user=ubuntu ansible_ssh_private_key_file=instance.pem 
```

Ansible provide ad-hoc commands to execute module on target from the shell. Ping will do ssh into Linux machine, if it's succesful it means connection between ansible and hosts are right.
- For particular host use host_name in below command:
```
ansible -i inventory -m ping host_name
```
- For all hosts use below commands:
```
ansible -i inventory -m ping all
```
```
ansible -i inventory -m ping '*'
```

Add servers in group in Inventory file to manage all at once instead of executing commands for each server.
```
host_name1 ansible_host=private_addr ansible_user=ubuntu ansible_ssh_private_key_file=instance.pem 
host_name2 ansible_host=private_addr ansible_user=ubuntu ansible_ssh_private_key_file=instance.pem 
host_name3 ansible_host=private_addr ansible_user=ubuntu ansible_ssh_private_key_file=instance.pem 
-----Group-----
[webserver]
host_name1
host_name2
[dbserver]
host_name3
-----Group of Groups----
[AllServers:children]
webserver
dbserver
-----Variables at group level---
[AllServers:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=instance.pem 
```
Install httpd in web01 (use --become tp elevate rights as ubuntu user doesn't have permission to install https service)
```
ansible -i inventory -m apt-get -a "name-httpd state-present" host_name1 --become
```
Modules are idempotent in ansible meaning if you run a playbook with the same set of inputs, you should not expect it to make any changes on the system.

### Ansible playbook
Write playbook to install httpd in webserver[host_name1,host_name2] from inventory file:

```
---
- name: Setup webserver
  hosts: webserver
  become: yes
  tasks:
    - name: Install httpd
      yum:
         name: httpd
         state: present 
    - name: Copy html file     
      copy:
         src: index.html
         dest: /www/html/index.html
```
### Command to check playbook:
```
ansible-playbook -i inventory playbook_name.yml --syntax-chcek
```
### Command to run playbook:
```
ansible-playbook -i inventory playbook_name.yml
```
### To find use of any module
```
ansible-doc module_name
```
To know more about modules search it in official Ansible documentation

### Ansible Configuration
Change the default behaviour of ansible configuration by editing coniguration file

#### Order of Ansible Config
- ANSIBLE_CONFIG (set environment variable)
- ansible.cfg (in the current directory)
- ~/.ansible.cfg (in the home directory)
- /etc/ansible/ansible.cfg

## Variables
Vars defines the variables which you can use in your playbook. Its usage is similar to the variables in any programming language.
Example:vars:
```
- name: Setup webserver
  hosts: webserver
  vars:
     http_port: 80
```
We can define inventory based variables in the below path that's applicable globally for all hosts:
- group_vars/all
- group_vars/groupname
- host_vars/hostname

#### Fact variables:
Fact variables are runtime variables that get generated when setup module gets executed such as
- ansible_os_family
- ansible_processor_cores
- ansible_kernel
- ansible_default_ipv4

### Commands to execute setup module
```
ansible -m setup hostname 
```
### Use debug module in Ansible is to prints simple statements to stdout
```
- name: Printing ansible distribution
  hosts: all
  tasks: 
     - name: Print 
       debug:
          var: ansible_distribution

```        
### Conditions
Conditions in playbook are same as 'if/else' conditions in programming, use **when** in Ansible:   
Syntax: 
```
      tasks:
         - name: Install httpd service Ubuntu
           yum: 
             name: httpd
             state: present
           when: ansible_distribution == 'Ubuntu'
         - name: Install httpd service Centos 
           apt:
             name: httpd
             state: present
           when: ansible_distribution == 'Centos'
```    
### Loops
Loops are used to iterate the condition in playbook for diffrent servers:
Syntax:
```
    tasks:
         - name: Install httpd service Ubuntu
           yum: 
             name: "{{item}}"
             state: present
           when: ansible_distribution == 'Ubuntu'
           loop:
             - httpd
             - git
             - zip
         - name: Install httpd service Centos 
           apt:
             name: "{{item}}"
             state: present
             update_cache: yes
           when: ansible_distribution == 'Centos'
           loop:
             - httpd
             - git
             - zip
 ```     
### Handlers
Sometimes you want a task to run only when a change is made on a machine. For example, you may want to restart a service if a task updates the configuration of that service, but not if the configuration is unchanged. Ansible uses handlers to address this use case. Handlers are tasks that only run when notified.
            
```
---
- name: Verify apache installation
  hosts: webservers
  vars:
    http_port: 80
    max_clients: 200
  remote_user: root
  tasks:
    - name: Ensure apache is at the latest version
      ansible.builtin.yum:
        name: httpd
        state: latest

    - name: Write the apache config file
      ansible.builtin.template:
        src: /srv/httpd.j2
        dest: /etc/httpd.conf
      notify:
      - Restart apache

    - name: Ensure apache is running
      ansible.builtin.service:
        name: httpd
        state: started

  handlers:
    - name: Restart apache
      ansible.builtin.service:
        name: httpd
        state: restarted
```
