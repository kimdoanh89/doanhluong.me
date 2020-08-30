---
layout: single
title:  "Cisco SD-WAN 20.3.1 setup in GNS3"
date:   2020-08-29 
categories: sd-wan
toc: true
toc_label: "On This Post"
toc_sticky: true
---
## 1. Introduction

## 2. Lab Topology
### 2.1. Lab IP addressing
### 2.2. Lab Software
### 2.3. CLI modes
**Viptela-cli**

**vshell**

## 3. Configuration
### 3.1. vManage
organization name

{% include figure image_path="/assets/03_SD-WAN/00_Setup/images/00_organization_name.png" %}

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
- copy `SDWAN.pem` content to Administration > Settings > Controller Certification Authorization > Enterprise Root Certificate

{% include figure image_path="/assets/03_SD-WAN/00_Setup/images/01_SDWAN-pem.png" %}

{% include figure image_path="/assets/03_SD-WAN/00_Setup/images/02_controller-certificate-authorization-2.png" %}

- go to https://192.168.100.130/dataservice/system/device/sync/rootcertchain to to request a resync of the vManage 
database via API call. The answer in JSON format should be: `{"syncRootCertChain":"done"}`.
- create Certificate Signing Request (CSR): Configuration > Certificates > Controllers > Generate CSR
- copy the CSR content, go back to vManage `vshell` mode and create vManage.csr
- sign CSR using openssl
openssl x509 -req -in vManage.csr -CA SDWAN.crt -CAkey SDWAN.key -CAcreateserial -out vManage.crt -days 2000 -sha256
ls
cat vManage.crt
- Copy the content of vManage.crt and install the certificate 

---------------- Adding vBond controller------------------------------
- in vBond vshell mode, copy the content of SDWAN.crt and SDWAN.key from vManage
vim SDWAN.crt
vim SDWAN.key
- Configuration > Devices > Controllers > Add Controller
- Configuration > Certificates > Controller > vBond > View CSR
- Copy the content to vBond.csr
- Sign vBond.csr using openssl andd generate vBond.crt
openssl x509 -req -in vBond.csr -CA SDWAN.crt -CAkey SDWAN.key -CAcreateserial -out vBond.crt -days 2000 -sha256
cat vSmart.crt
- Copy the content of vSmart.crt and install the certificate 

---------------- Adding vSmart controller------------------------------
- in vSmart vshell mode, copy the content of SDWAN.crt and SDWAN.key from vManage
vim SDWAN.crt
vim SDWAN.key
- Configuration > Devices > Controllers > Add Controller
- Configuration > Certificates > Controller > vSmart > View CSR
- Copy the content to vSmart.csr
- Sign vSmart.csr using openssl abd generate vSmart.crt
openssl x509 -req -in vSmart.csr -CA SDWAN.crt -CAkey SDWAN.key -CAcreateserial -out vSmart.crt -days 2000 -sha256
cat vSmart.crt
- Copy the content of vSmart.crt and install the certificate
-----------------Enable tunnel-interface-----------------
- vManage + vSmart
vpn 0
 int eth1
  tunnel-interface

- vBond
vpn 0
 int ge0/0
  tunnel-interface
   encapsulation ipsec