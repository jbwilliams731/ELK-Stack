## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![ELK Stack Project Diagram](https://user-images.githubusercontent.com/76117195/117526217-80657080-af78-11eb-9950-ac4ca8c77d00.jpeg)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the elksetup.yml file may be used to install only certain pieces of it, such as Filebeat.
 
```
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: azureuser
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
```

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the "D*mn Vulnerable Web Application".

Load balancing ensures that the application will be highly available, in addition to restricting unauthorized inbound access to the network.
The load balances ensure that the incoming traffic is split between all three servers running the DVWA. Using a Jump Box gave us another layer of access control by only allowing authorized users, myself, to connect to the network. 

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to their file systems as well as report their system metrics to easily monitor things like resource usage, network IO, escalation failures, and SSH logins. 

The configuration details of each machine may be found below.

| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| Jump Box | Gateway  | 10.0.0.5   | Linux            |
| Web-1    |Web Server| 10.0.0.7   | Docker           |
| Web-2     |Web Server  | 10.0.0.9   | Docker    |
| Web-3     |Web Server    | 10.0.0.11   | Docker     |
| ELK-Server     |Monitoring    | 10.2.0.4   | Docker       |

In addition a load balancer has been setup in front of the web servers and dropped them into an availability set:

-WEB-SET: Web-1 + Web-2 + Web-3

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump Box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- 47.149.52.19

Machines within the Red Virtual network can only be accessed by the Jump Box. The ELK Server is only allowed to be accessed by our local machine (47.149.52.19) via port 5601 to give us access to Kibana's web app. 


A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes                 | 47.149.52.19    |
| ELK Server| Yes                | 47.149.52.19                      |
| Web-1        | No              | 10.0.0.1-10.0.255.254                    |
| Web-2        | No              | 10.0.0.1-10.0.255.254                     |
| Web-3        | No              | 10.0.0.7-10.0.255.254                     |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because of a variety of reasons. Firstly, if the files are properly configured it eliminates the possibility of human error. Secondly, it allows us to rapidly scale deployment if necessary and spin up large numbers of this (or similar) machines with proper config files and playbooks

The playbook implements the following tasks:
- Download Docker.io
- Install Python 3 PIP
- Use the PIP module to install the docker.io
- Increase virtual memory
- Use max memory available
- Download and launch a docker ELK container from image:sebp/elk761
- Enable docker on boot

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![docker ps on ELK](https://user-images.githubusercontent.com/76117195/117540333-acf2aa00-afc3-11eb-947d-9ea44d8bd030.png)


### Target Machines & Beats
This ELK server is configured to monitor the following machines:
| Machine Name  |IP   |
|---|---|
|Web-1   |10.0.0.7   |
|Web-2   |10.0.0.9   |
|Web-3   |10.0.0.11   |

We have installed the following Beats on these machines:
- Metricbeat 
- Filebeat 

These Beats allow us to collect the following information from each machine:
- Metricbeat installation allows us to monitor the metrics of each machine, such as CPU Usage, Memory usage, number of containers on each machine, and Network IO for each machine and as a whole.
- Filebeat allows us to monitor the file systems on each VM giving us visibility over things like escalation attempts, sudo commands, SSH login, etc. 
-
### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the elksetup.yml playbook file to Ansible container inside /etc/ansible/.
- Update the hosts file to include the ELK Server's private IP address under a new group of hosts known as [elk]   
- Run the playbook, and navigate to http://20.69.240.232:5601/app/kibana to check that the installation worked as expected.
- To navigate to the ELK server we must connect via our local machine directly to the ELK public IP and via port 5601.


