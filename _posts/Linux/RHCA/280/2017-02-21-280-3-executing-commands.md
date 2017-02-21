---
layout: post
title:  "Executing Commands(280-3)"
categories: Linux
tags: RHCA 280
---

### OC 

```
# oc help
OpenShift Client 

This client helps you develop, build, deploy, and run your applications on any OpenShift or Kubernetes compatible
platform. It also includes the administrative commands for managing a cluster under the 'adm' subcommand.

Basic Commands:
  types           An introduction to concepts and types
  login           Log in to a server
  new-project     Request a new project
  new-app         Create a new application
  status          Show an overview of the current project
  project         Switch to another project
  projects        Display existing projects
  explain         Documentation of resources
  cluster         Start and stop OpenShift cluster
  idle            Idle scalable resources

Build and Deploy Commands:
  rollout         Manage a Kubernetes deployment or OpenShift deployment config
  deploy          View, start, cancel, or retry a deployment
  rollback        Revert part of an application back to a previous deployment
  new-build       Create a new build configuration
  start-build     Start a new build
  cancel-build    Cancel running, pending, or new builds
  import-image    Imports images from a Docker registry
  tag             Tag existing images into image streams

Application Management Commands:
  get             Display one or many resources
  describe        Show details of a specific resource or group of resources
  edit            Edit a resource on the server
  set             Commands that help set specific features on objects
  label           Update the labels on a resource
  annotate        Update the annotations on a resource
  expose          Expose a replicated application as a service or route
  delete          Delete one or more resources
  scale           Change the number of pods in a deployment
  autoscale       Autoscale a deployment config, deployment, replication controller, or replica set
  secrets         Manage secrets
  serviceaccounts Manage service accounts in your project

Troubleshooting and Debugging Commands:
  logs            Print the logs for a resource
  rsh             Start a shell session in a pod
  rsync           Copy files between local filesystem and a pod
  port-forward    Forward one or more local ports to a pod
  debug           Launch a new instance of a pod for debugging
  exec            Execute a command in a container
  proxy           Run a proxy to the Kubernetes API server
  attach          Attach to a running container
  run             Run a particular image on the cluster

Advanced Commands:
  adm             Tools for managing a cluster
  create          Create a resource by filename or stdin
  replace         Replace a resource by filename or stdin
  apply           Apply a configuration to a resource by filename or stdin
  patch           Update field(s) of a resource using strategic merge patch
  process         Process a template into list of resources
  export          Export resources so they can be used elsewhere
  extract         Extract secrets or config maps to disk
  observe         Observe changes to resources and react to them (experimental)
  policy          Manage authorization policy
  convert         Convert config files between different API versions
  import          Commands that import applications

Settings Commands:
  logout          End the current server session
  config          Change configuration files for the client
  whoami          Return information about the current session
  completion      Output shell completion code for the given shell (bash or zsh)

Other Commands:
  help            Help about any command
  version         Display client and server versions

Use "oc <command> --help" for more information about a given command.
Use "oc options" for a list of global command-line options (applies to all commands).

```

```
# oc login https://master.podX.example.com:8443 -u student -p redhat

# oc login https://open.paas.redhat.com
Authentication required for https://open.paas.redhat.com:443 (openshift)
Username: ftan
Password: 
Login successful.

You have one project on this server: "ethel"

Using project "ethel".

# oc whoami
ftan

# oc new-project working
# oc delete project working
# oc status
In project ethel - account-manager-stage.app.eng.rdu2.redhat.com (ethel) on server https://open.paas.redhat.com:443

svc/glusterfs-cluster - 172.30.108.143:24007

http://test-ethel.int.open.paas.redhat.com to pod port 8080-tcp (svc/test)
  dc/test deploys istag/test:latest 
    bc/test source builds https://github.com/openshift/django-ex.git#master on openshift/python:2.7 
    deployment #1 deployed 2 hours ago - 1 pod

3 warnings identified, use 'oc status -v' to see details.


# oc logout
Logged "ftan" out on "https://open.paas.redhat.com:443"
```


### Configuring Resources with the CLI


*    oc get RESOURCE_TYPE RESOURCE_NAME. Typically, as an administrator goes about the daily routine, the oc get command is likely the tool that is used most frequently. This allows a user to get information about resources in the cluster. Generally, this command outputs only the most important characteristics of the resource in a tabulated output and omits any of the more detailed information.
*    oc get RESOURCE_TYPE. If the RESOURCE_NAME parameter is omitted, then all resources of the specified RESOURCE_TYPE is summarized. The following output is a sample of an execution of oc get pods:

```

# oc get pods
NAME           READY     STATUS      RESTARTS   AGE
test-1-0dxia   1/1       Running     0          2h
test-1-build   0/1       Completed   0          2h

# oc get all
NAME      TYPE      FROM         LATEST
bc/test   Source    Git@master   1

NAME                               TYPE      FROM          STATUS     STARTED        DURATION
builds/account-management-tool-1   Source    Git@master    Failed     18 hours ago   3m41s
builds/test-1                      Source    Git@8c20b55   Complete   2 hours ago    2m38s

NAME      DOCKER REPO                      TAGS      UPDATED
is/test   172.30.114.236:5000/ethel/test   latest    2 hours ago

NAME      REVISION   DESIRED   CURRENT   TRIGGERED BY
dc/test   1          1         1         config,image(test:latest)

NAME        DESIRED   CURRENT   READY     AGE
rc/test-1   1         1         1         2h

NAME          HOST/PORT                             PATH      SERVICES   PORT       TERMINATION
routes/test   test-ethel.int.open.paas.redhat.com             test       8080-tcp   

NAME                    CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
svc/glusterfs-cluster   172.30.108.143   <none>        24007/TCP   19h
svc/test                172.30.199.48    <none>        8080/TCP    2h

NAME              READY     STATUS      RESTARTS   AGE
po/test-1-0dxia   1/1       Running     0          2h
po/test-1-build   0/1       Completed   0          2h

# List all pods in ps output format.
# oc get pods
  
# List a single replication controller with specified ID in ps output format.
# oc get rc redis

# List all pods and show more details about them.
# oc get -o wide pods

# List a single pod in JSON output format.
# oc get -o json pod redis-pod

# Return only the status value of the specified pod.
# oc get -o template pod redis-pod --template={{.currentState.status}}

```

*    oc describe RESOURCE RESOURCE_NAME. When the summaries provided by oc get are insufficient, additional detailed information about the resource can be retrieved by using the oc describe command. Unlike the oc get command, there is no way to simply iterate through all of the different resources by type. Though most major resources can be described, this functionality is not available across all resources. Following is an example output from describing a pod resource: 

```
# oc describe svc/test
Name:			test
Namespace:		ethel
Labels:			app=test
Selector:		deploymentconfig=test
Type:			ClusterIP
IP:			172.30.199.48
Port:			8080-tcp	8080/TCP
Endpoints:		10.1.19.171:8080
Session Affinity:	None
No events.

# Provide details about the ruby-22-centos7 image repository
# oc describe imageRepository ruby-22-centos7

# Provide details about the ruby-sample-build build configuration
# oc describe bc ruby-sample-build
```

*    oc export. This command can be used to export a definition of a resource. Typical use cases include backup or to aid in modification of a definition. By default, the export command prints out the object representation in YAML format, but this can be changed by providing a -o option.
*    oc create. This command allows the user to create resources from a resource definition. Typically, this is paired with the oc exportcommand for editing definitions.
*    oc delete RESOURCE_TYPE name. The oc delete command allows the user to remove a resource from the OSE cluster. Note that a fundamental understanding of the OpenShift architecture is needed here, as deletion of managed resources will unexpectedly result in newer instances of those resources being created. For example, deleted pods that are backed by replication containers may automatically redeploy in an unintended manner.
*    oc types. For a quick refresher on the concepts and types available through the oc command, use oc types.

```
# oc types
Concepts and Types 

Kubernetes and OpenShift help developers and operators build, test, and deploy applications in a containerized cloud
environment. Applications may be composed of all of the components below, although most developers will be concerned
with Services, Deployments, and Builds for delivering changes. 

Concepts: 

* Containers:
    A definition of how to run one or more processes inside of a portable Linux
    environment. Containers are started from an Image and are usually isolated
    from other containers on the same machine.
    
* Image:
    A layered Linux filesystem that contains application code, dependencies,
    and any supporting operating system libraries. An image is identified by
    a name that can be local to the current cluster or point to a remote Docker
    registry (a storage server for images).
    
* Pods [pod]:
    A set of one or more containers that are deployed onto a Node together and
    share a unique IP and Volumes (persistent storage). Pods also define the
    security and runtime policy for each container.
    
* Labels:
    Labels are key value pairs that can be assigned to any resource in the
    system for grouping and selection. Many resources use labels to identify
    sets of other resources.
    
* Volumes:
    Containers are not persistent by default - on restart their contents are
    cleared. Volumes are mounted filesystems available to Pods and their
    containers which may be backed by a number of host-local or network
    attached storage endpoints. The simplest volume type is EmptyDir, which
    is a temporary directory on a single machine. Administrators may also
    allow you to request a Persistent Volume that is automatically attached
    to your pods.
    
* Nodes [node]:
    Machines set up in the cluster to run containers. Usually managed
    by administrators and not by end users.
    
* Services [svc]:
    A name representing a set of pods (or external servers) that are
    accessed by other pods. The service gets an IP and a DNS name, and can be
    exposed externally to the cluster via a port or a Route. It's also easy
    to consume services from pods because an environment variable with the
    name <SERVICE>_HOST is automatically injected into other pods.
    
* Routes [route]:
    A route is an external DNS entry (either a top level domain or a
    dynamically allocated name) that is created to point to a service so that
    it can be accessed outside the cluster. The administrator may configure
    one or more Routers to handle those routes, typically through an Apache
    or HAProxy load balancer / proxy.
    
* Replication Controllers [rc]:
    A replication controller maintains a specific number of pods based on a
    template that match a set of labels. If pods are deleted (because the
    node they run on is taken out of service) the controller creates a new
    copy of that pod. A replication controller is most commonly used to
    represent a single deployment of part of an application based on a
    built image.
    
* Deployment Configuration [dc]:
    Defines the template for a pod and manages deploying new images or
    configuration changes whenever those change. A single deployment
    configuration is usually analogous to a single micro-service. Can support
    many different deployment patterns, including full restart, customizable
    rolling updates, and fully custom behaviors, as well as pre- and post-
    hooks. Each deployment is represented as a replication controller.
    
* Build Configuration [bc]:
    Contains a description of how to build source code and a base image into a
    new image - the primary method for delivering changes to your application.
    Builds can be source based and use builder images for common languages like
    Java, PHP, Ruby, or Python, or be Docker based and create builds from a
    Dockerfile. Each build configuration has web-hooks and can be triggered
    automatically by changes to their base images.
    
* Builds [build]:
    Builds create a new image from source code, other images, Dockerfiles, or
    binary input. A build is run inside of a container and has the same
    restrictions normal pods have. A build usually results in an image pushed
    to a Docker registry, but you can also choose to run a post-build test that
    does not push an image.
    
* Image Streams and Image Stream Tags [is,istag]:
    An image stream groups sets of related images under tags - analogous to a
    branch in a source code repository. Each image stream may have one or
    more tags (the default tag is called "latest") and those tags may point
    at external Docker registries, at other tags in the same stream, or be
    controlled to directly point at known images. In addition, images can be
    pushed to an image stream tag directly via the integrated Docker
    registry.
    
* Secrets [secret]:
    The secret resource can hold text or binary secrets for delivery into
    your pods. By default, every container is given a single secret which
    contains a token for accessing the API (with limited privileges) at
    /var/run/secrets/kubernetes.io/serviceaccount. You can create new
    secrets and mount them in your own pods, as well as reference secrets
    from builds (for connecting to remote servers) or use them to import
    remote images into an image stream.
    
* Projects [project]:
    All of the above resources (except Nodes) exist inside of a project.
    Projects have a list of members and their roles, like viewer, editor,
    or admin, as well as a set of security controls on the running pods, and
    limits on how many resources the project can use. The names of each
    resource are unique within a project. Developers may request projects
    be created, but administrators control the resources allocated to
    projects.
    
For more, see https://docs.openshift.com

Usage:
  oc types [options]

Examples:
  # View all projects you have access to
  oc get projects
  
  # See a list of all services in the current project
  oc get svc
  
  # Describe a deployment configuration in detail
  oc describe dc mydeploymentconfig
  
  # Show the images tagged into an image stream
  oc describe is ruby-centos7

Use "oc options" for a list of global command-line options (applies to all commands).

```


### Guided Exercise: Managing an OpenShift instance using oc



### Managing Security Policies


OpenShift Enterprise (OSE) manages authorization policies for each login using authorization policies. Authorization policies are responsible for determining whether a developer is allowed to perform a given action within a project. There are two levels of authorization policy:

*    Cluster policy: Controls who has various access levels to the OpenShift platform and all projects. Roles that exist in the cluster policy are considered cluster roles.
*    Local policy: Controls which developers have access to their projects. Roles that exist only in a local policy are considered local roles. 

Authorization is managed using:

*    Rules: Sets of permitted verbs on a set of resources; for example, whether someone can delete projects.
*    Roles: Collections of rules. Users and groups can be bound to multiple roles at the same time.
*    Binding: Associations between users and/or groups with a role.

OpenShift provides, by default, seven roles:

*    Role 	            Description
*    admin 	            The project owner. If used in a local binding, an admin user will have rights to view and modify any resource in the project, except for role creation and quota. If the cluster-admin wants to allow an admin to modify roles, project-scoped Policy object using JSON must be created.
*    basic-user 	        A user that can get basic information about projects and users.
*    cluster-admin 	    A user that may perform any action in any project. A user within a local policy will have full control over quota and roles and on every action in any resource from the project.
*    cluster-status 	    A user that can get basic cluster status information.
*    edit 	            A user that can modify most objects in a project, but does not have the power to view or modify roles or bindings.
*    self-provisioner 	A user that can create their own projects.
*    view 	            A user that cannot make any modifications, but can see most objects in a project. They cannot view or modify roles or bindings.


```
# Viewing cluster policy. A matrix of the verbs and resources associated with these roles can be visualized in the cluster policy with the following command:
# oc describe clusterPolicy default
Name:				 default
Created:			    4 hours ago
Labels:			     <none>
Last Modified:		     2015-10-05 17:22:25 +0000 UTC
admin				Verbs					                             Resources  ...
				     [create delete get list update watch]  [pods/proxy...
				     [get list watch]			                    [pods/exec ...
				     [get update]				                       [imagestrea...
basic-user			Verbs					                         Resource  ...
... OUTPUT OMITTED ...


# To view the current set of cluster bindings, which shows the users and groups that are bound to various roles:
# oc describe clusterPolicyBindings :default 
Name:						:default
Created:					    4 hours ago
Labels:						<none>
Last Modified:					2015-10-05 17:22:26 +0000 UTC
Policy:						<none>
RoleBinding[basic-users]:
						Role:	basic-user
						Users:	[]
						Groups:	[system:authenticated]
RoleBinding[cluster-admins]:
						Role:	cluster-admin
						Users:	[]
						Groups:	[system:cluster-admins]
... OUTPUT OMITTED ...


# Viewing local policy. The list of local roles and their associated rule sets are not visible within a local policy. The local bindings, however, can be listed using the following command:
# oc describe policyBindings :default 

# The current project is used when viewing local policy by default. The flag -n can be used to specify an alternative project:
# oc describe policyBindings :default -n myproject
```

###### Managing role bindings

The oadm policy command is responsible to add and remove roles to and from users and groups. Adding a role to users or groups gives the user or group the relevant access granted by the role.

By default, the current project is used when describing the local policy. The flag -n can be used to specify an alternative project. The following commands are available for local policies:

*    Command 	                                            Description
*    oadm policy who-can <verb> <resource> 	                List which users or groups can perform an action on a resource.
*    oadm policy add-role-to-user <role> <username> 	    Bind a role to specified users in the current project.
*    oadm policy remove-role-from-user <role> <username> 	Remove a role from specified users in the current project.
*    oadm policy remove-user <username> 	                Remove specified users and all of their roles in the current project.
*    oadm policy add-role-to-group <role> <groupname> 	    Bind a role to specified groups in the current project.
*    oadm policy remove-role-from-group <role> <groupname> 	Remove a role from specified groups in the current project.
*    oadm policy remove-group <groupname> 	                Remove specified groups and all of their roles in the current project.

```
# By default, the user that creates a project will have a bind with the admin role. The following command will add a user called student to the admin role in the example project:
# oadm policy add-role-to-user admin student -n example

# The student user can be removed from the admin role in the example project with the following command:
# oadm policy remove-user student -n example
```

The cluster policy does not accept the -n flag because the it uses non-namespaced resources. The following commands are available for cluster policies:

*    Command 	                                                    Description
*    oadm policy add-cluster-role-to-user <role> <username> 	    Bind a role to specified users for all projects in the cluster.
*    oadm policy remove-cluster-role-from-user <role> <username> 	Remove a role from specified users for all projects in the cluster.
*    oadm policy add-cluster-role-to-group <role> <groupname> 	    Bind a role to specified groups for all projects in the cluster.
*    oadm policy remove-cluster-role-from-group <role> <groupname> 	Remove a role from specified groups for all projects in the cluster.

```
# The following command is responsible to add the student user to the cluster-admin role:
# oadm policy add-cluster-role-to-user cluster-admin student
```












