Running Openshift on top of Clear Containers 3.0
=================

# Overview

This documents how to get Openshift 3.6 on top of Kubernetes running with Clear Containers.  An ansible playbook is used to provision a Fedora server 25 machine.  Clear Containers is then installed and a basic test of deploying Clear Container and runc based pods is detailed.

## Openshift

[OpenShift](https://developers.openshift.com/) is a public cloud application development and hosting platform which builds on top of the orchestration provided by [Kubernetes](https://kubernetes.io/docs/whatisk8s/).  

## Problem Statement 

A Docker controlled Clear Container will start one VM per container. Providing the kubernetes pod semantics with one VM per container is very challenging, especially from a networking standpoint. Instead Clear Containers should be able to start one VM per pod and launch containers within those VMs/Pods.

## Clear Containers as a runtime for untrusted workloads

We are going to describe how to deploy a single node Openshift cluster that uses Clear Containers as the container runtime for untrusted workloads and CRI-O as its CRI sever.

# System Setup

Provisioning of a Fedora machine with the software stack to deploy a generic single-node Openshift cluster is carried out using an ansible-playbook.  After this is carried out, additional changes to enable Clear Containers with CRI-O are described.

## Initial Machine Provisioning

For running an ansible-playbook, a host machine is used to ssh and execute setup commands on the target system.

### Target Setup

The machine which we will install Openshift needs ansible installed and password-less root acccess enabled from the system which will run the ansible-playbook.  

1. Provision the machine with Fedora Server 25
2. Install ansible and enable SSH access
 
```
sudo dnf update
sudo dnf install -y ansible openssh-server
```
 
### Host Setup

The host machine will be used to install the needed software on our Openshift target system

1.  Create a public key and enable password-less SSH onto the target system:

If you don't already have one, create a public/prive rsa key pair:
```
ssh-keygen -t rsa
```
Transfer key to the target system:
```
ssh root@(target-host-name) mkdir -p .ssh
cat .ssh/id_rsa.pub | ssh root@(target-host-name) 'cat >> .ssh/authorized_keys'
```
You should now be able to run ssh root@(target-host-name) without entering any credentials.  

Depending on the SSH version used, you may need to put the public key in .ssh/authorized_keys2, change permissions of .ssh to 700, and change permissions of .ssh/authorized_keys2 to 640.


2.  Get the Openshift Origin / CRI-O install ansible playbook

```
wget https://raw.githubusercontent.com/egernst/cri-o-origin-test/master/origin-cri-o.yml
```

3. Setup the ansible inventory file

Add the IP address of your openshift target system to ```/etc/ansible/hosts```. If you aren't using ansible for anything else, the IP is the only line needed in the file.

4.  Run the playbook

```
ansible-playbook origin-cri-o.yml
```

## Prepare the Openshift Target Machine for Clear Containers

At this point the system is configured to be able to run a generic Openshift stack.  In this section we will add Clear Containers and configure CRI-O.  The host system will not be used for the rest of the guide, and it is assumed you are working on the Openshift target machine.

### Install Clear Containers

For Clear Containers 3.0, follow directions available at https://github.com/clearcontainers/runtime/blob/master/docs/developers-clear-containers-install.md

Currently there is an issue with cc-runtime HEAD, so you will need to install from 2eeb4f49cb4a:

```
$ cd $GOPATH/src/github.com/clearcontainers/runtime
$ git checkout 2eeb4f49cb4a
$ make clean && make build-cc-system && sudo -E PATH=$PATH make install-cc-system
```

 
 ### Switch CRI-O to use Clear Containers for untrusted containers
 
 Edit /etc/crio/crio.conf to make use of the cc-runtime by updated the default workoad type providing an untrusted runtime:
 
 ```
runtime_untrusted_workload = "/usr/local/bin/cc-runtime"
default_workload_trust = "untrusted"
```

# Testing Clear Containers with Openshift
 
1. Prepare Target system: disable SELinux and flush the IP tables: 
```
setenforce 0
iptables --flush
 ```
2. Bring up the Openshift cluster with a single master and single CRI-O enabled node.

```
openshift start master --config /root/go/src/github.com/openshift/origin/openshift.local.config/master/master-config.yaml &> master.log &
openshift start node --config /root/go/src/github.com/openshift/origin/openshift.local.config/ <system's hostname> /node-config-crio.yaml &> node.log &
 export KUBECONFIG=/root/go/src/github.com/openshift/origin/openshift.local.config/master/admin.kubeconfig
```

3. Start an explicilty trusted pod

```
wget https://raw.githubusercontent.com/egernst/k8s-testing-scripts/master/nginx-trusted.yaml
oc create -f nginx-trusted.yaml
```
After a moment, observe that the trusted pod is started and running via runc

```
oc status
oc describe pod/nginx-trusted
runc list
```
Remove the pod
```
oc delete pod/nginx-trusted
```

4. Start an explicitly untrusted pod

```
wget https://raw.githubusercontent.com/egernst/k8s-testing-scripts/master/nginx-untrusted.yaml
oc create -f nginx-untrusted.yaml
```
After a moment, observe that the trusted pod is started and running via runc

```
oc status
oc describe pod/nginx-untrusted
cc-runtime list
```
Remove the pod
```
oc delete pod/nginx-untrusted
```
5. Start a generic deployment
```
oc run redis --image redis:alpine
```
After a moment, take a look at the deployment, and verify which runtime is used (it should match the runtime associated with the default workload trust level):
```
oc status
oc describe dc/redis
cc-runtime list
runc list
```
Remove the deployment
```
oc delete dc/redis
```

6. Cleanup

To cleanup, stop the node and master openshift processes