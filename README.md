
Control Machine :- On which ansible will install
Host machine :- This is our host machine on which we want to install packages.


We will launch ubuntu 20.0 = This would be our Control Ansible.
Give name as Control_Ansible
Take Ubuntu 20.0 with t2.micro
Create key pair and download it.
In Security Group Click on Edit, Give Security Group name as Control_Ansible_SG
Launch the EC2 Instance.

--------------------------------------------------------------------

We will now launch host machine.
Give name as Host_Ansible
Take Centos 8 V CS8-20220920
Create key pair and download it.
In Security Group Click on Edit, Give Security Group name as Host_Ansible_SG
SSH from My IP
SSH from Control_Ansible_SG (To make the SSH connection  between Control and host machines)

In Instance Count make it 3 & Launch EC2 Instance.

--------------------------------------------------------------

Login to your Control Machine and install Ansible using below documents.

https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-ubuntu

After installation check ansible version.

------------------------------------------------------------------------

vi inventory.yaml 

all:
  hosts:
   web01:
     ansible_host: <privateip>
     ansible_user: centos
     ansible_ssh_private_key_file: <keyname>


Now we will need pem file of host machine. Goto your local terminal and copy keywords of your pem file. Before copy make sure your give read only permission to the pem file.
Once copy, come back to your instance terminal

vi <filename>.pem

Paste keyword

:wq

chmod 400 <filename>.pem

ansible web01 -m ping -i inventory.yaml

It is asking fingerprint message. To disable it we need to make change in ansible configuration file. 

cd /etc/ansible

cat ansible.cfg

sudo su
(Switch to root user)


ansible-config init --disabled -t all > ansible.cfg
(To Create ansible configuration file)

vi ansible.cfg

Alt + / 
search host_key_checking then press enter and i to insert. Remove ; & Make it false

exit

goto your working directory 

ansible web01 -m ping -i inventory.yaml

Now edit inventory.yaml file and add all other hosts.

all: 
 hosts: 
  web01: 
   ansible_host: <PrivateIP> 
   ansible_user: centos 
   ansible_ssh_private_key_file: <pemfile> 
  web02: 
   ansible_host: <PrivateIP> 
   ansible_user: centos 
   ansible_ssh_private_key_file: <pemfile> 
  db01: 
   ansible_host: <PrivateIP> 
   ansible_user: centos 
   ansible_ssh_private_key_file: <pemfile>


ansible web02 -m ping -i inventory.yaml

ansible db01 -m ping -i inventory.yaml

ansible all -m ping -i inventory.yaml

cp -r praticse1/ practise2

vi inventory.yaml

all:
  hosts:
    web01:
      ansible_host: 172.31.31.197
      ansible_user: centos
      ansible_ssh_private_key_file: Host_Ansible.pem
    web02:
      ansible_host: 172.31.18.26
      ansible_user: centos
      ansible_ssh_private_key_file: Host_Ansible.pem
    db01: 
      ansible_host: <privateIP>
      ansible_user: centos
      ansible_ssh_private_key_file: <pemfile>
  children:
    webservers:
      hosts:
        web01:
        web02: 
    dbservers:
      hosts:
        db01:
    dc_oregan:
      children:
       webservers:
       dbservers:



ansible webservers -m ping -i inventory.yaml

ansible dbservers -m ping -i inventory.yaml 

ansible dc_oregan -m ping -i inventory.yaml

➡️Ad-Hoc Commands

https://docs.ansible.com/ansible/latest/command_guide/intro_adhoc.html

ansible webservers -m ansible.builtin.yum -a "name=httpd state=present" -i inventory.yaml --become
(We will install httpd package with single ad-hoc command)

ansible webservers -m ansible.builtin.service -a "name=httpd state=started enabled=yes" -i inventory.yaml --become
(We will start and enable httpd service from it)

ansible webservers -m ansible.builtin.copy -a "src=index.html dest=/var/www/html/index.html" -i inventory.yaml --become

ansible webservers -m ansible.builtin.yum -a "name=httpd state=absent" -i inventory.yaml --become


➡️Playbook and module

Document Link :- 

ansible.builtin.yum
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/yum_module.html#ansible-collections-ansible-builtin-yum-module

ansible.builtin.service
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/service_module.html#ansible-collections-ansible-builtin-service-module

Playbook is collection of plays. Like installing or uninstalling packages, copy file from local to remote servers or copying any custom configuration file to remote servers, etc. 

Playbook written in yaml format. Below is sample playbook with single play & single task to install httpd package.

-hosts: websrvgrp
 tasks:
  - yum:
      name: httpd
      state: present


Lets write our own playbook 

vi playbook.yaml


---

- name: Webserver Setup
  hosts: webservers
  become: yes
  tasks:
    - name: Install httpd
      ansible.builtin.yum:
        name: httpd
        state: present
    - name: Start service
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: yes
- name: Dbservers setup
  hosts: dbservers
  become: yes
  tasks:
    - name: install mariadb server
      ansible.builtin.yum:
        name: mariadb-server
        state: present
    - name: Start mariadb server
      ansible.builtin.service:
        name: mariadb
        state: started
        enabled: yes


ansible-playbook -i inventory.yaml dbplaybook.yaml
