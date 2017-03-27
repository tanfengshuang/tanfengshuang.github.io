---
layout: post
title:  "Error initializing network controller"
categories: Docker
tags: Docker
---

```
# systemctl start docker
Job for docker.service failed because the control process exited with error code. See "systemctl status docker.service" and "journalctl -xe" for details.

# systemctl status docker.service
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Mon 2017-03-13 10:16:28 CST; 8s ago
     Docs: http://docs.docker.com
  Process: 7778 ExecStart=/usr/bin/docker-current daemon --exec-opt native.cgroupdriver=systemd $OPTIONS $DOCKER_STORAGE_OPTIONS $DOCKER_NETWORK_OPTIONS $INSECURE_REGISTRY (code=exited,
 Main PID: 7778 (code=exited, status=1/FAILURE)

Mar 13 10:16:27 dhcp-129-74.nay.redhat.com docker-current[7778]: time="2017-03-13T10:16:27.700000152+08:00" level=warning msg="devmapper: Usage of loopback devices is strongly discourag
Mar 13 10:16:27 dhcp-129-74.nay.redhat.com docker-current[7778]: time="2017-03-13T10:16:27.727579085+08:00" level=warning msg="devmapper: Base device already exists and has filesystem x
Mar 13 10:16:27 dhcp-129-74.nay.redhat.com docker-current[7778]: time="2017-03-13T10:16:27.764944821+08:00" level=info msg="[graphdriver] using prior storage driver \"devicemapper\""
Mar 13 10:16:27 dhcp-129-74.nay.redhat.com docker-current[7778]: time="2017-03-13T10:16:27.768008225+08:00" level=info msg="Graph migration to content-addressability took 0.00 seconds"
Mar 13 10:16:27 dhcp-129-74.nay.redhat.com docker-current[7778]: time="2017-03-13T10:16:27.775886532+08:00" level=info msg="Firewalld running: false"
Mar 13 10:16:28 dhcp-129-74.nay.redhat.com docker-current[7778]: time="2017-03-13T10:16:28.155347349+08:00" level=fatal msg="Error starting daemon: Error initializing network controller
Mar 13 10:16:28 dhcp-129-74.nay.redhat.com systemd[1]: docker.service: Main process exited, code=exited, status=1/FAILURE
Mar 13 10:16:28 dhcp-129-74.nay.redhat.com systemd[1]: Failed to start Docker Application Container Engine.
Mar 13 10:16:28 dhcp-129-74.nay.redhat.com systemd[1]: docker.service: Unit entered failed state.
Mar 13 10:16:28 dhcp-129-74.nay.redhat.com systemd[1]: docker.service: Failed with result 'exit-code'.

# ls /var/lib/docker/network/files/*
/var/lib/docker/network/files/545605073c274f406acb8c93b80f44ec2b9d03fb68c4cf6cb9f7355fa153d9c5.sock
/var/lib/docker/network/files/7bbbf817406fb7b93763883cf1c337c74431385fd4b66135a3af3e57d8c6ab67.sock
/var/lib/docker/network/files/e2c5b5725371ccdb5c60ada668fe8f66dcb16e5e41378aa55b40f3724b929150.sock
/var/lib/docker/network/files/ecf9d922b336475fc405df06d971605c047db775a8dc5a3d6fe726d92fb7d1ca.sock
/var/lib/docker/network/files/f769869bc3485934855ec8069bdb6d9fecb9936b480138287fcedecef66284cb.sock
/var/lib/docker/network/files/local-kv.db

# rm /var/lib/docker/network/files/* -rf

# systemctl start docker
# systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2017-03-13 10:19:27 CST; 4s ago
     Docs: http://docs.docker.com
 Main PID: 7965 (docker-current)
    Tasks: 14
   CGroup: /system.slice/docker.service
           └─7965 /usr/bin/docker-current daemon --exec-opt native.cgroupdriver=systemd --selinux-enabled --log-driver=journald

Mar 13 10:19:26 dhcp-129-74.nay.redhat.com docker-current[7965]: time="2017-03-13T10:19:26.870912643+08:00" level=info msg="Graph migration to content-addressability took 0.00 seconds"
Mar 13 10:19:26 dhcp-129-74.nay.redhat.com docker-current[7965]: time="2017-03-13T10:19:26.877990153+08:00" level=info msg="Firewalld running: false"
Mar 13 10:19:27 dhcp-129-74.nay.redhat.com docker-current[7965]: time="2017-03-13T10:19:27.081834928+08:00" level=info msg="Default bridge (docker0) is assigned with an IP address 172.1
Mar 13 10:19:27 dhcp-129-74.nay.redhat.com docker-current[7965]: time="2017-03-13T10:19:27.788677835+08:00" level=info msg="Loading containers: start."
Mar 13 10:19:27 dhcp-129-74.nay.redhat.com docker-current[7965]: ..time="2017-03-13T10:19:27.819476625+08:00" level=error msg="Error unmounting container f48bde61c9c0a07b369d740132ec2d4
Mar 13 10:19:27 dhcp-129-74.nay.redhat.com docker-current[7965]: time="2017-03-13T10:19:27.825776944+08:00" level=info msg="Loading containers: done."
Mar 13 10:19:27 dhcp-129-74.nay.redhat.com docker-current[7965]: time="2017-03-13T10:19:27.825804247+08:00" level=info msg="Daemon has completed initialization"
Mar 13 10:19:27 dhcp-129-74.nay.redhat.com docker-current[7965]: time="2017-03-13T10:19:27.825839086+08:00" level=info msg="Docker daemon" commit="e03ddb8/1.10.3" execdriver=native-0.2 
Mar 13 10:19:27 dhcp-129-74.nay.redhat.com systemd[1]: Started Docker Application Container Engine.
Mar 13 10:19:27 dhcp-129-74.nay.redhat.com docker-current[7965]: time="2017-03-13T10:19:27.834751215+08:00" level=info msg="API listen on /var/run/docker.sock"


```
