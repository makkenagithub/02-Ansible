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
$ssh ec2-user@IP -C "echo 'hello' > /tmp/hello.txt"
The above command creates hello.txt in the server with IP. Ansible remote login to nodes and executes the commands.

Earlier Ansible is push based, But recently its implemented pull based also. we can use which ever we want.
The server in which ANsible is installed, its the MASTER server/node. Rest all are Nodes.

Inventory:
List of servers Ansible is managing. This called as inventory.

1. We need to check Ansible server is able to reach/connect to Nodes. We need to check network, firewall rules etc.
We can use below command to check master node is reaching Nodes or not
We are in Ansible server (Master) node
```
$ansible -i <Node IP>, all -e ansible_user=ec2_user -e ansible_password=DevOps321 -m ping
```
-m is module

Linux/Shell COmmand == Module in Ansible
dnf install nginx -y
cmd  option input

2. Install nginx in Node
```
$$ansible -i <Node IP>, all -e ansible_user=ec2_user -e ansible_password=DevOps321 -b -m dnf -a "name=nginx state=installed"
```
-b -> become root
-a -> argument

3. Starting nginx:
```
$$ansible -i <Node IP>, all -e ansible_user=ec2_user -e ansible_password=DevOps321 -b -m service -a "name=nginx state=started"
```
These called as adhoc commands. It is the command issues from ansible server targetting node manually. Basically on some emergency/adhoc purpose

Playbooks: YAML (Yet another markup language)
Playbook is a list of modules, ansible server runs against its node.



