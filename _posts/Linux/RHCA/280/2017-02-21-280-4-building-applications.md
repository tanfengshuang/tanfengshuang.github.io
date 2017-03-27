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

###### Tools to build pods from container images

OSE provides three tools to create an application, that is, to create a pod, from container images. Two of them are CLI-based, and the last one is GUI-based:

    Tool name       CLI or Web      Description
*    oc create -f 	CLI 	        Passing as an argument a resource definition file using JSON or YAML syntax. The resource definition file describes each pod, the containers inside it, and usually includes additional resources such as services, routes, ReplicationControllers, and PersistentVolumeClaims. Those resource types will be detailed in this chapter.
*    oc new-app 	    CLI 	        Passing as an argument an image name. The pod definition using that image will be embedded by OSE inside a DeploymentController, which in turn embeds a Kubernetes ReplicationController and a service. This redirection (from DeploymentController to ReplicationController to pods, and from service to pod) makes it possible to implement scalable applications and also update strategies like rolling updates.
*    Add to project 	Web 	        OSE web console, which provides a graphical workflow to define resources much alike oc new-app would create. The web console makes it easier to customize the resources created for the application using a wizard-like interface.

###### Troubleshooting a pod

A pod may be created but never become READY, and have the RESTARTS count increasing from time to time. This means the container image was successfully pulled from the registry, but the containers inside the pod failed to start, and OSE 3 is retrying to create the pod. The failure has to be investigated from inside the container image.

There are two ways to inspect an existing pod resource definition

*    The first one is using oc get, which exposes OSE 3 runtime attributes such as "creationTime" and temporary, dynamic IP addresses - oc get pod mypod -o json
*    A cleaner alternative is using oc export, which creates a resource definition file that could be used to replicate the resource later on in another OSE 3 installation - oc export pod mypod -o json


*    OSE 3 collects the container standard output and standard error streams so they can be displayed with oc logs - oc logs mypod
*    An OSE 3 user can run any command inside the pod using oc exec - oc exec -p mypod -it /bin/bash

> Option -p provides the pod name (and option -c provides the container name if there is more than one container inside the pod). Options -i and -t are needed for interactive terminal-based programs like the Bash shell.

```
# oc get pods
NAME           READY     STATUS    RESTARTS   AGE
test-1-cqoqf   1/1       Running   0          13m
# oc export pod test-1-cqoqf -o json > test-1-cqoqf.json
# oc logs test-1-cqoqf
---> Migrating database ...
Operations to perform:
  Synchronize unmigrated apps: staticfiles, debug_toolbar, messages
  Apply all migrations: admin, contenttypes, welcome, auth, sessions
Synchronizing apps without migrations:
  Creating tables...
    Running deferred SQL...
  Installing custom SQL...
Running migrations:
  Rendering model states... DONE
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying sessions.0001_initial... OK
  Applying welcome.0001_initial... OK
---> Serving application with gunicorn (wsgi) ...
[2017-02-21 10:12:49 +0000] [1] [INFO] Starting gunicorn 19.4.5
[2017-02-21 10:12:49 +0000] [1] [INFO] Listening at: http://0.0.0.0:8080 (1)
[2017-02-21 10:12:49 +0000] [1] [INFO] Using worker: sync
[2017-02-21 10:12:49 +0000] [30] [INFO] Booting worker with pid: 30
[2017-02-21 10:12:50 +0000] [31] [INFO] Booting worker with pid: 31
[2017-02-21 10:12:50 +0000] [32] [INFO] Booting worker with pid: 32
[2017-02-21 10:12:50 +0000] [33] [INFO] Booting worker with pid: 33
[2017-02-21 10:12:50 +0000] [34] [INFO] Booting worker with pid: 34
[2017-02-21 10:12:50 +0000] [35] [INFO] Booting worker with pid: 35
[2017-02-21 10:12:50 +0000] [36] [INFO] Booting worker with pid: 36
[2017-02-21 10:12:50 +0000] [37] [INFO] Booting worker with pid: 37
[2017-02-21 10:12:50 +0000] [39] [INFO] Booting worker with pid: 39
[2017-02-21 10:12:50 +0000] [40] [INFO] Booting worker with pid: 40
[2017-02-21 10:12:51 +0000] [41] [INFO] Booting worker with pid: 41
[2017-02-21 10:12:51 +0000] [44] [INFO] Booting worker with pid: 44
[2017-02-21 10:12:51 +0000] [43] [INFO] Booting worker with pid: 43
[2017-02-21 10:12:51 +0000] [47] [INFO] Booting worker with pid: 47
[2017-02-21 10:12:51 +0000] [45] [INFO] Booting worker with pid: 45
[2017-02-21 10:12:51 +0000] [49] [INFO] Booting worker with pid: 49

# oc port-forward -p mypod 1234:80
```

### Building a Pod



### Enabling Services for a Pod

###### Working with services

Services are essential resources to any OSE application. They allow containers in one pod to open network connections to containers in other pod without one pod interfering on other pod life cycles. A pod may be restarted for many reasons, and will get a different internal IP each time. Instead of a pod having to discover other pods internal container network IPs after each restart, a service provides a stable IP address for other pods to connect, no matter what node runs the pod after each restart.

Most real-world applications will not run as a single pod. They need to scale horizontally, so many pods run the same containers from the same pod resource definition to meet a growing user demand. A service is tied to a set of pods, providing a single IP address to the set and load-balancing client request among the member pods.

The set of pods running behind a service is managed by a DeploymentConfig resource. A DeploymentConfig resource embeds a ReplicationController that manages how many pod copies (replicas) have to be created, and creates new ones if some of them fail, whatever the reason. DeploymentConfig and ReplicationControllers will be detailed presented later in this chapter, together with the Source-To-Image (S2I) process.



### Creating Routes

