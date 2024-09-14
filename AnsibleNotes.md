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


Ansible Roles:
Its DRY (do not repeat yourself)
Its a proper structure of playbook, which includes variables, files, templates, other dependencies, handlers etc.
We can reuse the roles.
Directory structure:
1. tasks: we can keep all tasks here
2. handlers: when there is change in particular task, we can notify other tasks
3. templates: we can keep all your files with variables (eg: DB_HOST in backend.service)
4. files: keep files without variables
5. vars: keep all variables
6. defaults: keep low priority variables.
7. meta: other dependencies
8. library: we can wrtie custom modules using python here
9. lookup_plugins: all plugins here


Ansible configuration file:
https://docs.ansible.com/ansible/latest/reference_appendices/config.html
/etc/ansible/ansible.cfg
Changes can be made and used in a configuration file which will be searched for in the following order:
ANSIBLE_CONFIG (environment variable if set)
ansible.cfg (in the current directory)
~/.ansible.cfg (in the home directory)
/etc/ansible/ansible.cfg

Its always better place the ansible config file in current working directory.

jinja2 -> templating language

ansible.builtin.copy vs ansible.builtin.template
We can copy files without variables using copy module.
We can copy files with variables using template module. Refer backend role in 02-expense-ansible-roles git repos

Handlers/notifiers: 
when a task is changed, you want to notify another task. Eg: restart nginx etc
When a task is not changes, we no need to notify and restart

Role dependencies:
we create meta directory in each role and add main.yaml with role dependencies.

Ansible vault:
we keep all the sensitive data like user name, passwords, tokens in ansible vault.

create  vault command. we ned to give a password here. Then it opens the file, we store the password as yaml file
mysql_root_password: ExpenseApp@1
we have to give ask_vault_pass=ture in ansible config file, which exists in current directory.
```
ansible-vault create credentials_mysql.yaml
```
To view (we need to give vault password)
```
ansible-vault view credentials_mysql.yaml
```
To edit (we need to give vault password)
```
ansible-vault edit credentials_mysql.yaml
```

Ansible Tags:
I have 100 taks in my playbook. How to run 10 tasks? We can run with ansible tags.
To run a playbook with specific tags for eg: deployment tag,
```
ansible-playbook backend.yaml -t deployment
```
tags can be applied at tasks level or play level or block level or roles level

ansible-playbook offers five tag-related command-line options:
--tags all - run all tasks, tagged and untagged except if never (default behavior).
--tags tag1,tag2 - run only tasks with either the tag tag1 or the tag tag2 (also those tagged always).
--skip-tags tag3,tag4 - run all tasks except those with either the tag tag3 or the tag tag4 or never.
--tags tagged - run only tasks with at least one tag (never overrides).
--tags untagged - run only tasks with no tags (always overrides).


Ansible dynamic inventory:
inventory.ini is a static file.
When the traffic increases, servers will also increase based on auto scalling. During that to configure the server we use dynamic inventory.
Ansible should query cloud and get the IP addresses dynamically at that time.
We give some query information as below
1. authenticate
2. region
3. name
4. running
5. private IP
   
 we need to use extra plugins for dynamic inventory. For aws its amazon.aws.aws_ec2
File name should be *.aws_ec2.yaml

For example to get dynamic IPs of hosts with filters, prepare a file with demo.aws_ec2.yaml as below
```
---
# dynamic inventory file
# initially 30 servers with ame nginx
plugin: amazon.aws.aws_ec2
regions:
- us-east-1
filters:
  tag:Name: nginx
  instance-state-name: running
hostnames:
  - private-ip-address

compose:
  # This sets the `ansible_host` variable to connect with the private IP address without changing the hostname.
  ansible_host: private-ip-address
```

Once we prepare the *.aws_ec2.yaml, we can test its working or not using below command
```
ansible-inventory -i demo.aws_ec2.yml --graph
```

Once it works we can call the command to install and run ngnix as below
```
ansible-playbook -i demo.aws_ec2.yml 03-nginx.yaml
```

ansible forks:
forks will be in ansible.cfg file. By default its value is 5. It means, ansible takes 5 servers at a time an completes the tasks

forks=5 -> connects to 5 servers at a time and completes the tasks. -> tasks level

serial=3 -> runs playbook in first 3 servers, again it connects to next 3 servers. -> play level

forks: if we set forks = 10 and assume we have 30 servers. we have a play book with 2 tasks. task1 is install nginx and task2 is start nginx. Initiallt it gathers facts for st 10 servers, then gather facts for 2nd 10 servers, then gather facts for next 10 servers. Then It completes task1 in 1st 10 servers, then task1 in 2nd 10 servers, then task1 in 3rd 10 servers. 

Then it completes task2 in 1st 10 servers, then task2 in 2nd 10 servers, then task2 in 3rd 10 servers.

serial: If we set serial as 3 in the playbook. assume we have 30 servers. we have a play book with 2 tasks. task1 is install nginx and task2 is start nginx. Initially it gather facts for 3 servers, Then complete task1 in 3 servers, Then complete task2 in 3 servers. After that again it starts gather facts for next 3 servers, complete task1 , an then complete task2. And so on.. First play completes in 3 servers, and then play goto next 3 servers.

Key based connection to hosts from ansible server:
Initially geneate ssh key , which generates public and private keys. Give public key to aws server while creating aws server.

Now copy the private key to some file with .pem as fle extension in ANSIBLE SERVER. We need to provide this private key file path in ansible.cfg file.

private_key_file=<file path.pem>


