# Installing  Clear Containers 3.0 Alpha on Fedora

## Introduction

Clear Containers **3.0-alpha** is available for Fedora\* versions **24** and **25**.

## Install latest Docker

This step is only required in case Docker is not installed on the system.

```
$ sudo dnf -y install dnf-plugins-core
$ sudo dnf config-manager \
    --add-repo \
    https://download.docker.com/linux/fedora/docker-ce.repo
$ sudo dnf makecache fast
$ sudo dnf install docker-ce
```

For more information on installing Docker please refer to the
[Docker Guide](https://docs.docker.com/engine/installation/linux/fedora)

## Install Clear Containers 3.0 components

```
$ source /etc/os-release
$ sudo -E dnf config-manager --add-repo \
http://download.opensuse.org/repositories/home:/clearcontainers:/clear-containers-3-staging/Fedora\_$VERSION_ID/home:clearcontainers:clear-containers-3-staging.repo
$ sudo dnf install cc-runtime cc-proxy cc-shim virtcontainers-pause
```

## Configure docker to use Clear Containers by default
```
sudo mkdir -p /etc/systemd/system/docker.service.d/
cat << EOF | sudo tee /etc/systemd/system/docker.service.d/clr-containers.conf
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -D --add-runtime cc3=/usr/bin/cc-oci-runtime --default-runtime=cc3
EOF
```

## Restart the docker and Clear Containers systemd services

```
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl enable cc-proxy.socket
sudo systemctl start cc-proxy.socket
```

## Run a Clear Container
You are now ready to run Clear Containers. For example:

```
sudo docker run -ti fedora bash
```