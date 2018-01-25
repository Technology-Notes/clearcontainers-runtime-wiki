* [Assumptions](#assumptions)
* [Build and install Kata proxy](#build-and-install-kata-proxy)
* [Build and install Kata shim](#build-and-install-kata-shim)
* [Build and install Clear Containers runtime for Kata](#build-and-install-clear-containers-runtime-for-kata)
* [Enable full debug](#enable-full-debug)
* [Create an image](#create-an-image)
    * [Build the Kata agent](#build-the-kata-agent)
    * [Get the osbuilder](#get-the-osbuilder)
    * [Create a rootfs image](#create-a-rootfs-image)
    * [Add the agent to the image](#add-the-agent-to-the-image)
    * [Build the image](#build-the-image)
* [Install the image](#install-the-image)
* [Install guest kernel images](#install-guest-kernel-images)
* [Update Docker config](#update-docker-config)
* [Test](#test)

# Assumptions

- You already have Clear Containers [installed](https://github.com/clearcontainers/runtime/wiki/Installation).

# Build and install Kata proxy

```
go get -d github.com/kata-containers/proxy
cd $GOPATH/src/github.com/kata-containers/proxy && make && sudo make install
```

# Build and install Kata shim

```
go get -d github.com/kata-containers/shim
cd $GOPATH/src/github.com/kata-containers/shim && make && sudo make install
```

# Build and install Clear Containers runtime for Kata

```
go get -d github.com/clearcontainers/runtime
cd $GOPATH/src/github.com/clearcontainers/runtime
make build-kata-system && sudo -E PATH=$PATH make install-kata-system
```

# Enable full debug

```
sudo sed -i -e 's/^# *\(enable_debug\).*=.*$/\1 = true/g' /usr/share/defaults/kata-containers/configuration.toml
```

# Create an image

## Build the Kata agent

```
go get -d github.com/kata-containers/agent
cd $GOPATH/src/github.com/kata-containers/agent && make
```

## Get the osbuilder

```
go get -d github.com/kata-containers/osbuilder
```

## Create a rootfs image

```
cd $GOPATH/src/github.com/kata-containers/osbuilder/rootfs-builder
script -fec 'sudo -E GOPATH=$GOPATH ./rootfs.sh clearlinux'
```

## Add the agent to the image

```
sudo install -o root -g root -m 0755 -t rootfs/bin ../../agent/kata-agent
sudo install -o root -g root -m 0444 ../../agent/kata-containers.target rootfs/usr/lib/systemd/system/
sudo install -o root -g root -m 0444 ../../agent/kata-containers.target rootfs/usr/lib/systemd/system/
```

## Build the image

```
cd $GOPATH/src/github.com/kata-containers/osbuilder/image-builder
script -fec 'sudo ./image_builder.sh ../rootfs-builder/rootfs'
```

# Install the image

```
commit=$(git log --format=%h -1 HEAD)
date=$(date +%Y-%m-%d-%T.%N%z)

image="kata-containers-${date}-${commit}"
sudo install -o root -g root -m 0640 -D kata-containers.img "/usr/share/kata-containers/${image}"
(cd /usr/share/kata-containers && sudo rm -f kata-containers.img && sudo ln -s "$image" kata-containers.img)
```

# Install guest kernel images

```
sudo ln -s /usr/share/clear-containers/vmlinux.container /usr/share/kata-containers/
sudo ln -s /usr/share/clear-containers/vmlinuz.container /usr/share/kata-containers/
```

# Update Docker config

```
sudo sed -i 's!^\(ExecStart=[^$].*$\)!\1 --add-runtime kata-runtime=/usr/local/bin/kata-runtime!g' /etc/systemd/system/docker.service.d/clear-containers.conf
```

# Test

```
sudo docker run -ti --runtime kata-runtime busybox sh
```