## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![](https://github.com/joshuabootcamp/Project1josh/blob/91cff7ea3d469b4d64fd79dd68bf201e6ad8b14f/Diagrams/elk-stack.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the network diagram file may be used to install only certain pieces of it, such as Filebeat.

```
Playbook01: Pentest.yml
---
- name: Config Web VM with Docker
  hosts: webservers
  become: true
  tasks:
  - name: docker.io
    apt:
      force_apt_get: yes
      update_cache: yes
      name: docker.io
      state: present

  - name: Install pip3
    apt:
      force_apt_get: yes
      name: python3-pip
      state: present

  - name: Install Docker python module
    pip:
      name: docker
      state: present

  - name: download and launch a docker web container
    docker_container:
      name: dvwa
      image: cyberxsecurity/dvwa
      state: started
      published_ports: 80:80

  - name: Enable docker service
    systemd:
      name: docker
      enabled: yes

This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build
```
```
Playbook02: Install-elk.yml
---

- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: azadmin
  become: true
  tasks:
    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        force_apt_get: yes
        name: docker.io
        state: present
      # Use apt module
    - name: Install python3-pip
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present
      # Use pip module (It will default to pip3)
    - name: Install Docker module
      pip:
        name: docker
        state: present
      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: '262144'
        state: present
        reload: yes
      # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        # Please list the ports that ELK runs on
        published_ports:
          -  5601:5601
          -  9200:9200
          -  5044:5044
      # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes
```
```
Playbook03: Filebeat.yml
---
  - name: installing and launching filebeat
    hosts: [webservers]
    become: yes
    tasks:

    - name: download filebeat deb
      command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb

    - name: install filebeat deb
      command: dpkg -i filebeat-7.6.1-amd64.deb

    - name: drop in filebeat.yml
      copy:
        src: /etc/ansible/filebeat-config.yml
        dest: /etc/filebeat/filebeat.yml

    - name: enable and configure system module
      command: filebeat modules enable system

    - name: setup filebeat
      command: filebeat setup

    - name: start filebeat service
      command: service filebeat start

    - name: enable service filebeat on boot
      systemd:
        name: filebeat
        enabled: yes
---
```
```
Playbook04: metric beat
---

- name: Install metric beat
  hosts: [webservers]
  become: true
  tasks:
    # Use command module
  - name: Download metricbeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.6.1-amd64.deb
    # Use command module
  - name: install metricbeat deb
    command: dpkg -i metricbeat-7.6.1-amd64.deb
    # Use copy module
  - name: drop in metricbeat.yml
    copy:
      src: /etc/ansible/filesd/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml
    # Use command module

  - name: enable and configure docker module
    command: metricbeat modules enable docker

    # Use command module
  - name: setup metricbeat
    command: metricbeat setup

   # Use command module
  - name: start metricbeat serivce
    command: service metricbeat start

   # Use systemd module
  - name: enable service metricbeat on boot
    systemd:
      name: metricbeat
      enabled: yes
---
```
### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly avalible  in addition to restricting access to the network.

What aspect of security do load balancers protect? 
ddos attacks from being overloaded 
What is the advantage of a jump box? 
this allow for quick and easy setup of dockers and ansble this will also add layer of security by using a ssh conntion thanks scurity network rules 

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the security rules and system users.
- What does Filebeat watch for? filebeat moniter logs data 
- What does Metricbeat records? logs and data with them

The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| Jump Box | Gateway  | 10.0.0.1   |    Linux         |
| elk-vm   | elk      | 10.1.0.4   |    linux         |
| web-vm-1 | dvwa     | 10.0.0.6   |    linux         |    
| web-vm-2 | dvwa     | 10.0.0.7   |    linux         |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the jump-box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
my personal ip

Machines within the network can only be accessed by jump-box with in ancible continer .
Which machine did you allow to access your ELK VM? What was its IP address?
Jumpbox-vm

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses   |
|----------|---------------------|------------------------|
| Jump Box | Yes                 | personal home ip       |
| web-1    | no                  | 10.0.0.5               |
| web-2    | no                  | 10.0.0.6               |
| load     | yes                 | open                   |
  balancer
### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- What is the main advantage of automating configuration with Ansible?
it quick to implment and also cost effective and is basicly a micro vm this also add to the security of data by segmenting deparment in to containers   

The playbook implements the following tasks:
- In 3-5 bullets, explain the steps of the ELK installation play. E.g., install Docker; download image; etc.
- ...
-ssh azadmin@jumpbox ip 

-start docker

-attach docker 

-cd /etc/ansible

-ansble-playbook install-elk-file.yml

- ...

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![](https://github.com/joshuabootcamp/Project1josh/blob/db4530a261d02969fb1531f4acecb9c0a06b7f37/Diagrams/Screenshot%20(114).png)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- List the IP addresses of the machines you are monitoring_
web-1 10.0.0.5 
web-2 10.0.0.6

We have installed the following Beats on these machines:
- Specify which Beats you successfully installed

-filebeat

-metricbeat

These Beats allow us to collect the following information from each machine:
-In 1-2 sentences, explain what kind of data each beat collects, and provide 1 example of what you expect to see. E.g., `Winlogbeat` collects Windows logs, which we use to track user logon events, etc._
beats 

-filebeat is use for colleting data logs of packets sent infomation we gathered can be: Ip-logs , data packets - size , location of source packets

-metricbeat is used for metrics of  systems this example of this would be oslogs of what the user used such as apache : with version 


### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the playbook YMAL file to ansible container.
- Update the hosts file to include...'
[webservers]
10.0.0.5 ansible_python_interpreter=/usr/bin/python3
10.0.0.6 ansible_python_interpreter=/usr/bin/python3
10.0.0.7 ansible_python_interpreter=/usr/bin/python3

[elk]
10.1.0.4 ansible_python_interpreter=/usr/bin/python3
- Run the playbook, and navigate to etc/ansible/ansible.cfg to check that the installation worked as expected.

Answer the following questions to fill in the blanks:_
- Which file is the playbook? 

- .YML Where do you copy it? /etc/ansible/files

- Which file do you update to make Ansible run the playbook on a specific machine? 

- How do I specify which machine to install the ELK server on 

- In the host files we asigned a vurtual machince to the elk stack  (note that it had to have 4gb of ram) 
- which excused with the install elk playbook

- which to install Filebeat on?

-ssh azadmin@jump ip 

-sudo docker start jankey_ham

-sudo docker attach jankey_ham

-ansible-playbook hamstyle.yml

-once its has install with correct configs

-you will be able to use filebeat and metricbeat though the kibana web page this let use the data from filebeat aned metricbeat  
Which URL do you navigate to in order to check that the ELK server is running?
http://elkip:5601/app/kibana


