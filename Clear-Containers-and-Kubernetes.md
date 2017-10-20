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

Ubuntu 16.04 4.10.0-27-generic

# Known working versions

## CNI Plugins
github.com/containernetworking/plugins: commit 7f98c94613021d8b57acfa1a2f0c8d0f6fd7ae5a 
(HEAD of master at the time of this editing)

## runc
github.com/opencontainers/runc: commit 84a082bfef6f932de921437815355186db37aeb1 
(v1.0.0-rc4 should be fine)

## kubelet, kubeadm, kubectl: Kubernetes v1.7.8
```
$kubelet --version
Kubernetes v1.7.8

$kubectl version 
Client Version: version.Info{Major:"1", Minor:"7", GitVersion:"v1.7.8", GitCommit:"bc6162cc70b4a39a7f39391564e0dd0be60b39e9", GitTreeState:"clean", BuildDate:"2017-10-05T06:54:19Z", GoVersion:"go1.8.3", Compiler:"gc", Platform:"linux/amd64"}
$ kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"7", GitVersion:"v1.7.8", GitCommit:"bc6162cc70b4a39a7f39391564e0dd0be60b39e9", GitTreeState:"clean", BuildDate:"2017-10-05T06:35:40Z", GoVersion:"go1.8.3", Compiler:"gc", Platform:"linux/amd64"}

``` 

## CRIO
github.com/kubernetes-incubator/cri-o: v1.0.0

## Clear Containers

github.com/clearcontainers/proxy: v3.0.4

github.com/clearcontainers/runtime: v3.0.4

github.com/clearcontainers/shim: v3.0.4


# Installation Steps

1. go 1.8.3:
```
wget https://storage.googleapis.com/golang/go1.8.3.linux-amd64.tar.gz
sudo tar -xvf go1.8.3.linux-amd64.tar.gz -C /usr/local/
mkdir -p $HOME/go/src
export GOPATH=$HOME/go
export PATH=$PATH:/usr/local/go/bin:$GOPATH/bin
go version
```


2. Clone clearcontainers/tests repository
```
go get github.com/clearcontainers/tests
cd $GOPATH/src/github.com/clearcontainers/tests/integration/kubernetes
```

3. Execute the setup

    This step will install Kubernetes, CRI-O, Clear Containers components and will make
    correct configurations to run Kubernetes on top of Clear Containers.
```
./setup.sh
```
4. Initialize Kubernetes

```
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
