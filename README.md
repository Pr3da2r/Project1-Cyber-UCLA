# Project1-Cyber-UCLA
# Automated Elk Stack Deployment 
Link to the Network Diagram of the Automated Elk Stack Deployment : https://drive.google.com/file/d/1sd9EQCLo42nhJksa735XsW_DbWa1a41j/view?usp=sharing (JPEG also added to the project).
This document contains the following details:
- Description of the Toplology
- Elk Configuration : 
                    - beats in use
                    - machines being monitored
- How to use the Ansible Build
- Security/Access Policies
The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the "Damn Vulnerable Web Application".
Load balancing ensures that the application will be highly available, in addition to restricting inbound access to the network. The load balancer ensures the incoming traffic will be shared by all the vulnerable veb servers. Access controls will ensure that only authorized users (namely ourselves) will be able to connect in the first place.
Integrating an Elk server allows users to easily monitor the vulnerable VM's for changes to the file system of the VM's on the nertwork, as well as watch the system metrics, such as CPU usage, attempted SSH logins, sudo escalation failuers, web traffic errors 

The configuration details of each machine may be found below.

| Name     | Function           | IP Address      | Operating System |
|----------|--------------------|-----------------|------------------|
| Jump Box | Gateway            | 10.0.0.4/24     | Linux            |
| DVWA 1   |Web Server          | 10.0.0.5/24     | Linux            |
| DVWA 2   |Web Server          | 10.0.0.6/24     | Linux            |
| DVWA 3   |Web Server          | 10.0.0.7/24     | Linux            |
| ElK      |Monitoring          | 10.1.0.4/24     | Linux            |

In addition a load balancer was provisioned in front of all the Web Servers except the Jump Box. The load balancer has a public ip address of 104.42.123.52 and his target is the availability zone 1 of the 3 Web Servers. All the Web VM's icluding the Jump-Box and Load Balancer are in the same region West US Elk Server is situated in a diffrent region than the rest East US with a public Ip address of: 52.188.65.248

ELK Server Configuration

The ELK VM exposes an Elastic Stack instance. Docker is used to download and manage anELK container.
In order to configure Elk on a more efficient manner we deployed a reusable Ansible Playbook to accomplish the task. To use this playbook, one must log into the Jump Box start the docker container which holds all the configuration and playbook (.yml) files, then in order to deploy this automation configuratio of the Elk-Stack we must issue the following commands whithingn the conontainer: ansible-playbook install-elk.yml

Access Policies

The machines on the internal network will not be accessible from the internet. Only the jup box VM can accept connections from the internet.Access to this machine allows only one connection from the ip 76.174.176.123.
Machines whithin the network can only be accessed by eachother. The DVWA1, DVWA2 and DVWA3 VM's send traffic to the Elk Server.

Bellow is the summary of the access policies in place:

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes                 | 104.42.123.52        |
| Elk      | No                  | 10.1.0.4/24          |
| DVWA1    | No                  | 10.0.0.5/24          |
| DVWA2    | No                  | 10.0.0.6/24          |
| DVWA3    | No                  | 10.0.0.7/24          |

Elk Configuration

As mentioned above we used Ansible to automate the cofiguration of the Elk Server. No manual configuration was performed. The advantages of running an automated configuration is that we can deploy and update multiple machines at the same time, and make sure they have the same applications and they all do the things we want them to do just by delpoying them trough the Ansible container.

The playbook implements the following tasks:
- install the docker.io on the Jump-Box VM
- pulled the Ansible container
- added the internal ip's of the desired VM's into the hosts file in ansible
- change the Ansible configuration file to use our administrator account for SSH connections
- run the config and playbook files for Elk Server 

The following screenshot displays the result of running docker ps after successfullyconfiguring the ELK instance.
![image](https://user-images.githubusercontent.com/77358715/117213001-e6eb6280-adaf-11eb-9334-3630a8dc1ed6.png)

The playbook is duplicated bellow:

---
- name: Configure Elk VM with Docker
  hosts: Elk
  remote_user: Pr3da2r
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

      # Use command module
    - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144

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


Target Machines & Beats

The Elk server is configured to monitor the DVWA1, DVWA2 and DVWA3 VM's at 10.0.0.5, 10.0.0.6 and 10.0.0.7
We have installed the following Beats on these machines:
- Filebeat
- Metricbeat
These Beats allow us to collect the following information from each machine:
- Filebeat detects changes to the filesystem. Specifically, we use it to collectApache logs.
- Metricbeat detects changes in system metrics, such as CPU usage. We useit to detect SSH login attempts, failed sudo escalations, and CPU/RAM statistics.

The playbook bellow installs Filebeat on target hosts. The playbook for installing Metricbeat is similar we just have to replace filebeat with metricbeat, and it will work as expected.

---
- name: installing and launching filebeat
  hosts:  webservers
  become: yes
  tasks:

  - name: download filebeat deb
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.$
  - name: install filebeat deb
    command: dpkg -i filebeat-7.4.0-amd64.deb

  - name: drop in filebeat.yml
    copy:
      src:  /etc/ansible/files/filebeat-config.yml
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
      
  Using the Playbooks
In order to use the playbooks, you will need to have an Ansible control node alreadyconfigured. We use the jump box for this purpose.
To use the playbooks, we must perform the following steps:
   - Copy the playbooks to the Ansible Control Node
   - Run each playbook on the appropriate targets
The easiest way to copy the playbooks is to use Git:    

   $cd /etc/ansible/files
   #Clone Repository + IaC Files
   $ git clone https://github.com/Pr3da2r/Project1-Cyber-UCLA.git
   # mkdir Ansible Diagrams Linux
   # Move Playbooks and hosts file Into '/etc/ansible/files/Project1-Cyber-UCLA/Ansible'
   $ cp install-elk.yml metricbet-playbook.yml filebeat-playbook.yml /etc/ansible/files/Project1-Cyber-UCLA/Ansible
   $ cp hosts /etc/ansible/files/Project1-Cyber-UCLA/Ansible/

Next, you must create a hosts file to specify which VMs to run each playbook on, like the bellow file:
 # This is the default ansible 'hosts' file.
#
# It should live in /etc/ansible/hosts
#
#   - Comments begin with the '#' character
#   - Blank lines are ignored
#   - Groups of hosts are delimited by [header] elements
#   - You can enter hostnames or ip addresses
#   - A hostname/ip can be a member of multiple groups

# Ex 1: Ungrouped hosts, specify before any group headers.

## green.example.com
## blue.example.com
## 192.168.100.1
## 192.168.100.10

# Ex 2: A collection of hosts belonging to the 'webservers' group

[webservers]
10.0.0.5 ansible_python_interpreter=/usr/bin/python3
10.0.0.6 ansible_python_interpreter=/usr/bin/python3
10.0.0.7 ansible_python_interpreter=/usr/bin/python3
[Elk]
10.1.0.4 ansible_python_interpreter=/usr/bin/python3
## alpha.example.org
## beta.example.org
## 192.168.1.100
## 192.168.1.110

# If you have multiple hosts following a pattern you can specify
# them like this:

## www[001:006].example.com

# Ex 3: A collection of database servers in the 'dbservers' group

## [dbservers]
##
## db01.intranet.mydomain.net
## db02.intranet.mydomain.net
## 10.25.1.56
## 10.25.1.57

# Here's another example of host ranges, this time there are no
# leading 0s:

## db-[99:101]-node.example.com
 
After this, the commands below run the playbook:
$cd /etc/ansible 
$ ansible-playbook install-elk.yml
$ ansible-playbook filebeat-playbook.yml
$ ansible-playbook metricbeat-playbook.yml
To verify success, wait five minutes to give ELK time to start up.
Then, run:
curl http://10.1.0.4:5601. This is the address of Kibana. If the installation succeeded, this command should print HTML to the console.


