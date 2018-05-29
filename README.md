# Hybrid Deployments to the EDGE powered by Red Hat Openshift & Red Hat Enterprise Linux

This is the main project for the IoT Solution Demo: "<b>Hybrid Deployments to the EDGE powered by Red Hat Openshift & Red Hat Enterprise Linux</b>".

This solution is made on top of "IoT Summit Lab 2016" -> https://github.com/redhat-iot/Virtual_IoT_Gateway

We've taken this demo, originally built as standalone applications to run on a virtual machine or remote raspberry pi device, and we ported it to Openshift, making its middleware apps working on containers.
The main objective of this demo is to show how Red Hat Openshift Container Platform may help you building your applications that you'll run on the "edge" smart gateways.
The applications developed with Openshift are by default portable thanks to Containers technology, we then added some bit of Ansible thanks to the Openshift Service Catalog and Ansible Service Broker and we'll monitor the deployments through Cockpit, the RHEL's integrated web interface.

The architecture to which we'll refer during this demo is the following:
![Architecture](/images/arch.png)

As you can see we have a Datacenter, in which we develops our applications thanks to a Virtualization Layer (Red Hat Virtualization) and a PaaS (Openshift Container Platform).
The apps will flow from Datacenter to the Hub and Smart Gateways thanks to this flow:
![Development Flow](/images/development.png)


You'll find all the different application projects used in this demo below:
[Software_Sensor](https://github.com/alezzandro/iotgw_Software_Sensor)
[Routing_Service](https://github.com/alezzandro/iotgw_Routing_Service)
[BusinessRules_Service](https://github.com/alezzandro/iotgw_BusinessRules_Service)

## Prerequisites

First of all you'll need:
- a working Openshift Container Platform 3.9 with exposed registry ([look here for documentation](https://docs.openshift.com/container-platform/3.9/install_config/registry/securing_and_exposing_registry.html))
- an empty RHEL7 properly configured (see below for the steps)

All the brand new installation should come with a pre-configured Ansible Service Broker and Openshift Service Catalog.
<i>You'll not get it enabled in Minishift or CDK, please refer to their docs for instruction on enabling it.</i>

As said, you'll need an empty RHEL7 configured with the following steps:
```
# yum remove firewalld
# yum install -y iptables-services docker docker-python cockpit
# systemctl enable cockpit docker iptables
```

Before starting the needed services you have to add your Openshift externally exposed registry into '/etc/containers/registries.conf':
```
# cat /etc/containers/registries.conf

# This is a system-wide configuration file used to
# keep track of registries for various container backends.
# It adheres to TOML format and does not support recursive
# lists of registries.

# The default location for this configuration file is /etc/containers/registries.conf.

# The only valid categories are: 'registries.search', 'registries.insecure',
# and 'registries.block'.

[registries.search]
registries = ['registry.access.redhat.com']

# If you need to access insecure registries, add the registry's fully-qualified name.
# An insecure registry is one that does not have a valid SSL certificate or only does HTTP.
[registries.insecure]
registries = ['registry-default.apps.52.179.125.95.nip.io']


# If you need to block pull access from a registry, uncomment the section below
# and add the registries fully-qualified name.
#
# Docker only
[registries.block]
registries = []
```

Then you can start the services:
```
# systemctl start iptables cockpit docker
```


## Openshift Setup Steps

You can run the following commands for creating the base environment on Openshift side:
```
# git clone https://github.com/alezzandro/iotgw_mainproject
# oc new-project iot-development
# oc process -f project-iot-development.yaml | oc create -f
# oc new-project iot-testing
# oc process -f project-iot-testing.yaml | oc create -f
# oc new-project iot-hub
# oc process -f project-iot-hub.yaml | oc create -f
```

These commands will configure three Openshit's project:
- Development project with all the tools and pipeline for promoting containers in the Testing environment
- One dedicated for Testing, no building elements here, it receives updated containers from Dev env.
- A project dedicated to simulate the Hub Datacenter that usually is placed in the Factory.

After that we need to give to Jenkins' Service Account of the iot-development project the right of managing resources in the iot-testing project:
```
# oc policy add-role-to-user edit system:serviceaccount:iot-development:jenkins -n iot-testing
```

## Ansible Playbook Bundle Setup

We can now setup the Ansible Playbook Bundle, first of all we need to generate a SSH key that will be bundled in the APB:
```
# cd iotgw_mainproject/deploy-containers-apb
# ssh-keygen -f ./id_rsa
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in ./id_rsa.
Your public key has been saved in ./id_rsa.pub.
The key fingerprint is:
SHA256:3fBEgjbjMZh1+W2lkaDQSdqYpwwKecxbpUQ/yLbTBJk alex@lenny
The key's randomart image is:
+---[RSA 2048]----+
|     .+*+++oo. . |
|   + oE*BB++  o .|
|  o + BoB==... + |
|   o = *.= =. +  |
|    o o S . o.   |
|       .         |
|                 |
|                 |
|                 |
+----[SHA256]-----+

# ls id_rsa*
id_rsa  id_rsa.pub

# cat id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC1JONXBS1XQpx7fwU8ttL311/XhFO9l8qOWrPIw3D14Y/ZCgMUkzfnySCR+gSs9pmI+8mO3SizJ0bnwkCrd8y+BlSuVSIGN37cLKc04YMwFhk5aQRpvigaIIyHEp2eakUEEdE2qLs1WoYhRputVxLGsWqzdhdv6vX7CDncgcRDVmdxGANXBdcfn6H7CFYw9f5PP6GVsTQBO4negowBp6q6fN+o3/eoo03BwFBub7J6sWXPOb9txiE6yOMs4+h3SvcTYkPyxEoBPcCkwyb/MhvjTvJl3SDvE1IbcUrruBVfvkfEOQ9mRKj7ZEtrJIS4gwLEOodRMaHRhI2nG5F1wSO1 alex@lenny
```

After that we have to take the token for the default Service Account created in each project, named "builder" and paste it inside our Playbook:
```

```


On the Openshift side we need to enable the whitelist for the internal Openshift registry and let the Ansible Service Broker to scan for images ending with "-apb":
```

```

Finally we can start the build of the Ansible Playbook Bundle and proceed with the upload to the Openshift Registry:
```

```

We should see the just uploaded APB in the Service Catalog, as soon as we refresh the Openshift Service Catalog page, as shown in the image below:
![Ansible Playbook Bundle](/images/apb.png)


We can now take the public key and deploy it on our Smart Gateway RHEL7 based:
```
[alex@smartgw ~]$ mkdir -p .ssh
[alex@smartgw ~]$ echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC1JONXBS1XQpx7fwU8ttL311/XhFO9l8qOWrPIw3D14Y/ZCgMUkzfnySCR+gSs9pmI+8mO3SizJ0bnwkCrd8y+BlSuVSIGN37cLKc04YMwFhk5aQRpvigaIIyHEp2eakUEEdE2qLs1WoYhRputVxLGsWqzdhdv6vX7CDncgcRDVmdxGANXBdcfn6H7CFYw9f5PP6GVsTQBO4negowBp6q6fN+o3/eoo03BwFBub7J6sWXPOb9txiE6yOMs4+h3SvcTYkPyxEoBPcCkwyb/MhvjTvJl3SDvE1IbcUrruBVfvkfEOQ9mRKj7ZEtrJIS4gwLEOodRMaHRhI2nG5F1wSO1 alex@lenny" >> ,ssh/authorized_keys
[alex@smartgw ~]$ chmod 700 .ssh
[alex@smartgw ~]$ chmod 600 .ssh/authorized_keys
```

That's all! You now have an Openshift environment with all the necessary for showing the demo, consisting in:
1. Development environment with Jenkins' pipelines and three components:
   - Software-Sensor: that will simulate a temperature Sensor
   - Routing_Service: handling the communication and messages dispatching between different AMQ queues
   - BusinessRules_Service: that will filter the data coming from the Sensor, deciding when trigger an alarm (that will be a message on a dedicated queue)
2. Testing environment without Build components, that let you test the software and spawn the containers on a remote RHEL (via APB)
  - This is the right place where to execute the Ansible Playbook Bundle and trigger the remote deployments
3. Hub environment, that simulates the Factory's datacenter, containing for demo purposes only an AMQ container, handling just one queue where you receive the sensor alarm.
  - This is the backend for the containers you will deploy on the remote RHEL7
