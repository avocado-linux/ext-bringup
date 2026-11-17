# Changelog

All notable changes to avocado-ext-bringup are documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0]

### Added
- Initial release: hardware bringup tools (network, bus probing, storage, device tree,
  tracing) and a serial console that logs in as root without a password. Not for
  production.
- CI via the shared `avocado-linux/actions` reusable workflows: PR build check
  (`test.yml`) and tag-driven package + publish to the Avocado feed (`release.yml`).
