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

| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| Jump Box | Gateway  | 10.0.0.4   | Linux            |
| DVWA 1   |Web Server| 10.0.0.5   | Linux            |
| DVWA 2   |Web Server| 10.0.0.6   | Linux            |
| DVWA 3   |Web Server| 10.0.0.7   | Linux            |
| ElK      |Monitoring|  10.1.0.4  | Linux            |

