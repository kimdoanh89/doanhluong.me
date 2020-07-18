---
layout: single
title:  "Mega-lab for network automation in GNS3 + Ansible + Python/Jinja2 - Part 2"
date:   2020-07-18 22:48:15 +0100
categories: mega-lab
toc: true
toc_label: "On This Post"
---
## 2. DHCP config with Python/Jinja
### 2.1. Adding more CORE routers to the topology
The topology is as follow:

{% include figure image_path="/assets/02_mega_lab/images/02_more_core_routers.png" %}

- Create the key on all routers `crypto key generate rsa mod 2048`.
- Check the ssh connection from Ubuntu-control-station to each CORE router.
- Check EIGRP on EDGE router `sh ip eigrp nei`.

### 2.2. Install required packages on Ubuntu-control-station

- Install pyenv-installer following this [link](https://github.com/pyenv/pyenv-installer).

```bash
curl https://pyenv.run | bash
pyenv update
```

- Install pyenv prerequisites:

```bash
sudo apt-get install -y build-essential libssl-dev zlib1g-dev libbz2-dev \
libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \
xz-utils tk-dev libffi-dev liblzma-dev python-openssl git
```

- Check python version with `pyenv install -l | grep 3.7`
- Install with `pyenv install 3.7.8`
- Copy this to the .zshrc

```bash
export PATH="/home/doanh/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

- Set global python version to 3.7.8 with `pyenv global 3.7.8`.

- Further install some required packages with pip:

```bash
pip install -U pip setuptools black flake8 bpython bdbpp mypy
```

- Install poetry with

```bash
curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python
```

- Modify the .zshrc as this:

```bash
export PATH="/home/doanh/.pyenv/bin:$HOME/.poetry/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

- Config poetry with

```bash
poetry config virtualenvs.in-project true
poetry config --list
```

- Install jinja with `poetry add jinja2`.

### 2.3. Using Jinja to generate the DHCP config

- The jinja template should be consistent with the dhcp config file
that we created manually.

- The jinja file is as follow:

```liquid
{% raw %}
default-lease-time 600;
max-lease-time 7200;
ddns-update-style none;
option domain-name "lab.doanh";
option domain-name-servers 192.168.134.1;
subnet 192.168.134.0 netmask 255.255.255.0 {
}

{% for i in range(1,1001) -%}
class "R{{i}}" {
    match if (option host-name = "R{{i}}");
}
{% endfor %}

{% for i in range(1,11) -%}
subnet 10.15.{{i}}.0 netmask 255.255.255.0 {
  option subnet-mask 255.255.255.0;
  option routers 10.15.{{i}}.254;
  {% for j in range(1,101) -%}
  pool {
    allow members of "R{{(i-1)*100+j}}";
    range 10.15.{{i}}.{{j}} 10.15.{{i}}.{{j}};
  }
  {% endfor %}
}
{% endfor %}
{% endraw %}
```

- The output `dhcpd.conf` file is beautiful, with around 7000 lines of code
of DHCP configurations for 1000 routers.

- Add a random router R450 to Switch 5 connect to CORE5, bring the router
R450 up. The router is assigned the correct IP address as expected.

```bash
R450#sh ip int br
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                10.15.5.50      YES DHCP   up                    up
```

- One more router R320:

```bash
R320#sh ip int br
Interface                  IP-Address      OK? Method Status                Protocol
Ethernet0/0                10.15.4.20      YES DHCP   up                    up
```