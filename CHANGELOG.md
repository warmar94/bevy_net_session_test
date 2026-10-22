# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project
adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html) (before 1.0, a breaking
change or a Bevy / bevy_net_session / bevy_headless_test bump raises the minor version).

## [0.1.0] - Unreleased

First release, for Bevy 0.20.0, bevy_net_session 0.2 and bevy_headless_test 0.2. The helpers
moved here from bevy_headless_test 0.1's `loopback` module (feature `net_session`), unchanged
except for their path.

### Added

- `net_builder`, `host`, `join`, `connect` (a host and a client in one process over real UDP
  loopback), `NET_THIRD_PARTY`, `LOOPBACK_PAUSE`, `DEFAULT_JOIN_FRAMES`; re-exports of
  `bevy_net_session` and `bevy_headless_test`.
