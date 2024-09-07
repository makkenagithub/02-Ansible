Configuration management:
Plain-> we make ready serve application/end user -> Manual -> shell script

Disadvantages of shell script:
1. Not idempotent. We write custom code to make it idempotent.
2. Error handling is not good. We need to write code to check error.
3. Homogeneous: If we write shell script in RHEL, it may not work in Ubuntu. So its homogeneous.
4. If we have 1000's of scripts, then we have login to each server and run in all servers. So its not scalable when we have lot of servers
5. Syntax is not easily understandble.

CM (config management) tools: Puppet, chef, rundeck, Ansible ..
Two architectures in CM:
1. Push - Ansible (we do not need to install agent in Nodes)
2. Pull architecture - Puppet, chef, rundeck (We need to install agent in Nodes)

Ansible used SSH protocol to connect to Nodes
```
ssh ec2-user@IP -C "echo 'hello' > /tmp/hello.txt"
```
The above command creates hello.txt in the server with IP. Ansible remote login to nodes and executes the commands.

Earlier Ansible is push based, But recently its implemented pull based also. we can use which ever we want.
The server in which ANsible is installed, its the MASTER server/node. Rest all are managed Nodes.
```
sudo dnf install ansible -y  # installs ansible in the server
```

Inventory:
List of servers Ansible is managing. This called as inventory.

1. We need to check Ansible server is able to reach/connect to Nodes. We need to check network, firewall rules etc.
We can use below command to check master node is reaching Nodes or not
We are in Ansible server (Master) node
```
ansible -i <Node IP>, all -e ansible_user=ec2_user -e ansible_password=DevOps321 -m ping
```
-m is module

Linux/Shell COmmand == Module in Ansible
dnf install nginx -y
cmd  option input

2. Install nginx in Node
```
ansible -i <Node IP>, all -e ansible_user=ec2_user -e ansible_password=DevOps321 -b -m dnf -a "name=nginx state=installed"
```
-b -> become root
-a -> argument

3. Starting nginx:
```
ansible -i <Node IP>, all -e ansible_user=ec2_user -e ansible_password=DevOps321 -b -m service -a "name=nginx state=started"
```
These called as adhoc commands. It is the command issues from ansible server targetting node manually. Basically on some emergency/adhoc purpose

Playbooks: YAML (Yet another markup language)
Playbook is a list of modules, ansible server runs against its node.

YAML:
#works on indentation.

-indicates list

Ansible can connect to any system externally (not only linux, can connect to azure, aws, github etc also) and can perform tasks given by us..

Ansible Module:
Module name is mandatory, we can pass other arguments, which can be optional / mandatory.

Inventory:
A file inventory.ini contains list of servers ansible is connecting to. Here grouping of servers as follows

[backend]
192.2.3.4
192.2.3.5
192.2.3.6
[frontend]
192.2.3.7
192.23.4.6
192.23.45.55
[DB]
192.4.5.6
192.44.55.66

Ansible playbook: is a list of plays, which contains modules that do specific tasks. Create a playbook.yaml file as below
```
---
- name: ping the server
  hosts: backend #list of hosts to ping. backend group is added in inventory.ini
  tasks: # tasks/modules/collections
  - name: ping the servers
    ansible.builtin.ping:
```
The server in which anible is installed is the ansible server (master/control node). Rest other nodes are managed nodes.
Command to run ansible playbook:

```
ansible-playbook -i inventory.ini -e ansible_user=ec2user -e ansible_password=DevOps321 playbook.yaml 
```
We can write multiple plays in a playbook. Generally we write one play in a playbook

Variables:
We can define variables at play level, tasks level. and their usage is with {{ }}
Taks level variables overwrtie play level variables if teh same variable name is used in both taska and plays.

But the general intention is, we do not define variables in play yaml. We define them in seperate files and use **var_files** in playbook

We can pass the variables as arguments also. Below we passed NAME and GREETING varaiables as arguments. -e is for extra variables
```
ansible -i inventory.ini -e ansible_user=ec2_user -e ansible_password=DevOps321 vars_playbook.yaml -e "NAME=Suresh GREETING=morning" 
```
Variable preference sequence:
1. arguments/command line
2. Task level variables
3. Vars from files
4. vars from prompt
5. Play level variables
6. vars from inventory
7. Roles level variables



Conditions:
when condition: when condition is to decide a task/module to run or not

Ansible cant guarentee builtin modules exist for everything. We can run a command by using ansible.builtin.command module

Facts is nothing but variables.
facts == varaibles
Gathering facts:
Ansible before connecting to hosts/servers, it collects entire information, so that it can take decisions based on that information.

Functions:(filters) - We call functions as filters in ansible.
Ansible is giving opportunity to write custom functions. If we want to write function in ansible we have to write in python code and get the functionality. 

ansible.builtin.command vs ansible.builtin.shell :
Command -> simple commands
Shell -> complex commands like pipes, redirections etc. It gets shell environment
command module just issue the command. But shell login to server and execute the command.
Command is secure than Shell. So prefer use command if works.


Ansible is not only a CM tool. It can connect to any system if module is available.

AWS module:
amazon.aws.ec2instance module is available. For this we need to install boto3 and botocore collections in ansible server
```
pip3.9 install boto3 botocore
```

We have to configure AWS in ansible server. Initially create a user say "ansible" in the aws console. And then create access key. It gives access and secret keys

Then run below command in ansible server, which asks for access key, secret key and region
```
aws configure
```
After that create the ec2 instaces using ansible.




