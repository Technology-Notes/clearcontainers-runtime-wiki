* :baby_symbol: [Basics](#basics)
  * [Create a Clear Container](#create-a-clear-container)
  * [Environment](#environment)
* :biking_man: [Intermediate](#intermediate)
* :racing_car:  [Advanced](#advanced)
___

If you are new to Clear Containers, try working through the following tasks. They should help you to become familiar with the system.

# Preliminaries

- Read the project's [README](https://github.com/clearcontainers/runtime/blob/master/README.md) carefully.
- Read the [architecture document](https://github.com/clearcontainers/runtime/blob/master/docs/architecture/architecture.md) for a technical overview and details of the system components.

# :baby_symbol: Basics

## Create a Clear Container

1. [[Install|installation]] Clear Containers.
1. Check that Clear Containers has installed, and is available in `docker` (hint: `docker info`).
1. Create a Clear Container (hint: `docker run` as normal! :)
1. Prove that you are running a Clear Container (hints: `uname -r` and `ps aux | grep qemu`).

## Environment

1. Run `sudo cc-runtime cc-check` and seeing if you understand all the output

   (see https://github.com/clearcontainers/runtime#hardware-requirements).

1. Run `sudo cc-runtime cc-env` and seeing if you understand all the output

   (see https://github.com/clearcontainers/runtime#configuration).

# :biking_man: Intermediate

1. [Enable debugging](https://github.com/clearcontainers/runtime#debugging).
1. Find a way to force an error
   (hint: Try adding spurious data into the config file).
1. Ensure you get a useful error message returned to you.
1. See if you can find evidence of the error you forced [in the logs]
   (https://github.com/clearcontainers/runtime#logging).

# :racing_car: Advanced

:information_source: Note:

This section assumes you have read the [architecture document](https://github.com/clearcontainers/runtime/blob/master/docs/architecture/architecture.md).

Clear Containers uses a small root filesystem (`rootfs`) image (which can be based on any Linux distro) and a guest kernel image to boot the virtual machine. If you wish, you can generate your own custom `rootfs` and / or custom `kernel`. This exercise gets you to create a custom `rootfs` image and boot a Clear Container using it.

1. Create a new `rootfs` image using [osbuilder](https://github.com/clearcontainers/osbuilder).
1. Read the [osbuilder](https://github.com/clearcontainers/osbuilder) docs to establish how to modify your runtime configuration to use the new `rootfs` image.
1. Start a Clear Container using your new `rootfs` image.
1. Find a way to prove that you are using your new image.

If you come across any issues or you have thoughts on how we can improve the system, **please** [raise an issue](https://github.com/clearcontainers/runtime/issues/new) so we can help! :smile: