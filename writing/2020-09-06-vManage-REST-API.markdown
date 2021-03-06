---
layout: single
title:  "vManage REST APIs - Cisco SD-WAN 20.3.1 and building the SDWANCLI"
date:   2020-09-06
categories: sd-wan
toc: true
toc_label: "On This Post"
toc_sticky: true
---
In this post, we will explore the vManage REST APIs, and use python `requests` module to 
interact with vManage for extracting some useful information, including list of devices in
our SD-WAN system, list of device templates. We can also make changes to the configuration
of SLA Class, template using python. Finally, we make the python scripts as an `sdwancli` by
using python `click` module. This `sdwancli` is published in CiscoDevnet CodeExchange.

[![published](https://static.production.devnetcloud.com/codeexchange/assets/images/devnet-published.svg)](https://developer.cisco.com/codeexchange/github/repo/kimdoanh89/sdwan-python-rest-api)

## 1. vManage REST APIs basics
There are some excellent materials from Cisco DevNet that can help you get started with
the vManage REST APIs. I highly recommend to look at these resources first.

https://developer.cisco.com/learning/modules/sd-wan

The vManage REST API library and documentation are bundled with and installed on the vManage 
web application software. To access the API documentation from a web browser, use this URL:
```bash
https://ip-address:port/apidocs
```
ip-address is the IP address of the vManage server, and port is the port used for the vManage 
server, could be either 8443 or 8444. 

Since I reinstalled the SD-WAN lab on the new host machine, the vManage server now has
the ip-address of 192.168.148.129 (changed from 192.168.134.138 on the old machine).

In our lab, the vManage web server has the URL https://192.168.148.129:8444. We can access the 
vManage REST API at https://192.168.148.129:8444/apidocs.

## 2. Click setup

Our project has the following structure:
```bash
├── LICENSE
├── poetry.lock
├── pyproject.toml
├── README.md
├── requirements.txt
├── sdwancli.py
├── sdwan-test.py
└── vmanage
    ├── authenticate.py
    ├── bfd.py
    ├── constants.py
    ├── device.py
    ├── ipsec.py
    ├── omp.py
    ├── sla.py
    ├── template.py
    └── utils.py
```
Our main 

```python
@click.group()
def cli():
    """Command line tool to interact with CISCO SDWAN vManage.
    """
    pass


cli.add_command(cli_device)
cli.add_command(cli_template)
cli.add_command(cli_bfd)
cli.add_command(cli_sla)
cli.add_command(cli_omp)
cli.add_command(cli_ipsec)
```

Each command have subcommand
```python
@click.group(name="sla")
def cli_sla():
    """Commands for managing SLA Class: list, create, edit, delete
    """
    pass


@cli_sla.command(name="list", help="Get list of SLA Classes")
@cli_sla.command(name="create", help="Create a SLA Class")
@click.option("--name", help="name of the SLA Class")
@click.option("--description", help="description of the SLA Class")
@click.option("--loss", help="loss 0 - 100 %")
@click.option("--latency", help="latency 1 - 1000 ms")
@click.option("--jitter", help="jitter 1 - 1000 ms")
```

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

