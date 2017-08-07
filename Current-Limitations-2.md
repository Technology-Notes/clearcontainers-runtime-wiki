# Clear Containers known differences and limitations

As Intel® Clear Containers utilises Virtual Machines (VM) to enhance security and isolation of container workloads, the `cc-runtime` has a number of differences and limitations when compared with the standard Docker runtime, `runc`. Some of these limitations have potential solutions, whereas others exist due to fundamental architectural differences generally related to the use of VMs.

The Clear Container runtime launches each container within its own hardware isolated VM, and each VM has its own kernel. Due to this higher degree of isolation, there are certain container capabilities that cannot be supported or are implicitly enabled via the VM.

The below sections describe in brief the known limitations, and where applicable further links off to the relevant open Issues or pages with more detailed information.

## Pending items

This section lists items that may technically be fixable:

### Networking

#### Adding networks dynamically

The runtime does not currently support adding networks to an already running container (`docker network connect`).

The VM network configuration is set up with what is defined by the CNM plugin at startup time. It would be possible to watch the networking namespace on the host to discover and propagate new networks at runtime but, it is not implemented today (tracked in issue [\#388](https://github.com/01org/cc-oci-runtime/issues/388)).

#### Support for joining an existing VM `docker run --net=containers`)

Docker supports the ability for containers to join another containers namespace. This allows multiple containers to share a common network namespace and the network interfaces placed in the network namespace. Clear Containers does not support network namespace sharing. If a Clear Container is setup to share the network namespace of a `runc` container, the runtime effectively takes over all the network interfaces assigned to the namespace and binds them to the VM. So the `runc` container will lose its network connectivity. 

### Resource management

Due to the way VMs differ in their CPU and memory allocation and sharing across the host system, the implementation of an equivalent method for these commands is potentially challenging.

#### `docker run -m`
The `docker run -m MEMORY` option is not currently supported. At the runtime level, this equates to the `linux.resources.memory` OCI configuration. It should be feasible to pass the relevant information through to the QEMU `-m` memory size option. This is also related to the `docker update` command.
[\#381](https://github.com/clearcontainers/runtime/issues/381)

#### `docker run --cpus=`
The `docker run --cpus=` option is not currently implemented. At the runtime level, this equates to the `linux.resources.cpu` OCI configuration. It should be possible to pass this information through to the QEMU command line CPU configuration options to gain a similar effect.
[\#341](https://github.com/clearcontainers/runtime/issues/341)

#### shm

The runtime does not implement the `docker run --shm-size` command to set the size of the `/dev/shm tmpfs` size within the container. It should be possible to pass this configuration value into the VM container and have the appropriate mount command happen at launch time.

#### cgroup constraints

Docker supports cgroup setup and manipulation generally through the `run` and `update` commands. With the use of VMs in Clear Containers, the mapping of cgroup functionality to VM functionality is not always straight forward.

For information on specific support, see the relevant sub-sections within this document.

Generally, support can come down to a number of methods:
- Implement support inside the VM/container
- Implement support wrapped around the VM/container
- Potentially a combination or sub-set of both inside and outside the VM/container
- No implementation necessary, as the VM naturally provides equivalent functionality

#### Capabilities

The `docker run --cap-[add|drop]` commands are not supported by the runtime. At the runtime level, this equates to the `linux.process.capabilities` OCI configuration. Similar to the cgroup items, these capabilities could be modified either in the host, in the VM, or potentially both.

[\#51](https://github.com/clearcontainers/runtime/issues/51) is related.

#### sysctl

The `docker run --sysctl` feature is not implemented. At the runtime level, this equates to the `linux.sysctl` OCI configuration. Docker allows setting of sysctl settings that support namespacing. It may make sense from a security and isolation pov to set them in the VM (effectively isolating sysctl settings). Also given that each Clear Container has its own kernel, we can support setting of sysctl settings that are not namespaced. In some cases, we may need to support setting some of the settings both on the host side Clear Container namespace as well as the Clear Containers kernel.

#### tmpfs

The `docker run --tmpfs` command is not supported by the runtime. Given the nature of a tmpfs, it should be feasible to implement this command as something passed through to the VM kernel startup in order to set up the appropriate mount point.

### Other

#### checkpoint and restore

The runtime does not provide `checkpoint` and `restore` commands. There have been discussions around using VM save and restore to give `criu` like functionality, and it is feasible that some workable solution may be achievable.

There is some more background and information in the `cc-oci-runtime` Issue [\#22](https://github.com/01org/cc-oci-runtime/issues/22).

Note that the OCI standard does not specify `checkpoint` and `restore` commands.

#### `docker stats`

The `docker stats` command does not return meaningful information for Clear Containers at present. This requires the runtime to support the `events` command.

Some thought needs to go into if we display information purely from within the VM, or if the information relates to the resource usage of the whole VM container as viewed from the host. The latter is likely more useful from a whole system point of view.

[\#200](https://github.com/clearcontainers/runtime/issues/200) is related.

Note that the OCI standard does not specify an `stats` command.

### runtime commands

#### ps command

The Clear Containers runtime does not currently support the `ps` command.

See [\#95](https://github.com/clearcontainers/runtime/issues/95) for more information.

Note, this is *not* the same as the `docker ps` command. The runtime ps command lists the processes running within a container. The `docker ps` command lists the containers themselves. The runtime `ps` command is invoked from `docker top`

Note that the OCI standard does not specify a `ps` command.

#### events command

The runtime does not currently implement the `events` command. We may not be able to perfectly match every sub-part of the `runc` events command, but we can probably do a subset, and maybe add some VM specific extensions.

See here for the [runc implementation](https://github.com/opencontainers/runc/blob/e775f0fba3ea329b8b766451c892c41a3d49594d/events.go)

See [\#379](https://github.com/clearcontainers/runtime/issues/379) for more information.

Note that the OCI standard does not specify an `events` command.

#### update command

The runtime does not currently implement the `update` command, and hence does not support some of the `docker update` functionality. Much of the `update` functionality is based around cgroup configuration.

It may be possible to implement some of the update functionality by adjusting cgroups either around the VM or inside the container VM itself, or possibly by some other VM functional equivalent. It needs more investigation.

Note that the OCI standard does not specify an `update` command.

See [\#380](https://github.com/clearcontainers/runtime/issues/380) for more details.

## Architectural limitations

This section lists items that may not be fixed due to fundamental architectural differences between 'soft containers' and those based on VMs.

### Networking

#### `docker --net=host`

Docker host network support (`docker --net=host run`) is not supported. It is not possible to directly access the host networking configuration from within the VM.

`--net=host` can still be used with `runc` containers and inter-mixed with with running `cc-runtime` containers, thus still enabling use of `--net=host` when necessary.

It should be noted, currently passing the `--net=host` option into a Clear Container may result in the Clear Container networking setup modifying, re-configuring and therefore possibly breaking the host networking setup. Do not use `--host=net` with Clear Containers.

#### `docker run --link`

The runtime does not support the `docker run --link` command. This command is now effectively deprecated by docker, so we have no intentions of adding support. Equivalent functionality can be achieved with the newer docker networking commands.

See more documentation at [docs.docker.com](https://docs.docker.com/engine/userguide/networking/default_network/dockerlinks/)

### Host resource sharing

#### `docker --device`

host device support (`docker run --device`) is not supported. This could be mapped to the quite similar QEMU `-device` option to enact device passthrough if that particular device is supported by VFIO, but the QEMU option requires precise knowledge of which sort of device is being mapped, and that information is not necessarily easily available from the docker `--device` command.

#### `docker -v /dev/...`

Docker volume support for devices (`docker run -v /dev/foo`) is not supported for similar reasons to the `--device` option. Note however that non-device file volume mounts are supported.

#### `docker run --privileged`

The `docker run --privileged` command is not supported in the runtime. There is no natural or easy way to grant the VM access to all of the host devices that this command would need to be complete.

`--privileged` can still be used with `runc` containers and inter-mixed with with running `cc-runtime` containers, thus still enabling use of `--privileged` when necessary.

### Other

#### Annotations

OCI Annotations are nominally supported, but the OCI specification is not clear on their purpose. Note that the annotations are not exposed inside the Clear Container.

### runtime commands

#### init command

The runtime does not implement the `init` command as it has no useful equivalent in the VM implementation, and appears to be primarily for `runc` internal use.

#### spec command

The runtime does not implement the `spec` command as the `runc` spec command generates a template .json specification file that is useable by the Clear Containers runtime. Addition of `spec` to the Clear Containers runtime would just be duplication that would likely always be playing catchup with `runc`.
