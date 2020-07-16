---
layout: posts
title:  "NetBox APIs with POSTMAN and Python"
date:   2020-07-16 19:24:15 +0100
categories: about
---
## 1. Install NetBox-docker in the Ubuntu-control-station
- This Ubuntu-control-station is a Virtual machine, installed in the
Windows 10 host machine.
- Install docker-compose on the VM
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

- Install netbox-docker, follow the instructions [here](https://github.com/netbox-community/netbox-docker).
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

- Add the hostname in **Windows 10** host machine at `C:\Windows\System32\drivers\etc\hosts`
```bash
192.168.134.128 gns3vm
192.168.100.147 ubuntu
```

- Access the Netbox on your Windows 10 host machine at `http://ubuntu:8000/`.
  - Credential: admin/admin.

## 2. Exploring NETBOX APIs with POSTMAN
- Access API docs at `http://ubuntu:8000/api/docs/`.
