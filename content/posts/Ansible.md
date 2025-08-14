---
title: "Ansible"
date: 2025-07-07T13:10:56Z
draft: true
---

Ansible is a great bit of software that can streamline and remove human error in deploying and managing a large network. It automates the management of remote systems and controls their desired state.

![Overview of Ansible Components](https://docs.ansible.com/ansible/latest/_images/ansible_inv_start.svg)


- The Control node is where Ansible is installed. This is where the network inventory is created and stored.
- The Managed node(s) are the remote managed systems

I have deployed Ansible on a server that can reach my Arista nodes' mgmt network:


![Arista Ansible Lab](/Ansible_diagram.png)

Ansible obviously needs mgmt connectivity to the devices, it should fit in nicely along other network management resources which <b>of course are firewalled with 5-tuple rules, zoned approapriatly alongside same-trust-level resources, and only have strictly-controlled web-proxy-based outbound access to the Internet (if at all).</b>

My root Ansible directory contains a config file and 3 folders:

    drwxr-xr-x   5 root root  4096 Jul  3 11:05 .
    drwxr-xr-x 101 root root  4096 Jul  7 13:44 ..
    -rw-r--r--   1 root root 40643 Jul  3 09:25 ansible.cfg
    drwxr-xr-x   3 root root  4096 Jul  7 13:44 inventory
    drwxr-xr-x   2 root root  4096 Jul  3 11:21 playbooks
    drwxr-xr-x   2 root root  4096 Jul  3 11:05 templates
<br>

- Inventory - this is where you define and organise your remote systems 
- Playbooks - playbooks are files that contain instructions and tasks to acheive the device management and configuration
- Templates - if using JinJa2 in playbooks, this is where the device configuration templates are stored ready for access by the playbook

<h2>Inventory</h2>

The inventory can be organised in various different ways; I organise by device vendor and/or CLI language so I can easily apply playbooks to the whole vendor estate

    root@mgmt-host:/etc/ansible/inventory# ls
    arista

    root@mgmt-host:/etc/ansible/inventory# cd arista/
    root@mgmt-host:/etc/ansible/inventory/arista# ls
    hosts.yaml  host_vars
<br>

Hosts.yaml is the inventory file where the devices are listed and organised into groups. "All" contains all the devices in this file, but you can use "children" to organise into arbitrary groups

    root@mgmt-host:/etc/ansible/inventory/arista# cat hosts.yaml
    all:
    hosts:
        leaf1:
        leaf2:
        spine1:
    children:
        leafs:
        leaf1:
        leaf2:
        spines:
        spine1:
    vars:
        ansible_connection: ansible.netcommon.network_cli
        ansible_network_os: arista.eos.eos
<br>

"vars" contains variables common to the specific groups; here I've listed vars under "all" to apply to all the hosts in the inventory file.


    root@mgmt-host:/etc/ansible/inventory/arista# cd host_vars/
    root@mgmt-host:/etc/ansible/inventory/arista/host_vars# ls
    leaf1.yaml  leaf2.yaml  spine1.yaml
<br>

"host-vars" contains files that allow arbitrary per-device variables; here I've defined hostname and vtep_ip. These can be accessed in the Jinja2 templating module

    root@mgmt-host:/etc/ansible/inventory/arista/host_vars# cat leaf1.yaml
    hostname: leaf1
    vtep_ip: 10.0.0.1
<br>

<h2>Playbooks</h2>

Playbooks are lists of tasks and actions to perform on the remote devices. They are YAML files, and can have 1 or many tasks.

Ansible content collections can add content not included in the Ansible "core". They are a way to add vendor or other content without building it directly into Ansible.

The Arista EOS collection is called "arista.eos" - there are many modules in the collection, including an arbitrary CLI and HTTP API module.

[Ansible Arista EOS Collection](https://docs.ansible.com/ansible/latest/collections/arista/eos/index.html)

See an example of a Playbook below, updating all the management interface descriptions:

    ---

    - name: Configure interface descriptions
    hosts: all
    gather_facts: false

    tasks:

        - name: Add interface desc
        arista.eos.eos_interfaces:
            config:
            - name: Management1
                description: "configured by Ansible"
            state: merged
        become: true
<br>

When the playbook is run against the "arista" inventory, you get an output based of the Playbook's success

    root@mgmt-host:/etc/ansible# ansible-playbook playbooks/pb_interface_desc.yaml -i inventory/arista  -uadmin

    PLAY [Configure interface descriptions] ***********************************************************************************************************

    TASK [Add interface desc] *************************************************************************************************************************
    changed: [leaf1]
    changed: [spine1]
    changed: [leaf2]

    PLAY RECAP ****************************************************************************************************************************************
    leaf1                      : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    leaf2                      : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
    spine1                     : ok=1    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

    root@mgmt-host:/etc/ansible#


<h2>Summary</h2>

There's lots to unpack, but this is what's meant as "infrastructure as code"; an entire network can be managed and maintained purely by building inventories and playbooks. Once you've got to grips with host_vars, the size of the network almost becomes irrelevant.