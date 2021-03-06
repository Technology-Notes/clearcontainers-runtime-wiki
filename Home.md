Welcome to the [Next Generation Intel® Clear Containers runtime](https://github.com/clearcontainers/runtime) wiki!

This wiki contains information to help you understand and use `cc-runtime` and its associated components.

Whilst the [main repository](/clearcontainers/runtime) contains a lot of [documentation](/clearcontainers/runtime/tree/master/docs), this wiki collects together useful information that may change frequently so is more appropriate to store here.

## Quickstart

- See the [installation pages](https://github.com/clearcontainers/runtime/wiki/Installation).

## System components

- Runtime (`cc-runtime`): https://github.com/clearcontainers/runtime
- Hypervisor (`qemu-lite`): https://github.com/clearcontainers/qemu
- Proxy process (`cc-proxy`): https://github.com/clearcontainers/proxy
- Process shim (`cc-shim`): https://github.com/clearcontainers/shim
- VM agent process (`cc-agent`): https://github.com/clearcontainers/agent
- Guest kernel: https://github.com/clearcontainers/linux

## Related repositories

- O/S builder: https://github.com/clearcontainers/osbuilder

  Creates a "mini O/S" image which is required by the hypervisor which boots it before switching to the workload.

- Test repository: https://github.com/clearcontainers/tests
- Packaging files: https://github.com/clearcontainers/packaging
- Jenkins CI configuration: https://github.com/clearcontainers/jenkins

## Clear Containers with Kubernetes

- Early preview of Clear Containers running with Kubernetes: https://github.com/clearcontainers/runtime/wiki/Clear-Containers-and-Kubernetes

[![asciicast](https://asciinema.org/a/134681.png)](https://asciinema.org/a/134681)

## External Links

- Clear Containers project page:

  https://clearlinux.org/containers

## Previous generation runtime

For information on the Intel® Clear Containers 2.x runtime (`cc-oci-runtime`), see:

- https://github.com/01org/cc-oci-runtime