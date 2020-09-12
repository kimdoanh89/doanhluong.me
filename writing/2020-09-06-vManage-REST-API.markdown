---
layout: single
title:  "Exploring vManage REST APIs - Cisco SD-WAN 20.3.1"
date:   2020-09-06
categories: sd-wan
toc: true
toc_label: "On This Post"
toc_sticky: true
---
The vManage REST API library and documentation are bundled with and 
installed on the vManage web application software. To access the API 
documentation from a web browser, use this URL:
```bash
https://ip-address:port/apidocs
```
ip-address is the IP address of the vManage server, and port is the port 
used for the vManage server, could be either 8443 or 8444. 

In our lab, the vManage web server has the URL https://192.168.134.138:8444, we can access the 
vManage REST API at https://192.168.134.138:8444/apidocs.

- Start at 

https://developer.cisco.com/learning/modules/sd-wan
- click setup

https://stackoverflow.com/questions/34643620/how-can-i-split-my-click-commands-each-with-a-set-of-sub-commands-into-multipl/

## MGMT Plane
- Centralized Control Policy: 
  - control how routes are learned and advertised to and from vSmart over the OMP session
  - similar to have a route-map where you can apply the inbound or outbound of a route relector
  to filter and manipulate routes
  - to create different types of topology: hub-spoke
- Centralized Data Policy
  - Per VPN
  - Affect actual data packets directly
  - match the IP packets by IP port protocols, ACLs, and to perform QoS, Classification, 
  Marking
  - Check: ??The configuration is pushed from vManage and stored in the WAN Edge memory, and
  does not get into the device local configuration. Centralized means that the config is not
  actually on the the WAN Edge
- Localized Control Policy
  - Filtering and manipulating all OSPF and EIGRP routes to the local service VPN
- Localized Data Policy
  - Match the pkts and perform some actions
  - Apply per interface

