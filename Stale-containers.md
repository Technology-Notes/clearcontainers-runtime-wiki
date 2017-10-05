A Clear Container is created by using a number of assets including a guest kernel and a root filesystem image [*].

On a normally [installed system](Installation), these assets are automatically [updated](/clearcontainers/runtime/blob/master/docs/upgrading.md) by the systems package manager. However, if any containers were running *at the time the package manager was updating the assets*, those containers will not be using the latest versions of the assets. These containers are thus referred to as "stale".

To ensure such stale containers benefit from the latest assets versions (features, security fixes and performance improvements), they should be restarted.

To list "stale" containers that should be restarted:

```
$ sudo cc-runtime list --cc-all|awk '$12 != "" { printf("%s;%s;%s;%s;%s;%s\n", $1, $8, $10, $9, $11, $12)}'
```


---

[*] - See [osbuilder](https://github.com/clearcontainers/osbuilder) for further details and an explanation of how to create custom assets.