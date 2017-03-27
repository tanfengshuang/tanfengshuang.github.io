---
layout: post
title:  "Docker Quick Start"
categories: Docker
tags: Docker
---

```
# yum install docker
# # docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 1.10.3
Storage Driver: devicemapper
 Pool Name: docker-253:0-1451155-pool
 Pool Blocksize: 65.54 kB
 Base Device Size: 10.74 GB
 Backing Filesystem: xfs
 Data file: /dev/loop0
 Metadata file: /dev/loop1
 Data Space Used: 11.8 MB
 Data Space Total: 107.4 GB
 Data Space Available: 32.61 GB
 Metadata Space Used: 581.6 kB
 Metadata Space Total: 2.147 GB
 Metadata Space Available: 2.147 GB
 Udev Sync Supported: true
 Deferred Removal Enabled: false
 Deferred Deletion Enabled: false
 Deferred Deleted Device Count: 0
 Data loop file: /var/lib/docker/devicemapper/devicemapper/data
 WARNING: Usage of loopback devices is strongly discouraged for production use. Either use `--storage-opt dm.thinpooldev` or use `--storage-opt dm.no_warn_on_loop_devices=true` to suppress this warning.
 Metadata loop file: /var/lib/docker/devicemapper/devicemapper/metadata
 Library Version: 1.02.122 (2016-04-09)
Execution Driver: native-0.2
Logging Driver: journald
Plugins: 
 Volume: local
 Network: null host bridge
Kernel Version: 4.5.5-300.fc24.x86_64
Operating System: Fedora 24 (Workstation Edition)
OSType: linux
Architecture: x86_64
Number of Docker Hooks: 2
CPUs: 8
Total Memory: 7.756 GiB
Name: dhcp-129-74.nay.redhat.com
ID: XH2Z:GRM6:A5FT:Q6HT:6OWN:JREZ:NONN:MSA2:YSJU:U327:5WPL:YCFP
Registries: docker.io (secure)

# docker version
Client:
 Version:         1.10.3
 API version:     1.22
 Package version: docker-common-1.10.3-55.gite03ddb8.fc24.x86_64
 Go version:      go1.6.3
 Git commit:      e03ddb8/1.10.3
 Built:           
 OS/Arch:         linux/amd64

Server:
 Version:         1.10.3
 API version:     1.22
 Package version: docker-common-1.10.3-55.gite03ddb8.fc24.x86_64
 Go version:      go1.6.3
 Git commit:      e03ddb8/1.10.3
 Built:           
 OS/Arch:         linux/amd64

# docker search fedora
INDEX       NAME                                    DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
docker.io   docker.io/fedora                        Official Docker builds of Fedora                501       [OK]       
docker.io   docker.io/mattsch/fedora-nzbhydra       Fedora NZBHydra                                 4                    [OK]
docker.io   docker.io/dockingbay/fedora-rust        Trusted build of Rust programming language...   3                    [OK]
docker.io   docker.io/startx/fedora                 Simple container used for all startx based...   3                    [OK]
docker.io   docker.io/darksheer/fedora              Hourly update latest Fedora Image               1                    [OK]
docker.io   docker.io/darksheer/fedora22            Base Fedora 22 Image -- Updated hourly          1                    [OK]
docker.io   docker.io/darksheer/fedora23            Hourly updated Fedora 23                        1                    [OK]
docker.io   docker.io/mattsch/fedora-rpmfusion      Base container for Fedora with RPM Fusion ...   1                    [OK]
docker.io   docker.io/pacur/fedora-22               Pacur Fedora 22                                 1                    [OK]
docker.io   docker.io/heichblatt/fedora-pkgsrc      pkgsrc on latest Fedora                         0                    [OK]
docker.io   docker.io/kino/fedora                   fedora base                                     0                    [OK]
docker.io   docker.io/krystalcode/fedora            Fedora base image that includes some addit...   0                    [OK]
docker.io   docker.io/mattsch/fedora-couchpotato    Fedora Couchpotato                              0                    [OK]
docker.io   docker.io/mattsch/fedora-htpc           Fedora HTPC                                     0                    [OK]
docker.io   docker.io/mattsch/fedora-nzbget         Fedora NZBGet                                   0                    [OK]
docker.io   docker.io/mattsch/fedora-plex           Fedora Plex                                     0                    [OK]
docker.io   docker.io/mattsch/fedora-sonarr         Fedora Sonarr                                   0                    [OK]
docker.io   docker.io/mattsch/fedora-transmission   Fedora Transmission                             0                    [OK]
docker.io   docker.io/opencpu/fedora                Stable version of opencpu for Fedora            0                    [OK]
docker.io   docker.io/qnib/fedora                   Base QNIBTerminal image of fedora               0                    [OK]
docker.io   docker.io/smartentry/fedora             fedora with smartentry                          0                    [OK]
docker.io   docker.io/termbox/fedora                Fedora                                          0                    [OK]
docker.io   docker.io/vcatechnology/fedora          A Fedora image that is updated daily            0                    [OK]
docker.io   docker.io/vcatechnology/fedora-ci       A Fedora image that is used in the VCA Tec...   0                    [OK]
docker.io   docker.io/zartsoft/fedora               Base fedora image                               0                    [OK]

# docker pull docker.io/fedora
Using default tag: latest
Trying to pull repository docker.io/library/fedora ... 
latest: Pulling from docker.io/library/fedora
1b39978eabd9: Pull complete 
Digest: sha256:8d3f642aa4d3fa8f9dc52ab0e3bbbe8bc2494843dc6ebb26c4a6958db888e5a2
Status: Downloaded newer image for docker.io/fedora:latest

# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/fedora    latest              1f8ec1108a3f        2 weeks ago         230.3 MB

# docker run docker.io/fedora echo "hello word"
hello word

# docker ps -a
CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS                     PORTS               NAMES
fa71ff875c69        docker.io/fedora    "echo 'hello word'"   2 minutes ago       Exited (0) 2 minutes ago                       tender_euler



```

```
https://docs.docker.com/docker-hub/

step1——找到本地镜像的ID：docker images
step2——登陆Hub：docker login --username=username --password=password --email=email
step3——tag：docker tag <imageID> <namespace>/<image name>:<version tag eg latest>
step4——push镜像：docker push <namespace>/<image name>

1. 建立一个private的 registry
2. 制作一个image：可以dockerfile，也可以docker commit
3. docker push 即可 
```



```
1. 进入docker容器
# docker run -p 8080:8080 --name test -it docker.io/fedora /bin/bash
# docker ps -a
CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS                  PORTS               NAMES
f48bde61c9c0        docker.io/fedora    "/bin/bash"           4 days ago          Up 4 days                                   test
fa71ff875c69        docker.io/fedora    "echo 'hello word'"   4 days ago          Exited (0) 4 days ago                       tender_euler
# docker inspect -f {{.State.Pid}} f48bde61c9c0
20734


2. install python2.7
# yum install python

3. install git and clone 
# yum install git
# git clone http://git.app.eng.bos.redhat.com/git/account-management-tool.git

4. install Ethel environment
# yum install gcc
# yum install redhat-rpm-config
# yum install python-devel
# yum install openssl-devel
# cd account-management-tool/
# pip install -r requirements.txt

5. 

``` 

```
#git clone https://github.com/tanfengshuang/ethel.git
# chcon -Rt  svirt_sandbox_file_t /ethel/
# chcon -R --reference /var/www/cgi-bin/test/ /ethel/ 

# yum install -y python git gcc redhat-rpm-config python-devel openssl-devel httpd mod_wsgi which vim
# vim /etc/httpd/conf.d/wsgi.conf
LoadModule wsgi_module modules/mod_wsgi.so

WSGIScriptAlias / /opt/app-root/src/wsgi/application

# apachectl -k start
# apachectl -k restart
```


```
oc login https://open.paas.redhat.com
oc project test
oc status

# oc login https://open.paas.redhat.com
# oc status
# oc get services
NAME                CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
ethel               172.30.226.237   <none>        8080/TCP    10m
glusterfs-cluster   172.30.127.104   <none>        24007/TCP   4d

# oc get all
NAME       TYPE      FROM         LATEST
bc/ethel   Source    Git@master   1

NAME             TYPE      FROM         STATUS    STARTED          DURATION
builds/ethel-1   Source    Git@master   Failed    11 minutes ago   3m45s

NAME       DOCKER REPO                      TAGS      UPDATED
is/ethel   172.30.114.236:5000/test/ethel             

NAME       REVISION   DESIRED   CURRENT   TRIGGERED BY
dc/ethel   0          1         0         config,image(ethel:latest)

NAME           HOST/PORT                             PATH      SERVICES   PORT       TERMINATION
routes/ethel   ethel-test.int.open.paas.redhat.com             ethel      8080-tcp   

NAME                    CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
svc/ethel               172.30.226.237   <none>        8080/TCP    11m
svc/glusterfs-cluster   172.30.127.104   <none>        24007/TCP   4d

NAME               READY     STATUS    RESTARTS   AGE
po/ethel-1-build   0/1       Error     0          11m

# oc get pods
  NAME            READY     STATUS    RESTARTS   AGE
ethel-1-build   0/1       Error     0          11m

# oc export pod ethel-1-build -o json > ethel-1-build.json
# oc logs ethel-1-build
# oc exec -p ethel-1-build -it /bin/bash

# oc get svc
# oc get services
NAME                CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
ethel               172.30.226.237   <none>        8080/TCP    14m
glusterfs-cluster   172.30.127.104   <none>        24007/TCP   4d
# oc describe service ethel
Name:			ethel
Namespace:		test
Labels:			app=ethel
Selector:		deploymentconfig=ethel
Type:			ClusterIP
IP:			172.30.226.237
Port:			8080-tcp	8080/TCP
Endpoints:		<none>
Session Affinity:	None
No events.


```

https://access.redhat.com/documentation/en-us/openshift_container_platform/3.3/html-single/installation_and_configuration/

openshift 3 - new app: 
https://docs.openshift.org/latest/dev_guide/application_lifecycle/new_app.html


https://docs.openshift.org/latest/using_images/s2i_images/python.html

# oc env dc/username APP_FILE=wsgi/application  

Opening a Remote Shell to Containers
https://docs.openshift.org/latest/dev_guide/ssh_environment.html

```
oc exec docker-registry-1-qkxpw -it bash
oc exec docker-registry-1-qkxpw cat /etc/resolv.conf
oc exec docker-registry-1-qkxpw ls /

# oc get pods
NAME                  READY     STATUS             RESTARTS   AGE
test-ethel-1-build    0/1       Completed          0          10m
test-ethel-1-deploy   1/1       Running            0          7m
test-ethel-1-rekra    0/1       CrashLoopBackOff   6          6m
# oc rsh test-ethel-1-deploy
```


https://gitlab.com/osevg/python-flask-modwsgi



Testing out deployment of Python based Opal health care framework.
http://blog.dscpl.com.au/2016/08/testing-out-deployment-of-python-based.html

Building a better user experience for deploying Python web applications.
http://blog.dscpl.com.au/2016/02/building-better-user-experience-for.html

openshift 3 - database:
https://docs.openshift.org/latest/using_images/db_images/mysql.html

# oc new-app -e MYSQL_USER=account_tool -e MYSQL_PASSWORD=redhat -e MYSQL_DATABASE=ethel_database openshift/mysql
# oc get pods
NAME                  READY     STATUS      RESTARTS   AGE
mysql-5-pbyw7         1/1       Running     0          38s


openshift 3 - sso: 
https://docs.openshift.com/enterprise/3.1/using_images/xpaas_images/sso.html

openshift 3 - 
https://docs.openshift.com/enterprise/3.1/install_config/advanced_ldap_configuration/configuring_form_based_authentication.html
https://docs.openshift.com/enterprise/3.0/admin_guide/configuring_authentication.html




https://blog.openshift.com/getting-started-python/




https://docs.openshift.com/enterprise/3.0/architecture/core_concepts/builds_and_image_streams.html
https://docs.openshift.com/container-platform/3.3/dev_guide/managing_images.html#creating-an-image-stream-by-manually-pushing-an-image

```
# oc import-image test-fedora --from=docker.io/fedora --confirm
The import completed successfully.

Name:			test-fedora
Namespace:		test
Created:		1 seconds ago
Labels:			<none>
Annotations:		openshift.io/image.dockerRepositoryCheck=2017-03-13T07:55:53Z
Docker Pull Spec:	172.30.114.236:5000/test/test-fedora
Unique Images:		1
Tags:			1

latest
  tagged from docker.io/fedora

  * docker.io/fedora@sha256:8d3f642aa4d3fa8f9dc52ab0e3bbbe8bc2494843dc6ebb26c4a6958db888e5a2
      1 seconds ago

# oc get is
NAME          DOCKER REPO                            TAGS      UPDATED
test-ethel    172.30.114.236:5000/test/test-ethel    latest    4 hours ago
test-fedora   172.30.114.236:5000/test/test-fedora   latest    18 seconds ago

# oc import-image ethel:test-ethel --from=docker.io/tanfengshuang/ethel:test-ethel --confirm

# oc import-image ethel --from=docker.io/tanfengshuang/ethel --confirm
The import completed successfully.

Name:			ethel
Namespace:		test
Created:		Less than a second ago
Labels:			<none>
Annotations:		openshift.io/image.dockerRepositoryCheck=2017-03-13T10:17:13Z
Docker Pull Spec:	172.30.114.236:5000/test/ethel
Unique Images:		1
Tags:			1

latest
  tagged from docker.io/tanfengshuang/ethel

  * docker.io/tanfengshuang/ethel@sha256:fe0098800d8c3a27040f3ffb7d4bae2709b4534f702bafb3a783fd2c9c822560
      Less than a second ago

# oc get is
NAME         DOCKER REPO                           TAGS      UPDATED
ethel        172.30.114.236:5000/test/ethel        latest    25 seconds ago
test-ethel   172.30.114.236:5000/test/test-ethel   latest    7 hours ago
```

```
重新部署新的代码
# git push
Counting objects: 5, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (5/5), 547 bytes | 0 bytes/s, done.
Total 5 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), completed with 1 local objects.
To git@github.com:tanfengshuang/ethel.git
   afb6f9a..e076350  master -> master
# oc start-build wsgi-new
build "wsgi-new-2" started


# oc get scc
NAME               PRIV      CAPS      SELINUX     RUNASUSER          FSGROUP     SUPGROUP    PRIORITY   READONLYROOTFS   VOLUMES
anyuid             false     []        MustRunAs   RunAsAny           RunAsAny    RunAsAny    10         false            [configMap downwardAPI emptyDir persistentVolumeClaim secret]
hostaccess         false     []        MustRunAs   MustRunAsRange     MustRunAs   RunAsAny    <none>     false            [configMap downwardAPI emptyDir hostPath persistentVolumeClaim secret]
hostmount-anyuid   false     []        MustRunAs   RunAsAny           RunAsAny    RunAsAny    <none>     false            [configMap downwardAPI emptyDir hostPath nfs persistentVolumeClaim secret]
hostnetwork        false     []        MustRunAs   MustRunAsRange     MustRunAs   MustRunAs   <none>     false            [configMap downwardAPI emptyDir persistentVolumeClaim secret]
nonroot            false     []        MustRunAs   MustRunAsNonRoot   RunAsAny    RunAsAny    <none>     false            [configMap downwardAPI emptyDir persistentVolumeClaim secret]
privileged         true      []        RunAsAny    RunAsAny           RunAsAny    RunAsAny    <none>     false            [*]
restricted         false     []        MustRunAs   MustRunAsRange     MustRunAs   RunAsAny    <none>     false            [configMap downwardAPI emptyDir persistentVolumeClaim secret]
```

    WSGI can be deployed by installing gunicorn or python-flask-modwsgi after modify code in openshift 3, but in order to resolve kerberos configuration issue, I think it's better to deploy apache to finish wsgi and kerberos
    
    ftan> 还在么，如果我有权限修改pod中的apache配置，当我更新代码后，需要重新build，pod然后也会重新部署，那每次都需要再去修改pod阿？
<ftan> 如果我需要自己配置apache, 只能修改pod么？还是也可以修改image
<deshuai> https://docs.openshift.org/latest/dev_guide/builds/index.html    https://docs.openshift.org/latest/dev_guide/deployments/how_deployments_work.html
<deshuai> build+deployment可以做CI/CD
