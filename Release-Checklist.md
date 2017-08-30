Change release process, instead create a release candidate we can test the HEAD what will be the last commit using new automated obs package generation we can generate and test commits based on HEAD.

- [ ] Test the HEAD commit of the master branch using the tests included in the [tests repository][tests].
- [ ] Update ```VERSION``` in the runtime repository.
- [ ] Update the versions of `cc-proxy` and `cc-shim`.
- [ ] Generate OBS packages based on HEAD.
- [ ] Test OBS packages
	- [ ] Manual tests
		- [ ] Installation test.
		- [ ] Package signature test.
	- [ ] Automated tests
		- [ ] Integration tests included in the [tests repository][tests] under the integration directory.
- [ ] Tag release using new version.
- [ ] Write release notes:
  - [ ] Brief summary of known issues, pointing to the appropriate Issues/PRs.
  - [ ] Version of Docker (ideally range of versions, or "up to version X") supported by the release.
  - [ ] Version of the OCI spec (ideally range of versions, or "up to version X") supported by the release.
  - [ ] Version of Clear Container image used by the release.
  - [ ] Add links to Installation instructions.
  - [ ] Document any common vulnerabilities and exposures (CVEs) fixed with links to the CVE database.
- [ ] Check if the [limitations doc](https://github.com/clearcontainers/runtime/blob/master/docs/limitations.md) needs updating.
- [ ] Post release details on the public mailing list.
- [ ] Update public IRC channel with a link to the latest release.


[tests]: https://github.com/clearcontainers/tests