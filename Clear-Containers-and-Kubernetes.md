# Overview

This documents how to get Kubernetes running with Clear Containers using CRIO.

# Base OS

Ubuntu 16.04 4.10.0-27-generic

# Known working commits

## CNI Plugins
github.com/containernetworking/plugins: commit dcf7368eeab15e2affc6256f0bb1e84dd46a34de

## runc
github.com/opencontainers/runc: commit e775f0fba3ea329b8b766451c892c41a3d49594d

## kubelet, kubeadm, kubectl: Kubernetes v1.6.7
Client Version: version.Info{Major:"1", Minor:"6", GitVersion:"v1.6.7", GitCommit:"095136c3078ccf887b9034b7ce598a0a1faff769"
kubeadm version: version.Info{Major:"1", Minor:"6", GitVersion:"v1.6.7", GitCommit:"095136c3078ccf887b9034b7ce598a0a1faff769"

## CRIO
github.com/kubernetes-incubator/cri-o: commit 9dbd60a0dfb8a517590ab3981408fc54fe400262

## Clear Containers

github.com/clearcontainers/proxy: commit b73e4a37c3ff01f087ee5efaf1409380810ea4ce
github.com/clearcontainers/runtime: commit cd98417d5f03f1081a4d0b181adcbefcd5ce7470
github.com/clearcontainers/shim: commit ab14648926c47d7ebb02e0adba3e95ffbd20765e


# Get go 1.8.3:
```
wget https://storage.googleapis.com/golang/go1.8.3.linux-amd64.tar.gz
sudo tar -xvf go1.8.3.linux-amd64.tar.gz -C /usr/local/
mkdir -p $HOME/go/src
export GOPATH=$HOME/go
export PATH=$PATH:/usr/local/go/bin:$GOPATH/bin
go version
```

# Build CRI-O from source:

```
sudo apt-get install -y \
  autoconf \
  btrfs-tools \
  git \
  libapparmor-dev \
  libassuan-dev \
  libdevmapper-dev \
  libglib2.0-dev \
  libc6-dev \
  libgpgme11-dev \
  libgpg-error-dev \
  libseccomp-dev \
  libselinux1-dev \
  pkg-config

sudo add-apt-repository ppa:alexlarsson/flatpak
sudo apt-get update

sudo apt-get install -y libostree-dev



go get -d github.com/kubernetes-incubator/cri-o
cd $GOPATH/src/github.com/kubernetes-incubator/cri-o
make install.tools
make
sudo make install
sudo make install.config
```

# Install latest RUNC:

```
go get github.com/opencontainers/runc
cd $GOPATH/src/github.com/opencontainers/runc
make
sudo make install
runc --version
```

This will install runc at `/usr/local/sbin/runc`.  CRIO expects it at `/usr/bin/runc`.  

Either move the binary to the expected path or modify `/etc/crio/crio.conf` to point to the installed location.

# Get CNI Plugins from source:

```

go get -d github.com/containernetworking/plugins
cd $GOPATH/src/github.com/containernetworking/plugins
git checkout dcf7368eeab15e2affc6256f0bb1e84dd46a34de
./build.sh
sudo mkdir -p /opt/cni/bin
sudo cp bin/* /opt/cni/bin/
 ```
 
 
# Install Clear Containers Runtime

For Clear Containers 3.0, follow directions available at https://github.com/clearcontainers/runtime/blob/master/docs/developers-clear-containers-install.md


# Modify CRI-O configuration to make use of Clear Containers:

Ensure the runc path is set to `/usr/local/sbin/runc`.  CRIO default is it at `/usr/bin/runc`.  

Modify ```/etc/crio/crio.conf``` to select ``cc-runtime`` as the ```runtime_untrusted_workload```
and set the ```default_workload_trust``` to ```untrusted```.

```
runtime_untrusted_workload = "/usr/local/bin/cc-runtime"
default_workload_trust = "untrusted"
```


# Start CRI-O System Daemon

Note the `Environment` parameters set below for proxy which will need to be updated if you are operating behind a proxy. 
```
# sh -c 'echo "[Unit]
Description=OCI-based implementation of Kubernetes Container Runtime Interface
Documentation=https://github.com/kubernetes-incubator/cri-o

[Service]
ExecStart=/usr/local/bin/crio --debug
Environment="HTTP_PROXY=http://myproxy.example.com:8080" "NO_PROXY=example.com,.example.com,localhost"
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/crio.service'
```

```
sudo systemctl daemon-reload
sudo systemctl enable crio
sudo systemctl start crio
sudo crioctl runtimeversion
```

# Setup Kubernetes to Use our CRI-O Setup

## Install Kubernetes

###  Install the 1.6.7 binaries:
```
# cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial-unstable main
EOF
# curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
# apt-get update
# apt-get install -y docker.io kubelet=1.6.7-00 kubeadm=1.6.7-00 kubectl=1.6.7-00 
# sudo apt-mark hold kubelet kubeadm kubectl

```

### Modify the default systemd file to make use of CRI:

Modify `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf` to add the following parameters to kubelet:
```
--container-runtime=remote --container-runtime-endpoint=/var/run/crio.sock --runtime-request-timeout=15m
```

An example for reference of the full file is:
```
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--kubeconfig=/etc/kubernetes/kubelet.conf --require-kubeconfig=true"
Environment="KUBELET_SYSTEM_PODS_ARGS=--pod-manifest-path=/etc/kubernetes/manifests --allow-privileged=true"
Environment="KUBELET_NETWORK_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin"
Environment="KUBELET_DNS_ARGS=--cluster-dns=10.96.0.10 --cluster-domain=cluster.local"
Environment="KUBELET_AUTHZ_ARGS=--authorization-mode=Webhook --client-ca-file=/etc/kubernetes/pki/ca.crt"
Environment="KUBELET_EXTRA_ARGS=--container-runtime=remote --container-runtime-endpoint=/var/run/crio.sock --runtime-request-timeout=30m"
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_SYSTEM_PODS_ARGS $KUBELET_NETWORK_ARGS $KUBELET_DNS_ARGS $KUBELET_AUTHZ_ARGS $KUBELET_EXTRA_ARGS
```


```
# systemctl daemon-reload
```

### Create network plugin configuration files (Flannel)

#### kube-flannel-rbac.yml
```
# Create the clusterrole and clusterrolebinding:
# $ kubectl create -f kube-flannel-rbac.yml
# Create the pod using the same namespace used by the flannel serviceaccount:
# $ kubectl create --namespace kube-system -f kube-flannel.yml
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: flannel
rules:
  - apiGroups:
      - ""
    resources:
      - pods
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes/status
    verbs:
      - patch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: flannel
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: flannel
subjects:
- kind: ServiceAccount
  name: flannel
  namespace: kube-system
```

#### kube-flannel.yml

```
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: flannel
  namespace: kube-system
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: kube-flannel-cfg
  namespace: kube-system
  labels:
    tier: node
    app: flannel
data:
  cni-conf.json: |
    {
      "name": "cbr0",
      "type": "flannel",
      "delegate": {
        "isDefaultGateway": true
      }
    }
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
---
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: kube-flannel-ds
  namespace: kube-system
  labels:
    tier: node
    app: flannel
spec:
  template:
    metadata:
      labels:
        tier: node
        app: flannel
    spec:
      hostNetwork: true
      nodeSelector:
        beta.kubernetes.io/arch: amd64
      tolerations:
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      serviceAccountName: flannel
      containers:
      - name: kube-flannel
        image: quay.io/coreos/flannel:v0.8.0-amd64
        command: [ "/opt/bin/flanneld", "--ip-masq", "--kube-subnet-mgr" ]
        securityContext:
          privileged: true
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        volumeMounts:
        - name: run
          mountPath: /run
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      - name: install-cni
        image: quay.io/coreos/flannel:v0.8.0-amd64
        command: [ "/bin/sh", "-c", "set -e -x; cp -f /etc/kube-flannel/cni-conf.json /etc/cni/net.d/10-flannel.conf; while true; do sleep 3600; done" ]
        volumeMounts:
        - name: cni
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      volumes:
        - name: run
          hostPath:
            path: /run
        - name: cni
          hostPath:
            path: /etc/cni/net.d
        - name: flannel-cfg
          configMap:
            name: kube-flannel-cfg
```

## Test Kubernetes end to end

```

#First Cleanup
export KUBECONFIG=/etc/kubernetes/admin.conf
sudo -E kubeadm reset
sudo systemctl stop kubelet
sudo systemctl stop docker
for c in `sudo crioctl ctr list | grep ^ID | cut -c5-`; do sudo crioctl ctr stop --id $c; sudo crioctl ctr remove --id $c ; done
for c in `sudo crioctl pod list | grep ^ID | cut -c5-`; do sudo crioctl pod stop --id $c; sudo crioctl pod remove --id $c ; done

sudo systemctl stop crio
sudo rm -rf /var/lib/cni/*
sudo rm -rf /var/run/crio/*
sudo rm -rf /var/log/crio/*
sudo rm -rf /var/lib/kubelet/*
sudo rm -rf /run/flannel/*
sudo rm -rf /etc/cni/net.d/*
sudo rm -rf /var/lib/virtcontainers/pods/*
sudo rm -rf /var/run/virtcontainers/pods/*
sudo ifconfig cni0 down
sudo ifconfig cbr0 down
sudo ifconfig flannel.1 down
sudo ifconfig docker0 down
sudo ip link del cni0
sudo ip link del cbr0
sudo ip link del flannel.1
sudo ip link del docker0


sudo systemctl start docker
sudo systemctl start crio
sudo systemctl start cc-proxy


mkdir -p /var/lib/cni
mkdir -p /var/run/crio
mkdir -p /var/log/crio
mkdir -p /var/lib/kubelet
mkdir -p /run/flannel
mkdir -p /etc/cni/net.d
sudo -E kubeadm init --pod-network-cidr 10.244.0.0/16
export KUBECONFIG=/etc/kubernetes/admin.conf

sudo -E kubectl get nodes
sudo -E kubectl get pods
sudo -E kubectl get pods --all-namespaces
sudo -E kubectl create -f kube-flannel-rbac.yml
sudo -E kubectl create --namespace kube-system -f kube-flannel.yml

#Now run a test pod
master=$(hostname)

#Allow scheduling on master
sudo -E kubectl taint nodes $master node-role.kubernetes.io/master:NoSchedule-

sudo -E kubectl run nginx --image=nginx --replicas=2
sudo -E kubectl expose deployment nginx --port=80
sudo -E kubectl get svc,pod

# echo "Run this command to test"
# echo "# wget --spider --timeout=1 nginx"
# sudo -E kubectl run busybox --rm -ti --image=busybox /bin/sh
echo "Verifing end to end connectivity"
echo "This takes a bit of time to complete as it waits for nginx to come up"
echo "On sucess you will see a response from nginx"
sudo -E kubectl run busybox --rm -ti --image=busybox -- /bin/sh -c "wget --spider --timeout=1 nginx"

```


## Cleanup

```
sudo -E kubeadm reset
for c in `sudo crioctl ctr list | grep ^ID | cut -c5-`; do sudo crioctl ctr stop --id $c; sudo crioctl ctr remove --id $c ; done
for c in `sudo crioctl pod list | grep ^ID | cut -c5-`; do sudo crioctl pod stop --id $c; sudo crioctl pod remove --id $c ; done
```
