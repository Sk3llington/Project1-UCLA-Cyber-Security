# Project-1-UCLA-Cyber-Security

Deployment of an ELK Stack server to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

## Automated ELK Stack Deployment

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
 - Beats in Use
 - Machines Being Monitored
- How to Use the Ansible Build

### Description of the Topology

The files in this repository were used to configure the network depicted below.

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the YAML files may be used to install only certain pieces of it, such as Filebeat and Metricbeat.


![vNet Diagram](https://github.com/Sk3llington/Project1-UCLA-Cyber-Security/blob/main/Diagrams/Azure_RedTeam1_vNet_diagram.png)


The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting inbound access to the network.

  - The load balancer's main purpose is to distribute a set of tasks over a set of resources to make the overall processing more efficient. In our network, the load balancer's main goal is to process incoming traffic and make sure that it is shared by all 3 vulnerable web servers.

Why a Jump Box?

  - A Jump Box or a "Jump Server" is a gateway on a network used to access and manage devices in different security zones. A Jump Box acts as a security layer between networks and/or security zones and provides a controlled way to access them.

Via the Jump Box, we make sure that access controls are in place to ensure that only authorized users (in this case, ourselves), will be able to connect to the network.


Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to their file systems and system metrics such as privilege escalation failures, SSH logins activity, CPU and memory usage, etc.


The configuration details of each machine may be found below.


| Name       | Function   | IP Address | Operating System |
|------------|------------|------------|------------------|
| Jump Box   | Gateway    | 10.0.0.4   | Linux            |
| Web-1      | Web server | 10.0.0.5   | Linux            |
| Web-2      | Web server | 10.0.0.6   | Linux            |
| Web-3      | Web server | 10.0.0.7   | Linux            |
| ELK-Server | Monitoring | 10.1.0.4   | Linux            |

In addition to the above, Azure has provisioned a load balancer in front of all machines except for the jump box. The load balancer's targets are organized into the following availability zones:


- Availability Zone 1: Web-1 + Web-2 + Web-3

- Availability Zone 2: ELK-Server

## ELK Server Configuration

The ELK VM exposes an Elastic Stack instance. Docker is used to download and manage an ELK container.

Rather than configure ELK manually, we opted to develop a reusable Ansible Playbook to accomplish the task. This playbook is duplicated below.

To use this playbook, one must log into the Jump Box, then issue the command: 

```
ansible-playbook install_elk.yml
```

This runs the install_elk.yml playbook on the elk host.

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the Jump Box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP address: 91.219.212.205

*Note that if you have a dynamic IP address or if you are using a VPN, your IP address will change in the future and you will have to update your Network Security Group settings to allow access from your new IP address. The same update will apply for all rules that allow for a connection over a public IP address from your workstation only.*

Machines within the network can only be accessed by each other, i.e., the Web-1, Web-2, Web-3 web servers send traffic to the ELK Server (see diagram). 

A summary of the access policies in place can be found in the table below.

| Name       | Publicly Accessible | Allowed IP Addresses |
|------------|---------------------|----------------------|
| Jump Box   | Yes                 | 91.219.212.205       |
| Web-1      | No                  | 10.0.0.1-254         |
| Web-2      | No                  | 10.0.0.1-254         |
| Web-3      | No                  | 10.0.0.1-254         |
| ELK-Server | No                  | 10.0.0.1-254         |

### Elk Configuration

Ansible was used to automate configuration of the ELK server. No configuration was performed manually, which is advantageous because Ansible can be used to easily configure new machines, update programs and configurations on hundreds of servers at once, and the best part is that the process is the same whether you're managing one machine or dozens and even hundreds.

The playbook implements the following tasks:

```yaml
---
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: Web_1
  become: true
  tasks:
```

- In the above play, representing the header of the YAML file, we defined the title of our playbook based on the playbook's main goal by setting the keyword 'name:' to: "Configure Elk VM with Docker". 

Next we defined the managed nodes to target, in this case we set the keyword 'hosts:' to "elk", making sure that the playbook is run only on the machines in the "elk" group. 
To edit groups and add/remove machines from a group, the following inventory file located in /etc/ansible is used (see image below).

![hosts file editing](https://github.com/Sk3llington/Project1-UCLA-Cyber-Security/blob/main/Images/hosts_file_web_servers_edit.png) 

Next we defined the user account for the SSH connection, i.e., Web_1, by setting the keyword 'remote_user:' to "Web_1".

Next we activated privilege escalation by setting the keyword 'become:' to "true". 

Following the keyword 'tasks:', the second play is defined below.

```yaml
     # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present
```

In this play, the ansible package manager module is tasked with installing docker.io. The keyword 'update_cache:' is set to "yes" to download package information from all configured sources and their dependencies prior to installing docker, it is necessary to successfully install docker in this case. Next the keyword 'state:' is set to "present" to verify that the package is installed.

```yaml
      # Use apt module
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present
```

In this play, the ansible package manager module is tasked with installing  'pip3', a version of the 'pip installer' which is a standard package manager used to install and maintain packages for Python.
The keyword 'force_apt_get:' is set to "yes" to force usage of apt-get instead of aptitude. The keyword 'state:' is set to "present" to verify that the package is installed.

```yaml
      # Use pip module
    - name: Install Docker python module
      pip:
        name: docker
        state: present
```

In this play the pip installer is used to install docker and also verify afterwards that docker is installed ('state: present').

```yaml
      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes
```

In this play, the ansible sysctl module configures the target virtual machine (i.e., the Elk server VM) to use more memory. On newer version of Elasticsearch, the max virtual memory areas is likely to be too low by default (ie., 65530) and will result in the following error: "elasticsearch | max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]", thus requiring the increase of vm.max_map_count to at least 262144 using the sysctl module (keyword 'value:' set to "262144"). The keyword 'state:' is set to "present" to verify that the change was applied. The sysctl command is used to modify Linux kernel variables at runtime, to apply the changes to the virtual memory variables, the new variables need to be reloaded so the keyword 'reload:' is set to "yes" (this is also necessary in case the VM has been restarted).

```yaml
      # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044
```

In this play, the ansible docker_container module is used to download and launch our Elk container. The container is pulled from the docker hub repository. The keyword 'image:' is set with the value "sebp/elk:761", "sebp" is the creator of the container (i.e., Sebastien Pujadas). "elk" is the container and "761" is the version of the container. The keyword 'state:' is set to "started" to start the container upon creation. The keyword 'restart_policy:' is set to "always" and will ensure that the container restarts if you restart your web vm. Without it, you will have to restart your container when you restart the machine.
The keyword 'published_ports:' is set with the 3 ports that are used by our ELK stack configuration, i.e., "5601" is the port used by Kibana, "9200" is the port used by Elasticsearch for requests by default and "5400" is the default port Logstash listens on for incoming Beats connections (we will go over the Beats we installed in the following section "Target Machines & Beats").

```yaml
      # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes
```

In this play, the ansible systemd module is used to start docker on boot, setting the keyword 'enabled:' to "yes".


The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![Docker ps output](https://github.com/Sk3llington/Project1-UCLA-Cyber-Security/blob/main/Images/elk_docker_ps_output.png)

### Target Machines & Beats

This ELK server is configured to monitor the following machines:

- Web-1 (DVWA 1) | 10.0.0.5
- Web-2 (DVWA 2) | 10.0.0.6
- Web-3 (DVWA 3) | 10.0.0.7

We have installed the following Beats on these machines:

- Filebeat
- Metricbeat

These Beats allow us to collect the following information from each machine:

`Filebeat`: Filebeat detects changes to the filesystem. We use it to collect system logs and more specifically, we use it to detect SSH login attempts and failed sudo escalations.

Filebeat playbook we used below:

```yaml
---
- name: Install and Launch Filebeat
  hosts: webservers
  become: yes
  tasks:
    # Use command module
  - name: Download filebeat .deb file
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb
    # Use command module
  - name: Install filebeat .deb
    command: dpkg -i filebeat-7.4.0-amd64.deb
    # Use copy module
  - name: Drop in filebeat.yml
    copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml
    # Use command module
  - name: Enable and Configure System Module
    command: filebeat modules enable system
    # Use command module
  - name: Setup filebeat
    command: filebeat setup
    # Use command module
  - name: Start filebeat service
    command: service filebeat start
    # Use systemd module
  - name: Enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes
```


`Metricbeat`: Metricbeat detects changes in system metrics, such as CPU usage and memory usage.

Metricbeat playbook we used below:

```yaml
---
- name: Install and Launch Metricbeat
  hosts: webservers
  become: true
  tasks:
    # Use command module
  - name: Download metricbeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb
    # Use command module
  - name: install metricbeat
    command: dpkg -i metricbeat-7.4.0-amd64.deb
    # Use copy module
  - name: drop in metricbeat config
    copy:
      src: /etc/ansible/files/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml
    # Use command module
  - name: enable and configure docker module for metric beat
    command: metricbeat modules enable docker
    # Use command module
  - name: setup metric beat
    command: metricbeat setup
    # Use command module
  - name: start metric beat
    command: service metricbeat start
    # Use systemd module
  - name: Enable service metricbeat on boot
    systemd:
      name: metricbeat
      enabled: yes
```

### Using the Playbooks

In order to use the playbooks, you will need to have an Ansible control node already configured (we use our Jump Box as the Ansible control node), copy the playbooks to the Ansible control node and run the playbooks on the appropriate targets. 

First, we SSH into the control node and follow the steps below:

- Copy the playbook files to the Ansible control node.
- Update the "hosts" file to include the groups of hosts representing the targeted servers to run the playbooks on.
- Run the playbooks, and navigate to the ELK server to check that the installation worked as expected.

First we connect to our Jump Box using the following command to SSH into the box:

```
ssh azadmin@51.141.166.114
```


![SSH into ump box](https://github.com/Sk3llington/Project1-UCLA-Cyber-Security/blob/main/Images/SSH_into_Jump_Box.png)

Then we run the following command to start and launch our Ansible docker container (i.e., our Ansible Control Node):

```
sudo docker start hopeful_lalande && sudo docker attach hopeful_lalande 
```

Note: Your container will have a different name.

![Start and launch ansible container](https://github.com/Sk3llington/Project1-UCLA-Cyber-Security/blob/main/Images/start_launch_ansible_container.png)

We then copy the playbooks into the correct location. The easiest way to do that is to use Git and run the following commands in your terminal:

```
cd /etc/ansible

mkdir files

# Clone Repository + IaC Files

git clone https://github.com/yourusername/projectname.git

# Move Playbooks and hosts file Into `/etc/ansible`

cp projectname/playbooks/* .

cp projectname/files/* ./files
```

Now that all the files we need are copied into the correct location, we can update the list of web servers to run the playbooks on:

We need to edit the "hosts" file located in /etc/ansible using the following commands:

```
nano hosts
```


Then we will update the file with the IP of web servers we want to install Filebeat & Metricbeat & ELK on. To create a group we need to use brackets "[]", give the group of server a name (i.e., "webservers" & "elk") followed by the private IP addresses of the servers.

![hosts file web server edit](https://github.com/Sk3llington/Project1-UCLA-Cyber-Security/blob/main/Images/hosts_file_web_servers_edit.png)

Next, we run the playbooks.

First we run our ELK playbook to deploy our ELK server:

```
ansible-playbook install_elk.yml
```

Then we run the Filebeat and Metricbeat playbooks to to install the agents on our web servers (Web-1, Web-2, Web-3):

```
ansible-playbook install_filebeat.yml
```
```
ansible-playbook install_metricbeat.yml
```


To verify that our ELK server was successfully deployed, we SSH into our ELK server and run the following command:

```
curl http://localhost:5601/app/kibana
```


If the server was successfully installed and deployed we should see the following HTML code output in the terminal:

![confirm elk server running via localhost](https://github.com/Sk3llington/Project1-UCLA-Cyber-Security/blob/main/Images/confirm_ELK_server_running_localhost.png)

We can also use our web browser to confirm that the ELK server is up and running by opening a web browser page and entering the public ip address to access Kibana's web interface:

http://40.79.255.121:5601/app/kibana

If the server is up and functioning, we should access the page below:

![confirm elk running via public ip](https://github.com/Sk3llington/Project1-UCLA-Cyber-Security/blob/main/Images/confirm_ELK_server_running_public_ip.png)

Next, we want to verify that `filebeat` and `metricbeat` are actually collecting the data they are supposed to and that our deployment is fully functioning.

To do so, we have implemented 3 tasks:

1. Generate a high amount of failed SSH login attempts and verify that Kibana is picking up this activity.

2. Generate a high amount of CPU usage on the pen-testing machines and verify that Kibana picks up this data.

3. Generate a high amount of web requests to your pen-testing servers and make sure that Kibana is picking them up.


* Generating a hight amount of failed SSH login attempts:

To generate these attempts we intentionally tried to connect to our Web-1 web server from the Jump Box instead of connecting from our Ansible container in order to generate failed attempts.
To do so we used the following short script to automate 1000 failed SSH login attempts:

```
for i in {1..1000}; do ssh Web_1@10.0.0.5; done
```

![ssh failed attempts](https://github.com/Sk3llington/Project-1-UCLA-Cyber-Security/blob/f927b7cdbd50c0d4b7830f1839658fcfeaf2a96d/Images/ssh_failed_attempts.png)


Next we check Kibana to see if the failed attempts were logged:

![filebeat failed ssh attempts](https://github.com/Sk3llington/Project-1-UCLA-Cyber-Security/blob/f927b7cdbd50c0d4b7830f1839658fcfeaf2a96d/Images/filebeat_failed_ssh_attempts.png)

We can see all the failed attempts hits were detected and sent to Kibana.

Now Let's breakdown the syntax of our previous short script:

`for` begins the for loop.

`i in` creates a variable named `i` that will hold each number `in` our list.

`{1..1000}` creates a list of 1000 numbers, each of which will be given to our `i` variable.

`;` separates the portions of our `for` loop when written on one line.

`do` indicates the action taken each loop.

`ssh sysadmin@10.0.0.5` is the command run by `do`.

`;` separates the portions of our for loop when it's written on one line.

`done` closes the `for` loop.

Now we can run the same short script with a few modifications, to test that `filebeat` is logging all failed attempts on all web servers where `filebeat` was deployed.

We want to run a command that will attempt to SSH into multiple web servers at the same time and continue forever until we stop it:

```
while true; do for i in {5..7}; do ssh Web_1@10.0.0.$i; done
```

Now let's breakdown the syntax of our previous short script:


`while` begins the while loop.

`true` will always be equal to `true` so this loop will never stop, unless you force quit it.

`;` separates the portions of our `while` loop when it's written on one line.

`do` indicates the action taken each loop.

`i in` creates a variable named i that will hold each number in our list.

`{5..7}` creates a list of numbers (5, 6 and 7), each of which will be given to our `i` variable.

`do` indicates the action taken each loop.

`ssh sysadmin@10.0.0.$i` is the command run by `do`. It is passing in the `$i` variable so the `wget` command will be run on each server, i.e., 10.0.0.5, 10.0.0.6, 10.0.0.7 (Web-1, Web-2, Web-3).

Next, we want to confirm that `metricbeat` is functioning. To do so we will run a linux stress test.

* Generating a high amount of CPU usage on the our web servers (Web-1, Web-2 and Web-3) and confirm that Kibana is collecting the data.

1. From our Jump Box, we start our Ansible container with the follow command:

```bash
sudo docker start hopeful_lalande && sudo docker attach hopeful_lalande
```

2. We SSH from our Ansible container to one of our Web VM.

```bash
ssh Web_1@10.0.0.5
```

3. We install the `stress` module with the following command:

```bash
sudo apt install stress
```

4. We run the service with the following command and let the stress test run for a few minutes:

```bash
sudo stress --cpu 1
```



