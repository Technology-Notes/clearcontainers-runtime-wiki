The following documents the known limitations of `cc-runtime`:

## Sharing host resources

- host device support (`docker run --device ...`)
- volume support for devices (`docker run -v /dev/foo ...`)

## Networking

### Unsupported network types

- host network support (`docker --net=host run ...`)

This network type does not make sense when using a VM.

- Support for joining an existing VM `docker run --net=containers ...`)

### Adding networks dynamically

The runtime doesn't support adding networks to an already running container (`docker network connect ...`).

We currently only setup the VM network configuration with what is defined by the CNM plugin at startup time. It would be possible to watch the networking namespace to discover and propagate new networks at runtime but it's not implemented today (tracked in issue [\#388](https://github.com/01org/cc-oci-runtime/issues/388)).

## Annotations

OCI Annotations are nominally supported, but the OCI specification is not clear on their purpose. Note that the annotations are not exposed inside the Clear Container.

## Missing runtime commands

### `checkpoint` and `restore`

Although the runtime provides stub implementations of these commands, this is currently purely to satisfy Docker - the commands do *NOT* save/restore the state of the Clear Container.

### `ps` command

See https://github.com/clearcontainers/runtime/issues/95.

## Others

- `events`
- `init`
- `spec`
- `update`

## CPU and memory

The following items are not supported:

-   Unconstrained memory and CPU containers
-   `docker run -m MEMORY`
-   `docker run --cpus=`
-   `docker update`

## Resource Management (container QoS)

- shared memory size (`docker run --shm-size ...`)
