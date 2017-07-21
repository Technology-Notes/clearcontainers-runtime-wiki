## Versioning

The Intel® Clear Containers project follows the semantic [versioning](http://semver.org/) and will have a release each Friday.

Versions are in the form `MAJOR.MINOR.PATCH`, examples are: `3.0.0`, `3.0.0-rc.5`, or `99.123.77+foo.bar.baz.5`.
- When `PATCH` increases, the new release contains important security fixes and an upgrade is recommended.
- When `MINOR` increases, the new release added new features without changing the existing behavior.
- When `MAJOR` increases, the new release added new features, bug fixes, or both which change the behavior from the previous release.

`PATCH` can contain extra details after the number. Dashes denote pre-release versions. `3.0.0-rc.5` in the example denotes the fifth release candidate for release `3.0.0`. Plus signs denote other details. In our example, `+foo.bar.baz.5` is additional information regarding release `99.123.77`.

`MAJOR` releases will likely require a change of the container manager version used, for example Docker\*. Please refer to the release notes for further details on this regard.

## Release checklist
To always have a stable and fully functional version of Intel® Clear Containers working on all supported distributions, a new issue to handle tracking the new release will be filed. The issue description must contain the [Release checklist][checklist] and it must be fixed and closed before releasing a new version.

[checklist]: https://github.com/01org/cc-oci-runtime/wiki/Release-Checklist