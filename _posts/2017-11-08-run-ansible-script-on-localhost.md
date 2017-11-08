---
layout: default
title:  "How to run Ansible script on localhost"
date:   2017-11-08 16:00:00 +1100
categories: blog
---

# How to run Ansible script on localhost

Ansible is a fantastic automation/DevOps tool for automating a client server configuration management. Initalizating and developing your Ansible role locally, you can deploy and re-use it in different client servers in cloud. 

In a typical cloud typology, an Ansible master server run ansible playbook to spin or revision few client servers. But such master server is not mandatory. You can run ansible-playbook or ansible command against its own server via localhost.

#### Install Ansible package
```
# sudo pip install ansible
```

#### Configure Ansible hosts file
```
# vim /etc/ansible/hosts

[all:vars]
ansible_connection=ssh

[local:vars]
ansible_connection=local

[local]
127.0.0.1

```

#### Test the setup via ansible command:
```
# ansible local -m shell -a "ifconfig"
```

#### Setup you ansible playbook start with:
```
# vim testplaybook.yml

---
- hosts: local
  tasks:
    - shell: echo "hello world"
```

#### Test above playbook via:
```
#  ansible-playbook testplaybook.yml -vv
```

