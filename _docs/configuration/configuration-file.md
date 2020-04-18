---
title: Configuration file
description: How to add configure the wifipumpkin3 
---

## Configuration structure


Wifipumpkin3 uses `.ini` format for configuration file support. ~ is your home directory, usually /home/username. A file or folder name starting with a . is the Linux version of a hidden file/folder. So ~/.config is a hidden folder within your home directory. On ~/.config exist `~/.config/wifipumpkin3` when wp3 is installed. the file structure is like:

```bash
$ ls 
 config  exceptions  helps  logs  scripts
```

{% include alert.html type="info" title="Ini format file"  content="The INI file format is an informal standard for configuration files of computing platforms and software. NI files are simple text files with a basic structure composed of sections, properties, and values." %}

All files `.ini` format is located in `config/app` folder, let's take a look the structure files on this folder.

```bash
.
├── captive-portal.ini
├── config.ini
├── dns_hosts.ini
├── pumpkinproxy.ini
└── sniffkin3.ini

0 directories, 5 files
```
the mean file is `config.ini` that settings all information the access point, plugins,modules, proxy and all variable in CLI interfafce. 

### DNS hosts (Rouge DNS Server)

The hosts file is a file is used as a first point of lookup for DNS hostname resolution. the `wp3` using the imlpementation of python dns resolver created by Samuel Colvin @samuelcolvin. 

the format is very simple. Modifying the hosts dns file involves adding two entries to it. Each entry contains the IP address to which you want the site to resolve and a version of the Internet address. in resume is  rogue DNS server for translates domain names of desirable websites (search engines, banks, brokers, etc.) into IP addresses of sites with unintended content, even malicious websites. 

for example:

```bash
example.com  A       10.0.0.1
example.com  CNAME   whatever.com
example.com  MX      ["whatever.com.", 5]
example.com  MX      ["mx2.whatever.com.", 10]
example.com  MX      ["mx3.whatever.com.", 20]
example.com  NS      ns1.whatever.com.
example.com  NS      ns2.whatever.com.
example.com  TXT     hello this is some text
example.com  SOA     ["ns1.example.com", "dns.example.com"]
```

all request the client to domain example.com will be redirected by localhost 10.0.0.1 if you not change the dhcp range ip address.

### DHCP server Configuration

wp3 have a own dchp implementation like wp2, the dhcp server will be configure only setting on file `config.ini` for now. let take a look on this file.


{% include alert.html type="info" title="DHCP Server" content="A DHCP Server is a network server that automatically provides and assigns IP addresses, default gateways and other network parameters to client devices." %}


```ini
[dhcpdefault]
broadcast=10.0.0.255
leasetimeDef=600
leasetimeMax=7200
netmask=255.0.0.0
range=10.0.0.20/10.0.0.50
router=10.0.0.1
subnet=10.0.0.0
```

so far, this is the default settings dhcp server for network ip class A, you can set for class B or C if you want. in `config.ini`  have two example the class IPAddress type.
