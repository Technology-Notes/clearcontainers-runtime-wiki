# Clear Containers

# WIP

cc-runtime has a number of differences and limitations compared with standard Docker. Some of these limitations have solutions, whereas others exist due to fundamentally different architectural differences related to the use of Virtual Machines.

The below sections describe in brief the known limitations, and where applicable further link off to the relevant open Issues or pages with more detailed information.

## Pending items

This section lists items that should technically be fixable:

### Networking

#### Adding networks dynamically

The runtime doesn't support adding networks to an already running container (`docker network connect`).

We currently only setup the VM network configuration with what is defined by the CNM plugin at startup time. It would be possible to watch the networking namespace to discover and propagate new networks at runtime but it's not implemented today (tracked in issue [\#388](https://github.com/01org/cc-oci-runtime/issues/388)).

#### Support for joining an existing VM `docker run --net=containers`)

**FIXME**

What is this? Is this the same as adding networks dynamically??

### Resource management

The following items are not supported:
- Unconstrained memory and CPU containers
- `docker run -m MEMORY`
- `docker run --cpus=`
- `docker update`

Due to the way VMs differ in their CPU and memory allocation and sharing across the host system, the implementation of an equivalent method for these commands is potentially challenging.

Some commands are theoretically not too difficult (for instance, the `-m MEMORY` command could filter through to configure the  QEMU `-m` memory configuration line, as could the `--cpus` for instance.
For `docker update`, also see the `runtime commands` section of this document.

#### shm

The runtime does not implement the `docker run --shm-size` command to set the size of the /dev/shm tmpfs size within the container. It should be possible to pass this configuration value into the VM container and have the appropriate mount command happen at launch time.

#### cgroup constraints

(**FIXME** - how are these configured/requested - are these already covered by the sub-parts of 'docker update'?)
cgroup constraints are not currently applied to the workload. This is difficult problem since they constraints can be applied in various places including:

- The hypervisor process.
- The cc-shim process that represents the workload.
- The real workload running inside the virtual machine. There is work underway to solve this issue.

#### Capabilities

The 'docker run --cap-[add|drop]` commands are not supported by the runtime. Similar to the cgroup items, these capabilities could be modified either in the host, in the VM, or potentially both.

#### sysctl

The `docker run --sysctl` feature is not implemented. These kernel settings could take place on either the host or VM kernel. Given that sysctl settings are global (afaict they are not yet namespaced - if somebody knows differently then please post that info here), it may either make sense to just set them on the host kernel, or it may make sense from a security and isolation pov to set them in the VM (effectively isolating sysctl settings). Given this option is part of `run`, it almost feels like it should be 'per-container', and thus setting them in the VM may be a sensible option.

#### tmpfs

The `docker run --tmpfs` command is not supported by the runtime. Given the nature of a tmpfs, it should be feasible to implement this command as something passed through to the VM kernel startup in order to set up the appropriate mount point.

### Other

#### checkpoint and restore

There have been discussions around using VM save and restore to implement to give criu like functionality, and it is feasible that some workable solution may be achievable.

There is some more background and information in the `cc-oci-runtime` Issue [\#22](https://github.com/01org/cc-oci-runtime/issues/22)

#### `docker stats`

The `docker stats` command does not return meaningful information for Clear Container containers at present.
Some thought needs to go into if we display information pure from within the VM, or if the information relates to the resource usage of the whole VM container as based from the host. The latter is likely more useful from a whole system point of view.

### runtime commands

#### ps command

The Clear Containers runtime does not currently support the 'ps' command.

See [\#95](https://github.com/clearcontainers/runtime/issues/95) for more information.

Note, this is *not* the same as the `docker ps` command. The runtime ps command lists the processes running within a container. The `docker ps` command lists the containers themselves. The runtime ps command is invoked from `docker top`

#### events command

The runtime does not currently implement the `events` command. We may not be able to perfectly match every sub-part of the `runc` events command, but we can probably do a subset, and maybe add some VM specific extensions.

See here for the [runc implementaiton](https://github.com/opencontainers/runc/blob/e775f0fba3ea329b8b766451c892c41a3d49594d/events.go)

See [\#379](https://github.com/clearcontainers/runtime/issues/379) for more information.

#### update command

The runtime does not currently implement the `update` command, and hence does not support some of the `docker update` functionality. Much of the `update` functionality looks to be based around cgroup configuration.

It may be possible to implement some of the update functionality by adjusting cgroups either around the VM or inside the container VM itself, or possibly by some other VM functional equivalent. It needs more investigation.

See [\#380](https://github.com/clearcontainers/runtime/issues/380) for more details.

## Architectural limitations

This section lists items that may not be fixed due to fundamental architectural differences between 'soft containers' and those based on Virtual Machines.

### Networking

#### `docker --net=host`

Docker host network support (`docker --net=host run`) is not supported. It is not possible to directly access the host networking configuration from within the VM.

It should be noted, currently passing the `--net=host` option into a Clear Container may result in the Clear Container networking setup modifying, re-configuring and therefore possibly breaking the host networking setup. Do not use `--host=net` with Clear Containers.

#### `docker run --link`

The runtime does not support the `docker run --link` command. This command is now effectively deprecated by docker, so we have no intentions of adding support. Equivalent functionality can be achieved with the newer docker networking commands.

See more documents at [docs.docker.com](https://docs.docker.com/engine/userguide/networking/default_network/dockerlinks/)

### Host resource sharing

#### `docker --device`

host device support (`docker run --device`) is not supported. This could be mapped to the quite similar QEMU `-device` option to enact device passthrough, but the QEMU option requires precise knowledge of which sort of device is being mapped, and that information is not necessarily easily available from the docker `--device` command.

#### `docker -v /dev/...`

Docker volume support for devices (`docker run -v /dev/foo`) is not supported for similar reasons to the `--device` option. Note however that non-device file volume mounts are supported.

#### `docker run --privileged`

The `docker run --privileged` command is not supported in the runtime. There is no natural or easy way to grant the VM access to all of the host devices that this command would need to be complete. 

### Other

#### Annotations

OCI Annotations are nominally supported, but the OCI specification is not clear on their purpose. Note that the annotations are not exposed inside the Clear Container.

### runtime commands

#### init command

The runtime does not implement the `init` command as it has no useful equivalent in the VM implementation, and appears to be primarily for `runc` internal use.

#### spec command

The runtime does not implement the `spec` command as the `runc` spec command generates a template .json specification file that is useable by the Clear Containers runtime. Addition of `spec` to the Clear Containers runtime would just be duplication that would likely always be playing catchup with `runc`.
