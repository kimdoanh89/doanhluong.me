---
layout: single
title:  "Cisco SD-WAN 20.3.1 setup in GNS3"
date:   2020-08-29 
categories: sd-wan
toc: true
toc_label: "On This Post"
toc_sticky: true
---
In this post, we will go through all the steps of how to configure the CISCO SD-WAN lab in 
GNS3. 

## 1. Lab Topology
The topology in GNS3 is as in the figure below.

{% include figure image_path="/assets/03_SD-WAN/00_Setup/images/00_lab_topology.png" %}

We set up the topology with the following details in mind.

- Networks in host machine, created by VMware:
  - Host-only: VMnet1 - 192.168.134.0/24
  - NAT: VMnet8 - 192.168.100.0/24

### 1.1. Lab Software

- GNS3 version 2.2.12
- VMware® Workstation 15 Pro
- vManage - 20.3.1
- vBond and vEdge - 20.3.1
- vSmart - 20.3.1
- vEdge - 20.3.1

### 1.2. Viptela CLI modes
There are two cli modes in Viptela device software: `viptela-cli` and `vshell`. When you 
login to a Viptela device terminal, you are placed  in the `viptela-cli` mode. To enter 
the `vshell` mode, using the command  `vshell`, and `exit` to return back to `viptela-cli` 
mode.
 
```bash
vmanage# vshell
vmanage:~$ exit
exit
vmanage# 
```

You can find the best sdwan command cheatsheet [here](https://codingpackets.com/blog/cisco-sdwan-command-comparison-cheat-sheet/).

## 2. Configuration
In this lab, we will start by configuring the root CA. The root CA is configured in the vManage
device to simplify the topology. Next we move onto installing certificate on each Viptela 
device, including vManage, vBond, vSmart.
### 2.1. vManage

Certificate Authority in vManage, generate key and certificate. In vManage `vshell` mode:
```bash
openssl genrsa -out SDWAN.key 2048
openssl req -new -x509 -days 2000 -key SDWAN.key -out SDWAN.crt
```

{% include figure image_path="/assets/03_SD-WAN/00_Setup/images/02_controller-certificate-authorization.png" %}

Self-signed the certificate
```bash
openssl req -x509 -new -nodes -key SDWAN.key -sha256 -days 2000 \
        -subj "/C=UK/ST=LD/L=LD/O=SD-WAN-DOANH/CN=SD-WAN" \
        -out SDWAN.pem
ls
cat SDWAN.pem
```

Copy `SDWAN.pem` content to Administration > Settings > Controller Certification 
Authorization > Enterprise Root Certificate.

{% include figure image_path="/assets/03_SD-WAN/00_Setup/images/01_SDWAN-pem.png" %}

{% include figure image_path="/assets/03_SD-WAN/00_Setup/images/02_controller-certificate-authorization-2.png" %}

From the browser, go to 
https://192.168.100.130/dataservice/system/device/sync/rootcertchain to request a 
resync of the vManage database via API call. The answer in JSON format 
should be: `{"syncRootCertChain":"done"}`.
- create Certificate Signing Request (CSR): Configuration > Certificates > Controllers > Generate CSR
- copy the CSR content, go back to vManage `vshell` mode and create vManage.csr
- sign CSR using openssl

```bash
openssl x509 -req -in vManage.csr -CA SDWAN.crt -CAkey SDWAN.key -CAcreateserial -out vManage.crt -days 2000 -sha256
ls
cat vManage.crt
```

- Copy the content of vManage.crt and install the certificate 

### 2.2. Adding vBond controller
In vBond vshell mode, copy the content of SDWAN.crt and SDWAN.key from vManage
```bash
vim SDWAN.crt
vim SDWAN.key
```

- Configuration > Devices > Controllers > Add Controller
- Configuration > Certificates > Controller > vBond > View CSR
  - Copy the content to vBond.csr
- Sign vBond.csr using openssl andd generate vBond.crt

```bash
openssl x509 -req -in vBond.csr -CA SDWAN.crt -CAkey SDWAN.key -CAcreateserial -out vBond.crt -days 2000 -sha256
cat vBond.crt
```

- Copy the content of vBond.crt and install the certificate 

### 2.3. Adding vSmart controller
In vSmart vshell mode, copy the content of SDWAN.crt and SDWAN.key from vManage

```bash
vim SDWAN.crt
vim SDWAN.key
```

- Configuration > Devices > Controllers > Add Controller
- Configuration > Certificates > Controller > vSmart > View CSR
  - Copy the content to vSmart.csr
- Sign vSmart.csr using openssl abd generate vSmart.crt

```bash
openssl x509 -req -in vSmart.csr -CA SDWAN.crt -CAkey SDWAN.key -CAcreateserial -out vSmart.crt -days 2000 -sha256
cat vSmart.crt
```

- Copy the content of vSmart.crt and install the certificate

### 2.4. Enable tunnel-interface

In vManage + vSmart
```bash
vpn 0
 int eth1
  tunnel-interface
```


In vBond
```bash
vpn 0
 int ge0/0
  tunnel-interface
   encapsulation ipsec
```
