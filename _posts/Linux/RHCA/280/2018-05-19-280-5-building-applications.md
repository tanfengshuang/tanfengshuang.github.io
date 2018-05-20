---
layout: post
title:  "Building Applications(280-4)"
categories: Linux
tags: RHCA 280
---


### Selecting a Build Method

OpenShift Enterprise (OSE) runs applications inside containers as Kubernetes pods. OSE can build a pod from three different sources:

1. A container image: The first source (container images) leverages the container ecosystem. Many vendors already package their applications as container images, and the system administrator can create a pod to run those application images inside an OSE instance. Many open source applications are also ready to use as container images from the public image registry at the Docker Hub. If the application needs back-end services, such as a database, and those services are also packaged as container images, a system administrator can build and deploy the entire application and its supporting infrastructure to OSE from those images.

2. A Dockerfile: The second source (Dockerfiles) also leverages the container ecosystem. A Dockerfile is the Docker community standard way of specifying an script to build a container image from Linux OS distribution tools. Images generated from Dockerfiles are based on a minimum RHEL, CentOS, Fedora, or other Linux distribution installation, over which distribution packages are added and shell commands executed. A Dockerfile-based image build is similar to a RHEL Kickstart installation, but results in a container image instead of a physical or VM server installation. OSE can relieve the burden of running the Dockerfile build from the system administrator or developer workstation, and bring it to an OSE instance.

Dockerfiles are powerful because they can build over other container images that are already proven. Those images may have been generated from Dockerfiles, which in turn refer to other third-party images, such a Wordpress image built over a PHP5 image that in turn was built from a CentOS base image. Dockerfiles are really just a way to automate installing an OS and application binaries.

There are many ready-to-use Dockerfiles for popular open source application on public repositories like Github. Many of those projects do not provide container images at all, because they do not want the overhead of having to continually publish new images to cope with new application releases and dependency library updates. They provide just the Dockerfiles and leave to the user the task of rebuilding images from the Dockerfiles whenever there is an update or security fix.

3. Application source code(Source-to-Image(S2I)): The third source, S2I, empowers a developer to build container images for an application without dealing with or knowing about Docker internals, image registries, and Dockerfiles. It supports many modern development and deployment practices such as continuous delivery, microservices, and A/B testing. S2I can also relieve the developer workstation from the burden of compiling source code into binaries, minimizing JavaScript files and other resource-intensive tasks, leaving them to be executed by the OSE instance.

Building a pod from application source code does not actually need to start from raw source code. S2I also allows building the application binaries outside of an OSE instance and adding the packaged binaries to a base image, thus providing a superior alternative to Dockerfiles. Images generated from Dockerfiles can be very inefficient because of too many layers, but an image generated from S2I will have a single layer over the base image.

An independent software vendor (ISV) can use S2I to generate container images that are later offered to customers from a private or public container image registry. But any organization that develops software internally, such as a cloud application provider or an online retailer, can use S2I to minimize the time from development to production.

###### Creating a pod from a container image

1. Given the pod resource definition file mypod.json, the oc create -f command creates the pod in the user's current project:

$ oc create -f mypod.json

2. Another way to create a pod from a container image is by using the oc new-app command, passing an image name as argument. For example:

$ oc new-app openshift3/mysql-55-rhel7



###### Tools to build pods from container images

OSE provides three tools to create an application, that is, to create a pod, from container images. Two of them are CLI-based, and the last one is GUI-based:

    Tool name       CLI or Web      Description
*    oc create -f 	CLI 	        Passing as an argument a resource definition file using JSON or YAML syntax. The resource definition file describes each pod, the containers inside it, and usually includes additional resources such as services, routes, ReplicationControllers, and PersistentVolumeClaims. Those resource types will be detailed in this chapter.
*    oc new-app 	    CLI 	        Passing as an argument an image name. The pod definition using that image will be embedded by OSE inside a DeploymentController, which in turn embeds a Kubernetes ReplicationController and a service. This redirection (from DeploymentController to ReplicationController to pods, and from service to pod) makes it possible to implement scalable applications and also update strategies like rolling updates.
*    Add to project 	Web 	        OSE web console, which provides a graphical workflow to define resources much alike oc new-app would create. The web console makes it easier to customize the resources created for the application using a wizard-like interface.



###### Creating an application from a Dockerfile

$ oc new-app http://git.example.com/dockerfile -o json > dockerbuild.json
... edit dockerbuild.json as needed ...
$ oc create -f dockerbuild.json

This will not create a simple pod resource, but a pod embedded in a DeploymentConfig resource that uses a container image created by a BuildConfig. More details will appear later in this chapter in the section about Source-To-Image (S2I).

In a pure Docker environment, simple Dockerfiles may generate images that fail to run in OCP. Most of the time, this is because the image needs to run as root or some other fixed UID. But OCP runs containers using a random UID. To fix this, make sure folders being written to while running the container have a+rwx permis

### Creating a Pod from a Docker Image

```
1. Create a pod using the MySQL 5.5 image in OCP.

[student@workstation ~]$ oc login -u student -p redhat
[student@workstation ~]$ oc new-project database-pod
[student@workstation ~]$ oc create -f /home/student/DO280/labs/deploy-pod/mysqldb-pod.json
[student@workstation ~]$ less /home/student/DO280/labs/deploy-pod/mysqldb-pod.json
{
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
        "name": "mysqldb",
        "labels": {
            "name": "mysqldb"
        }
    },
    "spec": {
        "containers": [
            {
                "name": "mysqldb",
                "image": "openshift3/mysql-55-rhel7",
                "ports": [
                    {
                        "containerPort": 3306
                    }
                ],
                "env": [
                    {
                        "name": "MYSQL_USER",
                        "value": "ose"
                    },
                    {
                        "name": "MYSQL_PASSWORD",
                        "value": "openshift"
                    },
                    {
                        "name": "MYSQL_DATABASE",
                        "value": "quotes"
                    }
                ]
            }
        ]
    }
}

2. Verify that the pod was created successfully:

[student@workstation ~]$ watch oc get pods
[student@workstation ~]$ oc env pod mysqldb --list
# pods mysqldb, container mysqldb
MYSQL_USER=ose
MYSQL_PASSWORD=openshift
MYSQL_DATABASE=quotes

3. Populate the MySQL database.
 
[student@workstation ~]$ less /home/student/DO280/labs/deploy-pod/quote.sql
create table quote (id integer primary key, msg varchar(250));
insert into quote values (1, 'Always remember that you are absolutely unique. Just like everyone else.');
insert into quote values (2, 'Do not take life too seriously. You will never get out of it alive.');
insert into quote values (3, 'People who think they know everything are a great annoyance to those of us who do.');

    You need to populate the MySQL database with some data. To do this you must forward an unused local port to the MySQL server port on the database pod. You can then use the mysql client program on the workstation host to load data into the quotes database:

[student@workstation ~]$ oc port-forward mysqldb 13306:3306
Forwarding from 127.0.0.1:13306 -> 3306
Forwarding from [::1]:13306 -> 3306

[student@workstation ~]$ mysql -h127.0.0.1 -P13306 -uose -popenshift quotes < /home/student/DO280/labs/deploy-pod/quote.sql

> The MySQL client shows no output if the data was loaded successfully.
> Go back to the terminal where the oc port-forward command was left running. Use Ctrl+C to stop it. This terminal can now be closed. 

4. Verify that the MySQL server was successfully populated. Use oc exec to run the MySQL client inside the database pod:

[student@workstation ~]$ oc exec mysqldb -it -- /bin/bash -c "mysql -h127.0.0.1 -uose -popenshift quotes"
mysql> select count(*) from quote;
+----------+
| count(*) |
+----------+
| 3 |
+----------+
1 row in set (0.01 sec)
mysql> exit
    
> The -- option has to come before /bin/bash so that Bash options are not interpreted by the oc exec command.
> The reason oc exec was not used to populate the database is that there are issues with input redirection. If a developer wants to connect to the database using an integrated development environment (IDE) of choice, then the developer needs to use the oc port-forward command. However, a system administrator needs to run other commands inside a pod, so the oc exec command is used.

5. Optional: Compare the pod definition provided for this lab to the one which is generated by oc new-app:

[student@workstation ~]# oc new-app workstation.lab.example.com:5000/openshift3/mysql-55-rhel7 --name=mysqldb -o json > mysql-pod-new-app.json
[student@workstation ~]# less mysql-pod-new-app.json
-> Inspect the mysql-pod-new-app.json file to see the pod definition that is embedded inside a DeploymentConfig resource, as the "template" attribute
文件中资源类型是ImageStream， 有Service, DeploymentConfig
缺点： 文件中没有用于存储数据库用户名 密码等环境变量
所以，pod不推荐用oc new-app这种命令行创建，需要通过oc create -f ..， 如果创建project等简单的，可以用oc new-project

> Use the -o json option to just generate a resource definition file instead of creating another pod and associated resources.
> Do not forget the -o json option. You do not want to create any new resources inside the database-pod project.



6. Clean up. Deleting the database-pod project also deletes the pod.

[student@workstation ~]$ oc delete project database-pod
project "database-pod" deleted

```


### Enabling Services for a Pod

Pod像是沙箱, Pod中跑容器，可以多个容器。Pod只能在集群内部(master node)访问，需要通过端口转发被外部访问
Pod 会随时被摧毁，ip会改变，如果重启机器，集群时，或者弹性伸缩
Service: 为了抽象访问Pod，代理访问，会生成一个固定的集群IP，当dm/rc进行拉伸时，这个IP不变。但是这个集群IP也是不能被外部访问的，只是一个固定的优点
Openshift 中RC不是重点，因为openshift已经由RC跨越到了DeploymentConfig


###### Working with services

Services are essential resources to any OSE application. They allow containers in one pod to open network connections to containers in other pod without one pod interfering on other pod life cycles. A pod may be restarted for many reasons, and will get a different internal IP each time. Instead of a pod having to discover other pods internal container network IPs after each restart, a service provides a stable IP address for other pods to connect, no matter what node runs the pod after each restart.

Most real-world applications will not run as a single pod. They need to scale horizontally, so many pods run the same containers from the same pod resource definition to meet a growing user demand. A service is tied to a set of pods, providing a single IP address to the set and load-balancing client request among the member pods.

The set of pods running behind a service is managed by a DeploymentConfig resource. A DeploymentConfig resource embeds a ReplicationController that manages how many pod copies (replicas) have to be created, and creates new ones if some of them fail, whatever the reason. DeploymentConfig and ReplicationControllers will be detailed presented later in this chapter, together with the Source-To-Image (S2I) process.

The following listing shows a minimal service definition:
```
{
    "kind": "Service",
    "apiVersion": "v1",
    "metadata": {
        "name": "quotedb"
    },
    "spec": {
        "ports": [
            {
                "port": 3306,
                "targetPort": 3306
            }
        ],
        "selector": {
            "name": "mysqldb"
        }
    }
}
```

The service example forwards connections to a pod whose definition complies to the following listing:
```
{
    "kind": "Pod",
    "metadata": {
        "labels": {
            "name": "mysqldb"
        }
    },
    "spec": {
        "containers": [
            {
                "ports": [
                    {
                        "containerPort": 3306
                    }
                ],
                ... other container attributes do not matter ...
            }
        ]
    },
    ... other pod attributes do not matter ...
}
```

```
1. On the workstation host, log in as the student user, and create a new project with a database pod, and populate the database:
[student@workstation ~]$ oc login -u student -p redhat
[student@workstation ~]$ oc new-project service-pod
Now using project “service-pod” on server “https://master.lab.example.com:8443”.
[student@workstation ~]$ oc create -f /home/student/DO280/labs/deploy-service/mysqldb-pod.json
pod “mysqldb” created
[student@workstation ~]$ watch oc get pods
NAME 		READY 	REASON 			RESTARTS 	AGE
mysqldb 	1/1 		Running 			0 		12s
[student@workstation ~]$ oc port-forward mysqldb 13306:3306
Forwarding from 127.0.0.1:13306 -> 3306
Forwarding from [::1]:13306 -> 3306
[student@workstation ~]$ mysql -h127.0.0.1 -P13306 -uose -popenshift quotes < /home/student/DO280/labs/deploy-service/quote.sql

2. Create a service using the provided JSON resource definition file
[student@workstation ~]$ less /home/student/DO280/labs/deploy-service/mysql-service.json
{
    "kind": "Service",
    "apiVersion": "v1",
    "metadata": {
        "name": "quotedb"
    },
    "spec": {
        "ports": [
            {
                "port": 3306,
                "targetPort": 3306
            }
        ],
        "selector": {
            "name": "mysqldb"
        }
    }
}
[student@workstation ~]$ oc describe pod mysqldb | head -n 6
Name: 			mysqldb
Namespace: 		service-pod
Security Policy: 	restricted
Node: 			node.lab.example.com/172.25.250.11
Start Time:		Fri, 03 Feb 2017 01:06:47 +0530
Labels: 		name=mysqldb 
> Verify that the MySQL server pod defines a label that matches the selector from the quotedb service:

3. Create and validate the quotedb service:
[student@workstation ~]$ oc create -f /home/student/DO280/labs/deploy-service/mysql-service.json
service "quotedb" created
[student@workstation ~]$ oc describe svc quotedb
Name:			quotedb
Namespace:		service-pod
Labels:			<none>
Selector:		name=mysqldb
Type:			ClusterIP
IP:			172.30.59.128                   # IP
Port:			<unset>	3306/TCP
Endpoints:		10.129.0.17:3306            # 如果Endpoints为空，则service失败
Session Affinity:	None
No events.
> Inspect the newly created service to get its assigned IP address:

4. Use a MySQL client to verify that the service allows access to the MySQL server inside the pod:
[root@master ~]# oc describe svc quotedb -n service-pod
[root@master ~]# mysql -h172.30.59.128 -uose -popenshift quotes
mysql [quotes]> select count(*) from quote;
+----------+
| count(*) |
+----------+
|        3 |
+----------+
1 row in set (0.01 sec)

> Run an SQL query to verify that there are records in the quote table:

5. Create a test application in the service-pod project from the provided resource definition file:

[student@workstation ~]$ less /home/student/DO280/labs/deploy-service/quoteapp-pod.json
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "quoteapp",
    "labels": {
        "name": "quoteapp"
    }
  },
  "spec": {
    "containers": [
      {
        "name": "quoteapp",
        "image": "php-quote",
        "ports": [
          {
            "containerPort": 8080
          }
        ]
      }
    ]
  }
}
[student@workstation ~]$ oc create -f /home/student/DO280/labs/deploy-service/quoteapp-pod.json
pod “quoteapp” created
[student@workstation ~]$ watch oc get pods
NAME       READY     REASON    RESTARTS   AGE
mysqldb    1/1       Running   0          18m
quoteapp   1/1       Running   0          19s

6. Verify that the test application knows about the quotedb service.

[student@workstation ~]$ oc exec quoteapp -it bash
bash-4.2$ echo $QUOTEDB_SERVICE_HOST ; echo $QUOTEDB_SERVICE_PORT
172.30.59.128
3306
bash-4.2$ grep QUOTEDB_SERVICE /var/www/html/index.php
$link = mysqli_connect($_ENV["QUOTEDB_SERVICE_HOST"],"ose","openshift","quotes", $_ENV["QUOTEDB_SERVICE_PORT"])
        or die("Error " . mysqli_error($link)); 
bash-4.2$ exit

7. Verify that the test application can query the MySQL server in pod mysqldb through the quotedb service:
This step must be executed on the master host, because the workstation host cannot access the SDN internal pod and you have not created a route for the test application yet. The previous step must be executed on the workstation host.

[root@master ~]# oc describe pod quoteapp -n service-pod | grep IP
IP: 10.1.0.17

> The root user in the master host should be running the OCP client as system:admin, so option -n is needed to find resources inside a user project.

[root@master ~]# curl http://10.1.0.17:8080/
Do not take life too seriously. You will never get out of it alive.

8. Clean Up

[student@workstation ~]$ oc delete project service-pod
project/service-pod




也可以用expose生成service，不用上面的oc create -f命令
但是不太建议用expose，因为参数有点多
# oc expose pod mysqldb --name=quotedb --selector='name=mysqldb'
```

###### Discovering services

An application typically finds a service IP address and port by using environment variables. For each service inside an OCP project, the following environment variables are automatically defined for all pods inside the same project:

*    SVC_NAME_SERVICE_HOST is the service IP address.
*    SVC_NAME_SERVICE_PORT is the service TCP port.

Another way to discover a service from a pod is by using the OCP internal DNS server, which is visible only to pods. Each service is dynamically assigned an SRV record with a FQDN of the form:

*    SVC_NAME.PROJECT_NAME.svc.cluster.local

When discovering services using environment variables, a pod has to be created (and started) only after the service is created. But if the application was written to discover services using DNS queries, it can find services created after the pod was started.


### Creating Routes

```
1. Log in as the student user, create a new project that includes an application and a database pod, and populate the database:

[student@workstation ~]$ oc login -u student -p redhat
[student@workstation ~]$ oc new-project route-service-pod
[student@workstation ~]$ oc create -f /home/student/DO280/labs/deploy-route/quoteapp-with-mysqldb.json
[student@workstation ~]$ watch oc get pods
NAME       READY     REASON    RESTARTS   AGE
quoteapp   1/1       Running   0          11s
mysqldb    1/1       Running   0          12s
[student@workstation ~]$ oc port-forward mysqldb 13306:3306
Forwarding from 127.0.0.1:13306 -> 3306
Forwarding from [::1]:13306 -> 3306
[student@workstation ~]$ mysql -h127.0.0.1 -P13306 -uose -popenshift quotes < /home/student/DO280/labs/deploy-route/quote.sql

2. Review the provided route definition

route definition:
[student@workstation ~]$ less /home/student/DO280/labs/deploy-route/quoteapp-route.json
{
  "apiVersion": "v1",
  "kind": "Route",
  "metadata": {
    "name": "quoteapp"
  },
  "spec": {
    "host": "quoteapp.cloudapps.lab.example.com",
    "to": {
      "kind": "Service",
      "name": "quoteapp"
    }
  }
}

> This route definition references the quoteapp service resource, but that service was not created for the test application.

service definition:
[student@workstation ~]$ less /home/student/DO280/labs/deploy-route/quoteapp-service.json
{
    "kind": "Service",
    "apiVersion": "v1",
    "metadata": {
        "name": "quoteapp"
    },
    "spec": {
        "ports": [
            {
                "port": 80,
                "targetPort": 8080
            }
        ],
        "selector": {
            "name": "quoteapp"
        }
    }
}

> A service uses a selector attribute to refer to pods based on labels defined for the pods, but a route uses the service name directly.

[student@workstation ~]$ ping -c 1 quoteapp.cloudapps.lab.example.com
PING quoteapp.cloudapps.lab.example.com (172.25.250.11) 56(84) bytes of data.
64 bytes from node.lab.example.com (172.25.250.11): icmp_seq=1 ttl=64 time=2.40 ms
...
> Verify that DNS resolves the route's Fully Qualified Domain Name (FQDN) to the OCP router, which is the node host

> DNS resolutions depend on the wild-card DNS domain, which is a prerequisite for a successful OCP installation. It does not depend on the route definition.

3. Create the service and the route. (Names for different OCP resource types do not collide, so a service name can be the same as a pod name or a route name. It is customary to have the same name for related resources so it is easier to correlate them.)

[student@workstation ~]$ oc create -f /home/student/DO280/labs/deploy-route/quoteapp-service.json
service "quoteapp" created
[student@workstation ~]$ oc create -f /home/student/DO280/labs/deploy-route/quoteapp-route.json
route "quoteapp" created

4. Verify that the service and route were created as expected:

[student@workstation ~]$ oc describe svc quoteapp
Name:                 quoteapp
Namespace:            route-service-pod
Labels:               <none>
Selector:             name=quoteapp
Type:                 ClusterIP
IP:                   172.30.177.167
Port:                 <unset>	80/TCP
Endpoints:            10.129.0.22:8080
Session Affinity:     None
No events.

[student@workstation ~]$ oc describe route quoteapp
Name:                quoteapp
Namespace:           route-service-pod
Created:             About a minute ago
Labels:              <none>
Annotations:         <none>
Requested Host:      quoteapp.cloudapps.lab.example.com
                       exposed on router router about a minute ago
...
Endpoint Port:       <all endpoint ports>

Service:             quoteapp
Weight:              100 (100%)
Endpoints:           10.129.0.22:8080

5. Access the test application using the route. Use the FQDN from the route definition to access the application using curl:

[student@workstation ~]$ curl http://quoteapp.cloudapps.lab.example.com

6. Create a route using oc expose instead of a JSON file. (Delete the route and recreate it with expose)

[student@workstation ~]$ oc delete route quoteapp
route "quoteapp" deleted
[student@workstation ~]$ oc expose service quoteapp --port=8080
route "quoteapp" exposed
[student@workstation ~]$ oc describe route quoteapp
Name:                 quoteapp
Namespace:            route-service-pod
Created:              17 seconds ago
Labels:               <none>
Annotations:          openshift.io/host.generated=true
Requested Host:       quoteapp-route-service-pod.cloudapps.lab.example.com 
                        exposed on router router 17 seconds ago
...
Endpoint Port:        8080

Service:              quoteapp
Weight:               100 (100%)
Endpoints:            10.129.0.22:8080
[student@workstation ~]$ curl  http://quoteapp-route-service-pod.cloudapps.lab.example.com

> The route FQDN generated by OCP concatenates the service names with the project name, separated by a dash (-) as the host name. An alternate FQDN could have been selected by using the --hostname option to the oc expose command. For example:
# oc expose service quoteapp --port=8080 --hostname=quoteapp.cloudapps.lab.example.com

Access OCP router statistics.
Important

7. This step has some commands that must be run on the master host, others on the node host, and yet others on the workstation host.

    a. On the master host, ensure you are using the default project and then find the router name:

    [root@master]# oc project default
    [root@master]# oc get pods
    NAME                       READY     STATUS      RESTARTS       AGE
    docker-registry-6-kwv2i    1/1       Running     4              7d
    registry-console-1-zlrry   1/1       Running     4              7d
    router-1-32toa             1/1       Running     4              7d

    b. On the master host, check the router environment variables to find connection parameters for the HAProxy process running inside the pod:

    [root@master]# oc env pod router-1-32toa --list | tail -n 6
    ROUTER_SERVICE_NAME=router
    ROUTER_SERVICE_NAMESPACE=default
    ROUTER_SUBDOMAIN=
    STATS_PASSWORD=shRxnWSdn9
    STATS_PORT=1936
    STATS_USERNAME=admin

    > The password in variable STATUS_PASSWORD was randomly generated when the router was previously created. Variables STATS_USERNAME and STATS_PORT have fixed default values, but all of them can be changed at router creation time.

    c. On the node host, configure iptables to open STATS_PORT:

    [root@node ~]# iptables -I OS_FIREWALL_ALLOW -p tcp -m tcp --dport 1936 -j ACCEPT

    > This change is persistent only if the system administrator adds this firewall rule to /etc/sysconfig/iptables.

    d. On the workstation host, open a web browser and access the HAProxy statistics URL http://node.lab.example.com:STATS_PORT/.
    Type the value from STATS_USERNAME into User Name and from STATS_PASSWORD into Password.
    A web page will be opened, look for a section labeled #be_http_route-service-pod_quoteapp, which represents the route created in this lab for the quoteapp pod. 
    
    e. Alternatively, as the HTTP BASIC authentication without SSL is not secure, add the login and password to the URL itself, as in:

    http://STATS_USERNAME:STATS_PASSWORD@node.lab.example.com:STATS_PORT/

    For example: http://admin:shRxnWSdn9@node.lab.example.com:1936/

    f. On the workstation host, get a text-only version of the HAProxy statistics page, and filter the output to show only data for the quoteapp route:

    Add to the stats URL parameters ?stats;csv and pipe the output to grep.

    [student@workstation ~]$ curl -s 'http://admin:shRxnWSdn9@node.lab.example.com:1936/?stats;csv' | grep quoteapp
    be_http_route-service-pod_quoteapp,29a961b64ad8b4206120d3e
    301376f5f,0,0,0,1,,1,116,275,,0,,0,0,0,0,UP,100,1,0
    ,0,0,15094,0,,1,12,1,,1,,2,0,,1,L4OK
    ,,0,0,1,0,0,0,0,0,,,,0,0,,,,,15057,,,0,1,1,1,
    be_http_route-service-pod_quoteapp,BACKEND,0,
    0,0,1,1,1,116,275,0,0,,0,0,0,0,UP,100,1,0,,0,15094,
    0,,1,12,0,,1,,1,0,,1,,,,0,1,0,0,0,0,,,,,0,0,
    0,0,0,0,15057,,,0,1,1,1,

8. Clean up the route-service-pod project. This also deletes all pods, services, and routes created during this lab.

[student@workstation ~]$ oc delete project route-service-pod
project "route-service-pod" deleted

```


pod service 用文件创建
project route 用命令创建

