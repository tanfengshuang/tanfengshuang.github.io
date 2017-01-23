---
layout: post
title:  "Installing Central Authentication(413-11)"
categories: Linux
tags: 413 
---

### Installing an Identify Management(IdM) Server

Red Hat Identify Management(IdM) is a solution to centrally manage authentication and authorization policies from a Linux server for enrolled Linux clients using native Linux tools.

server package: ipa-server
client package: ipa-client ipa-admintools

###### Demonstration: Installing an Identify Management(IdM) Server

```
1. DNS NTP IPTABLES
2. RHEL6.3(idM 3.0)
3. NetworkManager off
# chkconfig NetworkManager off
# service NetworkManager stop

4. ifcfg-eth0 static no ip netmask dns

5. install ipa-server
# yum install -y ipa-server
```

###### Performance Checklist: Set Up an IdM Server

### Adding User and Group Entries to Identify Management(IdM)


### Registering a Client System with Identify Management(IdM)
