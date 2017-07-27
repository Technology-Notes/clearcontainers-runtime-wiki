The following documents the known limitations of `cc-runtime`:

# Missing docker features

- host network support (`docker --net=host run ...`)
- host device support (`docker run --device ...`)
- volume support for devices (`docker run -v /dev/foo ...`)
- shared memory size (`docker run --shm-size ...`)
- network connect (`docker network connect ...`)

