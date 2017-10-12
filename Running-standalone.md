It is possible to run `cc-runtime` without a container manager such as `docker` (+`containerd`) or `kubernetes`.

## Create an OCI bundle

```bash
$ bundle="/tmp/bundle"
$ rootfs="$bundle/rootfs"
$ mkdir -p "$rootfs" && (cd "$bundle" && sudo docker-runc spec)
$ sudo docker export $(sudo docker create busybox) | tar -C "$rootfs" -xvf -
```

## Create a Clear Container

Create a container called `foo`:

```bash
$ sudo cc-runtime run --bundle "$bundle" foo
```

You should now be running a busybox shell. In a different terminal, try running:

```bash
$ sudo cc-runtime list
```

Or for even more details:

```bash
$ sudo cc-runtime list --cc-all
```
