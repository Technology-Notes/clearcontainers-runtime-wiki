Running Kubernetes on top of Clear Containers 3.0
=================

# Overview

This documents how to get Kubernetes running with Clear Containers using CRIO.

## Kubernetes
[Kubernetes](https://kubernetes.io/docs/whatisk8s/) is a Google project and the dominant container orchestration engine.

Kubernetes clusters run [containers pods](https://kubernetes.io/docs/user-guide/pods/). Inside a pod, all containers share the pod resources (networking, storage, etc...) and all pods within a cluster have their own IP address.

By default Kubernetes runs the full Docker stack to start pods and containers within a pod. Rkt is an alternative container runtime for Kubernetes.

## Problem statement
A Docker controlled Clear Container will start one VM per container. Providing the kubernetes pod semantics with one VM per container is very challenging, especially from a networking standpoint.
Instead Clear Containers should be able to start one VM per pod and launch containers within those VMs/Pods.

## Solution
With the recent addition of [the Container Runtime Interface (CRI)](http://blog.kubernetes.io/2016/12/container-runtime-interface-cri-in-kubernetes.html) to Kubernetes, Clear Containers can now be controlled by any OCI compatible CRI implementation and get passed container annotations to let it know when and how to run Pod VMs or container workload within those VMs.

[CRI-O](https://github.com/kubernetes-incubator/cri-o) is the main OCI compatible CRI implementation and now supports Clear Containers.

## Clear Containers as a bare metal runtime for Kubernetes

We are going to describe how to deploy a bare metal Kubernetes cluster that uses Clear Containers as its container runtime and CRI-O as its CRI server.


# Base OS

Ubuntu 16.04 LTS

# Known working versions:

## CNI Plugins
github.com/containernetworking/plugins: v0.7.0 


## runc
github.com/opencontainers/runc: v1.0.0-rc4

## kubelet, kubeadm, kubectl: 1.9.2
```
$kubelet --version
Kubernetes v1.9.2

$kubectl version 
Client Version: version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.2", GitCommit:"5fa2db2bd46ac79e5e00a4e6ed24191080aa463b", GitTreeState:"clean", BuildDate:"2018-01-18T10:09:24Z", GoVersion:"go1.9.2", Compiler:"gc", Platform:"linux/amd64"}

$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"9", GitVersion:"v1.9.2", GitCommit:"5fa2db2bd46ac79e5e00a4e6ed24191080aa463b", GitTreeState:"clean", BuildDate:"2018-01-18T09:42:01Z", GoVersion:"go1.9.2", Compiler:"gc", Platform:"linux/amd64"}

``` 

## CRIO
github.com/kubernetes-incubator/cri-o: v1.9.0

## Clear Containers

github.com/clearcontainers/proxy: v3.0.21

github.com/clearcontainers/runtime: v3.0.21

github.com/clearcontainers/shim: v3.0.21


# Installation Steps

1. go 1.9.2:
```
wget https://storage.googleapis.com/golang/go1.9.2.linux-amd64.tar.gz
sudo tar -xvf go1.9.2.linux-amd64.tar.gz -C /usr/local/
mkdir -p $HOME/go/src
export GOPATH=$HOME/go
export PATH=$PATH:/usr/local/go/bin:$GOPATH/bin
go version
```


2. Clone clearcontainers/tests repository
```
go get github.com/clearcontainers/tests
```

3. Execute the setup

    This step will install Kubernetes, CRI-O, Clear Containers components and will make
    correct configurations to run Kubernetes on top of Clear Containers.
```
cd $GOPATH/src/github.com/clearcontainers/tests
.ci/setup.sh
```
4. Initialize Kubernetes

```
cd $GOPATH/src/github.com/clearcontainers/tests/integration/kubernetes
./init.sh 
```

5. Test your Kubernetes implementation:

```
bats nginx.bats 
```

6. Cleanup environment (optional)

    To stop Kubernetes services and clean all containers/pods created, execute:
```
./cleanup_env.sh
```
