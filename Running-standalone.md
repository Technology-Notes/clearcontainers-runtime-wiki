It is possible to run `cc-runtime` without a container manager such as `docker` (+`containerd`) or `kubernetes`.

## Create an OCI bundle

```bash
$ bundle="/tmp/bundle"
$ rootfs="$bundle/rootfs"
$ mkdir -p "$rootfs" && (cd "$bundle" && sudo runc spec)
$ sudo docker export $(sudo docker create busybox) | tar -C "$rootfs" -xvf -
```

## Create a Clear Container

```bash
$ sudo cc-runtime run --bundle "$bundle"
```

You should now be running a busybox shell. In a different terminal, try running:

```bash
$ sudo cc-runtime list
```

Or for even more details:

```bash
$ sudo cc-runtime list --cc-all
```
