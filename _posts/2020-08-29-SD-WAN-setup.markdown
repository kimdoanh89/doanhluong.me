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
GNS3. This includes of two main parts: 
- Setting up the Initial Topology for the control plane devices: vManage, vBond, and vSmart.
- Extend the initial lab by adding some more sites.

## 1. Software/hardware requirements and initial topology

### 1.1. Lab Software

- GNS3 version 2.2.12
- VMware® Workstation 15 Pro
- vManage - 20.3.1
- vBond and vEdge - 20.3.1
- vSmart - 20.3.1
- vEdge - 20.3.1

### 1.2. Hardware requirements

The initial topology in GNS3 is as in the figure below.

{% include figure image_path="/assets/03_SD-WAN/00_Setup/images/00_lab_topology.png" %}

We set up the topology with the following details in mind.

- Networks in host machine, created by VMware:
  - Host-only: VMnet1 - 192.168.134.0/24
  - NAT: VMnet8 - 192.168.100.0/24

We need to group the interfaces into 2 different VPNs: VPN 0 for control and VPN 512 for 
management.

| Host    | VPN 512 (mgmt)       | VPN 0 (control)                                   |
|---------|----------------------|---------------------------------------------------|
| vManage | eth0 - 172.16.1.1/24 | eth1 - 10.10.1.1/24;<br>eth2 - 192.168.134.147/24 |
| vSmart  | eth0 - 172.16.1.2/24 | eth1 - 10.10.1.2/24                               |
| vBond   | eth0 - 172.16.1.3/24 | Ge0/0 - 10.10.1.3/24                              |

### 1.3. Viptela CLI modes
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

## 2. Control plane Configuration
In this lab, we will start by configuring the root CA. The root CA is configured in the vManage
device to simplify the topology. Next we move onto installing certificate on each Viptela 
device, including vManage, vBond, vSmart.
### 2.1. vManage

Certificate Authority in vManage, generate key and certificate. In vManage `vshell` mode:
```bash
openssl genrsa -out SDWAN.key 2048
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
openssl x509 -req -in vManage.csr -CA SDWAN.pem -CAkey SDWAN.key -CAcreateserial -out vManage.crt -days 2000 -sha256
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
openssl x509 -req -in vBond.csr -CA SDWAN.pem -CAkey SDWAN.key -CAcreateserial -out vBond.crt -days 2000 -sha256
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
openssl x509 -req -in vSmart.csr -CA SDWAN.pem -CAkey SDWAN.key -CAcreateserial -out vSmart.crt -days 2000 -sha256
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
## 3. Extend initial topology with more sites
### 3.1. Extend topology

### 3.2. Adding Border Router
### 3.3. Upload the WAN Edge list
Go to `Configuration > Devices` and click `Upload WAN Edge List`. After uploading the WAN 
Edge List, you’ll see your devices in `Configuration > Devices`.  You will need to valid 
the edge device by clicking the `Validate` column.
### 3.4. Adding vEdge1 node
We will add vEdge1 of site-id 1 to the topology. The boostrap configuration is as follows:
```bash
system
 host-name               vEdge1
 system-ip               2.2.2.1
 site-id                 1
 admin-tech-on-failure
 no route-consistency-check
 organization-name       SD-WAN-DOANH
 vbond 10.10.1.3
vpn 0
 interface ge0/0
  ip address 172.19.0.11/16
  ipv6 dhcp-client
  tunnel-interface
   encapsulation ipsec
  !
  no shutdown
 !
 interface ge0/1
  ip address 172.18.0.11/16
  no shutdown
 !
 ip route 0.0.0.0/0 172.19.0.1
!
```

We will need to copy the content of SDWAN.pem from vManage to vEdge1. In vEdge1, paste the
content of SDWAN.pem to a new empty file.
```bash
vEdge1# vshell
vEdge1:~$ vim SDWAN.pem
vEdge1:~$ exit         
vEdge1# 
```
Now, we can import the certificate.
```bash
request root-cert-chain install /home/admin/SDWAN.pem
```

Go to vManage interface, `Configuration > Devices > Select unused entry > ... > Generate Bootstrap
Configuration`.

{% include figure image_path="/assets/03_SD-WAN/00_Setup/images/05_generate_bootstrap_configuration.png" %}

```bash
request vedge-cloud activate chassis-number 26e25eef-2ec0-94e4-5b6e-d3512f8ca2fb token 5726ba8c152b416eb804be6ba150cf30
```

Check with `show control local-properties`
### 3.5. Adding vEdge3 node
```bash
request vedge-cloud activate chassis-number 5997295d-c718-3109-6277-08b4caea2bcf token 764fa250066c4e90bb994ce60994bf90
```

{% include figure image_path="/assets/03_SD-WAN/00_Setup/images/06_registered.png" %}
