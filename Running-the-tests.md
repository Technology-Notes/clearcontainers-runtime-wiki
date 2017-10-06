The runtime comes with a comprehensive set of unit-tests (`*_test.go`). However, there are many other classes of tests which are stored in a separate [tests repository](https://github.com/clearcontainers/tests).

It is possible to run *all* the tests very easily, however **please** see the warning below **BEFORE** you run the commands below:

```
$ go get -d github.com/clearcontainers/runtime
$ .ci/setup.sh && .ci/run.sh; .ci/teardown.sh
```
The commands above will perform all setup (including cloning the `tests` repository), run all classes of test and display any problems at the end.

:warning: **WARNING:** :warning:

- It is not advisable to run the above on a system you care about since the tests are aggressive and run as the `root` user. If you still wish to run the entire suite of tests, the best advice is to run them inside a virtual machine, or on a non-essential system.