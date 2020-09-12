---
layout: single
title:  "Exploring OMP and configure VPN Advertise OMP"
date:   2020-09-12 
categories: sd-wan
toc: true
toc_label: "On This Post"
toc_sticky: true
---
In the previous post, we have spin up the CISCO SD-WAN controllers (vManage, vBond, and vSmart)
in GNS3. We also managed to onboard two WAN Edge routers: vEdge1 at site-id 1 and vEdge3 at
site-id 3. However, we did not configure the data plane. So the objectives of this lab are as 
follows:
- Figure out the data plane configuration, how it works, how to set TLOC, color, etc...
- Understand the BFD connections, monitor and troubleshoot
- SDWAN Routing
- OMP Control 

## 1. New Topology
So in this post, we will extend our previous topology by adding vEdge-2 at site-id 2. The 
extended topology is as in the figure below.

{% include figure image_path="/assets/03_SD-WAN/02_one_more_site/images/01_topology.png" %}

### Bootstrap configuration

First, we spin up vEdge2 in GNS3. During this step, we can set up the user and password to 
log into vEdge2 (`admin/admin`) and configure the boostrap configuration in configuration mode
`conf t`.

```bash
system
 host-name               vEdge2
 system-ip               2.2.2.2
 site-id                 2
 organization-name       SD-WAN-DOANH
 vbond 10.10.1.3
vpn 0
 interface ge0/0
  ip address 172.19.0.21/16
  ipv6 dhcp-client
  tunnel-interface
   encapsulation ipsec
  !
  no shutdown
 !
 interface ge0/1
  ip address 172.18.0.21/16
  no shutdown
 !
 ip route 0.0.0.0/0 172.19.0.1
!
```
### Install the certificate
We will need to copy the content of SDWAN.pem from vManage to vEdge2. In vEdge2, 
go to `vshell` mode, create an empty file with `vim SDWAN.pem`, then paste the copied content,
`exit` to return back to the `viptela-cli` mode.

```bash
vEdge2# vshell
vEdge2:~$ vim SDWAN.pem
vEdge2:~$ exit         
vEdge2# 
```

Now, we can import the certificate.
```bash
request root-cert-chain install /home/admin/SDWAN.pem
```

Go to vManage interface, `Configuration > Devices > Select unused entry > ... > Generate Bootstrap
Configuration`, to see the boostrap information, what we need is the UUID and token to be used
in the next command. Note that, since we use `vEdge`, the `unused entry` we select needs to 
be of model `vEdge Cloud`.

```bash
request vedge-cloud activate chassis-number uuid token otp
```
One example is as follows:
```bash
request vedge-cloud activate chassis-number 122028eb-cc55-f3bf-b8ed-fa38fcb5fa2c token ee20ed20da914acdad6d79974cd3b85e
```

## 2. OMP
### 2.1. OMP Fundamentals
### 2.2. Lab validation
#### OMP peers
In vSmart
```bash
sh omp peers
```

```bash
                         DOMAIN    OVERLAY   SITE                                
PEER             TYPE    ID        ID        ID        STATE    UPTIME           R/I/S  
------------------------------------------------------------------------------------------
2.2.2.1          vedge   1         1         1         up       0:07:05:11       0/0/0
2.2.2.2          vedge   1         1         2         up       0:06:40:52       0/0/0
2.2.2.3          vedge   1         1         3         up       0:07:04:22       0/0/0
```

In vEdge1
```bash
sh omp peers
```
```bash
                         DOMAIN    OVERLAY   SITE                                
PEER             TYPE    ID        ID        ID        STATE    UPTIME           R/I/S  
------------------------------------------------------------------------------------------
1.1.1.2          vsmart  1         1         1000      up       0:07:06:48       0/0/0
```

#### TLOC-paths & TLOCS
In vEdge1
```bash
sh omp tloc-paths
```

```bash
tloc-paths entries 2.2.2.1 default ipsec
tloc-paths entries 2.2.2.2 default ipsec
tloc-paths entries 2.2.2.3 default ipsec
```

tlocs
```bash
sh omp tlocs
```

```bash
---------------------------------------------------
tloc entries for 2.2.2.2
                 default
                 ipsec
---------------------------------------------------
            RECEIVED FROM:                   
peer            1.1.1.2
status          C,I,R
loss-reason     not set
lost-to-peer    not set
lost-to-path-id not set
    Attributes:
     attribute-type    installed
     encap-key         not set
     encap-proto       0
     encap-spi         256
     encap-auth        sha1-hmac,ah-sha1-hmac
     encap-encrypt     aes256
     public-ip         172.19.0.21
     public-port       12346
     private-ip        172.19.0.21
     private-port      12346
     public-ip         ::
     public-port       0
     private-ip        ::
     private-port      0
     bfd-status        up
     domain-id         not set
     site-id           2
     overlay-id        not set
     preference        0
     tag               not set
     stale             not set
     weight            1
     version           3
    gen-id             0x80000000
     carrier           default
     restrict          0
     on-demand          0
     groups            [ 0 ]
     bandwidth         0
     qos-group         default-group
     border             not set
     unknown-attr-len  not set

```

From the above table, some information that we need to notice:
- tloc = `system ip + color + encapsulation`. In this case, tloc = `2.2.2.2 + default + ipsec`
- peer: 1.1.1.2 means that it receives this tloc information from the vSmart 1.1.1.2
- public-ip
- private-ip: if behind the NAT, should be different from public-ip
- bfd-status: up
- site-id: 2

#### BFD sessions and summary
```bash
sh bfd sessions
sh bfd summary
```

#### IPsec connections
```bash
sh ipsec outbound-connections
```

## 3. Route advertising
Two types of interfaces: transport interface and service interface
- Transport interface
  - which is facing the transport side of the WAN Edge
- Service interface
  - which is facing the service side of the WAN Edge
  - point to the local site network. These networks need to be transported across the overlay,
  and have the subnets advertise.

### 3.1. Create VPN Template  
We need to configure the route using the templates for `vEdge Cloud, vEdge 1000`.
Go to `Configuration > Templates > Feature Template > Add Template > VPN` to create a VPN
feature template with Template name `VPN10_WEDGE` and description `VPN10_WEDGE`.

{% include figure image_path="/assets/03_SD-WAN/02_one_more_site/images/02_VPN_template_name.png" %}

VPN Basic Configuration

{% include figure image_path="/assets/03_SD-WAN/02_one_more_site/images/02_VPN_basic.png" %}

VPN Advertise OMP

{% include figure image_path="/assets/03_SD-WAN/02_one_more_site/images/02_VPN_advertise_OMP.png" %}

VPN Ipv4 Route GW
{% include figure image_path="/assets/03_SD-WAN/02_one_more_site/images/02_VPN_IPv4Route_GW.png" %}

VPN IPv4 Route Nexthop
{% include figure image_path="/assets/03_SD-WAN/02_one_more_site/images/02_VPN_IPv4Route_NextHop.png" %}

Select `Mark as Optional Row` and `Add`.