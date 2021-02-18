#Ansible estudos

github.com/ansible-community/toolset

#Docker Compose

docker compose -d up

#Construir um inventário para Ansible

___________________________________
Nome Hosts

Container
ID dos containers

VM
Ips

all:vars
ansible_connection=docker
________________________

ansible -i hosts -m ping all
-i -> qual arquivo de inventário
-m -> module
all -> agir em todas as máquinas, se fosse uma específica uso o id do container ou ip da máquina
______________________________________
edit ansible.cfg
---------------------------------------
[default]
ansible_connection=docker
_______________________________________
new file hosts

[nodes] //definido o grupo
host1 ansible_host=sd54f65s4f65sd //alies
host2 ansible_host=sd54f65s4f65sd //alies

[manager]
host3 ansible_host=sd54f65s4f65sd //alies

[all:vars]
ansible_connection=docker

[manager:vars]
foo=
_________________________________________
file ansible.cfg

[defaults]
inventory = hosts

[inventory]
unparsed_is_failed = true
_________________________________________
criando playbook YML -> playbook.yml

---
- hosts: all
  tasks:
  -name:
    ansible.builtin.ping:

executando o playbook ansible-playbook playbook.yml

--------------
Francílio 






