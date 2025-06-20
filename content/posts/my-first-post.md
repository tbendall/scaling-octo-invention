---
title: "My First Post"
date: 2025-06-03T13:41:12Z
draft: false
---

# Overview

I wanted to set up a site that demonstrated my foray into DevOps skills, so as a project in and of itself, I've created this platform as a challenge and demonstration of my infrastructure skills.

![Diagram of initial DevOps setup](/diagram.png)


* Hosted on a Free SKU Azure VM
* Uses GitHub for basic Version Controlling
* Uses my bendall.co domain to create a Dynamic DNS entry for "devops.bendall.co" using ddclient package
* VM is protected with a NSG to lockdown access for mgmt and web
* VM is secured with non-standard SSH port and public/private keys enabling secure and passwordless login
*
* NGINX is used as a reverse proxy for Hugo easy site builder (as I'm terrible at front-end)
* Let's Encrypt TLS Certificate used instead of self-signed
*
* VSCode with Remote-SSH used for remote development; let's face it, no-one should be coding in a terminal these days