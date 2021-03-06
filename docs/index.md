### **Introduction** 
  	- Configuration Management Tool Works on **SSH**
	- Agentless
	- Use Playbook for automation task 
	- Use Yaml files 
	- Ansible gives Kerberos 
	
### **Installation by apt**
	- sudo apt-get update 
	- sudo apt-get install software-properties-common 
	- sudo apt-add-repository ppa:ansible/ansible $ sudo apt-get update 
	- sudo apt-get install ansible


### Working With Ansible
  
 **1:ad-hoc commands**
 
	- Use for and Quick simple tasks
	- Use for temprary use
	- Always override every command

**2:module**

	- can executes directly on hosts
	- no database , deamons, servers requires.
	- modules comes with ansible package on server node.

**3:playbooks**

	- if we need to write two or more modules in one file - playbook comes in

## **Setup inventory**

By Deafult inventory file is : **/etc/ansible/hosts**

Edit file and add your hosts or group

	[gang] 		#Hosts Group Name
	10.0.3.44 	#Host1
	10.0.3.45 	#Host2
	10.0.3.46 	#Host3

Install **Openssh-server** and traffic on 22 port on all Hosts.


**Ad-hoc Commands**

	#Create abc.txt file on all avaliable nodes as per inventory
	ansible all -a "touch abc.txt" 
	
	#Create abc.txt file on 'gang' group which has 3 nodes
	ansible gang -a "touch abc.txt"   	 
	
	#Install nginx on gang group with sudo previledge
 	ansible gang -a "sudo apt install nginx"  
	
	#Range for first five nodes in "Gang: group"
	ansible gang[0:4] -ba "apt install nginx" 

	#**use sudo or -b :: same working :: either use sudo or use -b option** 
	ansible gang -a "sudo apt update -y"
	#or
	ansible gang -ba "apt update -y"



## Modules: -m

**basic**

	install   = present
	uninstall = absent
	update    = latest
	idempotency preset -- means if work does not repeat 

**Package Management**

	ansible gang -b(use for sudo) -m apt -a "pkg=httpd state=present"
	ansible gang -b -m yum -a "pkg=httpd state=absent"
	ansible gang -b -m yum -a "pkg=httpd state=latest"

**Service Management**

	ansible gang -b -m service -a "name=httpd state=started enabled=yes"
	ansible gang -b -m service -a "name=httpd state=stoped"
	ansible gang -b -m service -a "name=httpd state=restarted"
	ansible gang -b -m service -a "name=httpd state=reloaded"

**Copy File**

	ansible gang -b -m copy -a "src=file1.txt dest=/tmp"
	
	#copy file on last node in gang group
	ansible gang[-1] -b -m copy -a "src=file1.txt dest=/tmp"

**Setup Module**

	ansible gang -m setup
	ansible gang -m setup -a "filter=*ipv4*"
	
	


## Playbooks


**#1. Basic.yaml**

	--- # start with three hyphen
	- hosts: gang
	  user: ansible/root
	  become: yes                               - become root user
	  connection: ssh			    - by default "ssh"
	  gather_facts: yes			    - get data about nodes
	  
#> ansible-playbook basic.yaml  
  
**#2. task.yaml**

	---
	- hosts: gang
	  user: root
	  connection: ssh
	  tasks:
	   - name: install httpd on linux
	     action: yum name=httpd state=installed

     
**#3. task_with_var.yaml**

	---
	- hosts: gang
	  user: root
	  connection: ssh
	  vars:
	   pkg : httpd
	  tasks:
	   - name: install httpd on linux
	     action: yum name="{{pkg}}" state=installed

     

## HANDLERS:

**handler.yaml**

	---
	- hosts: gang
	  user: root
	  connection: ssh
	  vars:
	   pkg : httpd
	  tasks:
	   - name: install httpd on linux
	     action: yum name="{{pkg}}" state=installed
	     notify: handler_one		# any msg

	  handler:
	   - name: handler_one
	     action: service name:"{{pkg}}" state=restarted







## LOOPS: 
**item -- with_items**

	---
	- hosts: gang
	  user: root
	  connection: ssh
	  vars:
	   pkg : httpd
	  tasks:
	   - name: add users in linux
	     user: name="{{item}}" state=present
	     with_items:
	      - shubham
	      - kunal
	      - tushar




## Conditional

**IMP:** 
**	- always use underscore while writing conditional scripts like "when: ansible_os_family == "RedHat""
**


	--- #install apache on linux machin not on redhat machine
	- hosts: gang
	  tasks:
	   - name: install apache on ubuntu/dabian base os
	     command : apt install apache2 -y
	     when: ansible_os_family == "dabian"
	   - name: install apache on Red-Hat based os
	     command : yum install apache2 -y
	     when: ansible_os_family == "RedHat"
	     
	     
     	---
	- hosts: all
	  tasks:
		- name: Install VIM via yum
			yum:
				name: vim-enhanced
				state: installed
				when: ansible_os_family == "RedHat"
		- name: Install VIM via apt
			apt:
				name: vim
				state: installed
				when: ansible_os_family == "Debian"
		- name: Unexpected OS family
			debug:
			msg: "OS Family {{ ansible_os_family }} is not supported"
			fail: yes
			when: ansible_os_family != "RedHat" and ansible_os_family != "Debian"
  
  

## VAULT
- encrypt and decrypt playbooks
- for security 

**Examples**

	#encrypt already created file
	-$] ansible-vault encrypt install_apache2.yaml
	password: 
	re-enter password:

	#dencrypt already created file
	-$] ansible-vault decrypt install_apache2.yaml
	password: 

	#create new file 
	#>ansible-vault create install_apache2.yaml
	password:

	#change password of playbook  
	#>ansible-vault rekey install_apache2.yaml
	old password:
	New password:
	Re-enter:


	#edit existing file
	#>ansible-vault edit install_apache2.yaml
	password:


### Thank You 
