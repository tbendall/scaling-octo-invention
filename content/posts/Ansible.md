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

Ansible obviously needs mgmt connectivity to the devices, it should fit in nicely along other network management resources which of course are <u>firewalled with 5-tuple rules, zoned approapriatly alongside same-trust-level resources, and only have strictly-controlled web-proxy-based outbound access to the Internet (if at all)</u>