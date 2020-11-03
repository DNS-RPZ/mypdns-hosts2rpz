# My Privacy DNS hosts2rpz tool
Block ads and malicious domains with response policy zones.

![GitHub release (latest SemVer)](https://img.shields.io/github/v/release/DNS-RPZ/mypdns-hosts2rpz?sort=semver)
![PyPI](https://img.shields.io/pypi/v/mypdns-hosts2rpz)
![PyPI - Python Version](https://img.shields.io/pypi/pyversions/mypdns-hosts2rpz)
![GitHub commit activity](https://img.shields.io/github/commit-activity/y/DNS-RPZ/mypdns-hosts2rpz)
![GitHub commits since latest release (by SemVer)](https://img.shields.io/github/commits-since/DNS-RPZ/mypdns-hosts2rpz/latest/master?sort=semver)
![GitHub Workflow Status (branch)](https://img.shields.io/github/workflow/status/DNS-RPZ/mypdns-hosts2rpz/CI/master)

From From Wikipedia, the nearly free
[encyclopedia](https://en.wikipedia.org/wiki/Response_policy_zone):

> A response policy zone (RPZ) is a mechanism to introduce a customized 
> policy in Domain Name System servers, so that recursive resolvers 
> return possibly modified results. By modifying a result, access to the 
> corresponding host can be blocked. 

This program allows you to build and maintain RPZ zones from domain 
blocklist feeds. The resulting zones can be used with 
 - [Recursor](https://www.powerdns.com/recursor.html)
 - [PowerDNS](https://www.powerdns.com/auth.html) 
 - [ISC bind](https://www.isc.org/bind/)
 - Other Fully [RPZ](https://www.mypdns.org/w/rpz/) compatible DNS servers.

hosts2rpz is easy to deploy. Just copy it to your PATH. Optionally
write a config file, set up logging, or use a
[cron job](https://en.wikipedia.org/wiki/Cron) to keep your zone fresh.

## Before you Start
Make sure to understand DNS RPZ before using this tool. These sites
provide great documentation:
 - https://www.mypdns.org/w/rpz/
 - [Setup rpzFile in recursor](https://doc.powerdns.com/recursor/lua-config/rpz.html)
 - To benefit properly from this you should setup this as a [auth-zones](https://doc.powerdns.com/recursor/settings.html#auth-zones),
	then you don't need to reload your config after updating you just reload the zone :smiley: >>**Work smarter, not harder**<<
 - [Import zone with BindBackend for redistribution in PDNS](https://doc.powerdns.com/authoritative/backends/bind.html)
 - [Configuring a DNS firewall with RPZ](https://www.zytrax.com/books/dns/ch9/rpz.html)
 - [Response Policy Zone Configuration](https://www.zytrax.com/books/dns/ch7/rpz.html)
 
At minimum, you must create a [new zone clause](https://raw.githubusercontent.com/DNS-RPZ/mypdns-hosts2rpz/version-0.x/test/system/named_zone_centos.conf) 
for RPZ and mention that zone in a [response-policy statement](https://raw.githubusercontent.com/DNS-RPZ/mypdns-hosts2rpz/version-0.x/test/system/named_policy.conf).
 
## How to Install
Run the following as root.
```shell script
# Download hosts2rpz
curl -Ss https://raw.githubusercontent.com/DNS-RPZ/mypdns-hosts2rpz/version-0.x/hosts2rpz.py \
  -o /usr/local/bin/mypdns-hosts2rpz

# Set the executable bit
chmod 755 /usr/local/bin/mypdns-hosts2rpz
```
Alternatively, create a virtualenv and run pip install hosts2rpz.

## Quick Start
```shell script
# View the help screen
hosts2rpz --help

# Write, then review /etc/mypdns-hosts2rpz.ini
hosts2rpz --init

# Optionally set up logging
curl -Ss https://raw.githubusercontent.com/DNS-RPZ/mypdns-hosts2rpz/version-0.x/config/rpz-loggers.ini \
  -o /etc/rpz-loggers.ini

# Download block lists then write an RPZ zone file
hosts2rpz
```
 
## Automate with Ansible
Add the following to your role or playbook.

```yaml
# Customize hosts2rpz.ini and save it under files
- name: upload hosts2rpz.ini
  copy:
    src: files/mypdns-hosts2rpz.ini
    dest: /etc/mypdns-hosts2rpz.ini
    owner: root
    group: root
    mode: 'u=rw,g=r,o=r'

# Customize rpz-loggers.ini and save it under files
- name: upload rpz-loggers.ini
  copy:
    src: files/rpz-loggers.ini
    dest: /etc/rpz-loggers.ini
    owner: root
    group: root
    mode: 'u=rw,g=r,o=r'

# hosts2rpz will be updated to the latest version when force=yes
- name: download hosts2rpz
  get_url:
    url: https://raw.githubusercontent.com/DNS-RPZ/mypdns-hosts2rpz/version-0.x/hosts2rpz.py
    dest: /usr/local/bin/mypdns-hosts2rpz
    force: yes
    owner: root
    group: root
    mode: 'u=rwx,g=rx,o=rx'

# Use a cron job to keep your zone fresh
- name: run hosts2rpz daily
  cron:
    name: hosts2rpz
    special_time: daily
    job: /usr/local/bin/mypdns-hosts2rpz
    user: root
```

## Run Without Root
It is possible to run hosts2rpz without root permissions, though you must
be sure to update all relevant settings pertaining to the user.

For example:
```shell script
# Create an administrator belonging to the named group
useradd -m -G named admin

# Create the user cache directory
mkdir -p /home/admin/.cache

# Run hosts2rpz
hosts2rpz -o rpz.example.com. -z /var/named/rpz.example.com.zone \
  -u admin -g named -d /home/admin/.cache
```

Inspired by [Trellmor/bind-adblock](https://github.com/Trellmor/bind-adblock).
Original Script by [stevekroh/mypdns-hosts2rpz](https://github.com/stevekroh/mypdns-hosts2rpz)
Modified to work with proper DNS servers by [@spirillen](https://www.mypdns.org/p/Spirillen/)
