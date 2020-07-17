---
layout: single
title:  "NetBox APIs with POSTMAN and Python"
date:   2020-07-16 19:24:15 +0100
categories: netbox
toc: true
toc_label: "On This Post"
---
# 1. Introduction
This is the first post in a series about using NetBox for network management.
In this post, we will:
- **Install NetBox-docker**
- **Exploring NetBox GUI**: how to create devices, platforms, etc.
- **Exploring NetBox APIs with POSTMAN**: we will build a NetBox postman
collections for create site, create/modify/delete devices with POST/PATCH/DELETE
HTTP methods.

## 1.1. What is NetBox
[NetBox](https://netbox.readthedocs.io/en/stable/) is an open source web application designed to help manage and 
document computer networks. It encompasses the following aspects of 
network management:
- IP address management (IPAM) - IP networks and addresses, VRFs, and VLANs
- Equipment racks - Organized by group and site
- Devices - Types of devices and where they are installed
- Connections - Network, console, and power connections among devices
- Virtualization - Virtual machines and clusters
- Data circuits - Long-haul communications circuits and providers
- Secrets - Encrypted storage of sensitive credentials

# 2. Installation
## 2.1. Setup Development Environment

In this post, I use a local Windows 10 workstation and an Ubuntu 20.04
virtual machine that will install NetBox-docker. This VM has a NAT IP
address of `192.168.100.147`.

We need to add the hostname in **Windows 10** host machine at 
`C:\Windows\System32\drivers\etc\hosts`, so that we can access NetBox
GUI from the local Windows 10 workstation.
```bash
192.168.100.147 ubuntu
```

## 2.2. Install NetBox-docker 
We will install Netbox-docker on the Ubuntu 20.04 virtual machine. 
We need to install docker and docker-compose first.

To install docker-compose on the VM, running the following commands.
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

To install netbox-docker, running the following commands.
```bash
git clone -b release https://github.com/netbox-community/netbox-docker.git
cd netbox-docker
tee docker-compose.override.yml <<EOF
version: '3.4'
services:
  nginx:
    ports:
      - 8000:8080
EOF
docker-compose pull
docker-compose up -d
```
For full details around running Netbox-docker, follow the instructions 
[here](https://github.com/netbox-community/netbox-docker).

Once installed, we can access the Netbox GUI on the local Windows 10 
host machine at `http://ubuntu:8000/`.
  - Credential: admin/admin.

# 3. NetBox GUI

# 4. NetBox APIs 
## 4.1. Exploring NetBox APIs with POSTMAN
- Access API docs at `http://ubuntu:8000/api/docs/`.
