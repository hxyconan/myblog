---
layout: default
title:  "Programmatically call Ansible playbooks from your Python application"
date:   2017-11-22 15:30:00 +1100
categories: blog
---

# Programmatically call Ansible playbooks from your Python application

Ansible playbook is a well organized YMAL files for tracking all your configuration meta data. You can spin up your virtual machine via Ansible cli such as:
```
# ansible-playbook --inventory-file=hosts your-playbook.yml -v --extra-vars 'para1=value1 para2=value2'
```
Jenkins could call it via execute command in bash command field. But what about calling directly from an application codes. 

Python having a bunch of libraries allows running CLI commands such as: os.system(self.your-command), but this which always feels dirty. 


You will have more controls when calling the Ansible API programmatically...
... plus control host file in flexible.



# References


# TODO
- Bacic introduction of context and why
- Version of all required parts
- Draw a diagram of how each part connect to each other
- Refer to the reference links see if I can get some ideas
    + https://serversforhackers.com/c/running-ansible-programmatically
    + https://serversforhackers.com/c/running-ansible-2-programmatically
    + http://docs.ansible.com/ansible/latest/dev_guide/developing_api.html
    + Also the Ansible Python codes in github

