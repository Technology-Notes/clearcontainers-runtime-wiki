# Terms

- EOL - end-of-life.
- OBS - SuSE's [Open Build Service](https://build.opensuse.org/).
- `$version` - refers to the particular distro version the section applies to. `${version}+1` refers to the next version and `${version}-1` refers to the previous version.

# Overview 

This page captures the required process which should be triggered when either a new distro version is released, or an existing version is retired (EOL).

# Detecting version changes

Currently manual.

NOTE: We need an automated notification for new versions and EOL versions (ideally *before* the event!)

# Handling version changes

For EOL versions, the tasks below will need to:

- Remove the EOL versions.
- Replace them with a "reasonable" newer version (denoted as `${version}+1`) where "reasonable" will probably be one of:
  - the latest version.
  - the latest LTS version.
  - the latest version known to support Clear Containers.

Note: the task below should be performed **in the order listed**.

- [ ] Raise a [tracking issue](https://github.com/clearcontainers/runtime/issues/new) and paste in this checklist section. As activities are completed, check the boxes so progress is clear.
- [ ] Raise an issue to update the [install guide](https://github.com/clearcontainers/runtime/wiki/Installation) for the distribution.
- [ ] Raise a [Jenkins issue](https://github.com/clearcontainers/jenkins/issues/new) to add a new CI Jenkins node *for each repository*. For EOL, also request the EOL version nodes be retired.
- [ ] Re-test [osbuilder](https://github.com/clearcontainers/osbuilder):
  - [ ] On a host system running the new version.
  - [ ] Specifying the new version in the `Dockerfile` (then raise a PR to update that file).
- [ ] Raise a [packaging issue](https://github.com/clearcontainers/packaging/issues/new) for new OBS packages to be created.
- [ ] Raise a [test issue](https://github.com/clearcontainers/tests/issues/new) to test distro upgrading:
  - For EOL versions, test upgrading from `${version}` to `${version}+1`.
  - For new vesrions, test upgrading from `${version}-1` to `${version}`.
  - In both cases, ensure it is still possible to create a Clear Container and that tests pass.
- [ ] Review the following documents to see if any changes need to be made:
  - [ ] [Upgrade guide](https://github.com/clearcontainers/runtime/blob/master/docs/upgrading.md).
  - [ ] [Architecture Guide](https://github.com/clearcontainers/runtime/blob/master/docs/architecture/architecture.md).
  - [ ] [Developer guide](https://github.com/clearcontainers/runtime/blob/master/docs/developers-clear-containers-install.md).
  - [ ] [Limitations](https://github.com/clearcontainers/runtime/blob/master/docs/limitations.md).
- [ ] Announce availability of new versions on the public mailing list, irc and slack.
