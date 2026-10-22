# bevy_net_session_test

[![License: MIT OR Apache-2.0](https://img.shields.io/badge/license-MIT%20OR%20Apache--2.0-blue.svg)](#license)
[![CI](https://github.com/warmar94/bevy_net_session_test/actions/workflows/ci.yml/badge.svg)](https://github.com/warmar94/bevy_net_session_test/actions/workflows/ci.yml)
[![Bevy 0.20.0](https://img.shields.io/badge/Bevy-0.20.0-informational)](https://bevyengine.org)

**Multiplayer tests without a second machine**: host, join and play a real
[`bevy_net_session`](https://crates.io/crates/bevy_net_session) session over UDP loopback, inside
one `cargo test`. Built on [`bevy_headless_test`](https://crates.io/crates/bevy_headless_test).

It is a **dev-dependency**: nothing of it ships in your game.

```rust,no_run
use bevy::prelude::*;
use bevy_net_session_test::bevy_headless_test::prelude::*;
use bevy_net_session_test::bevy_headless_test::replicon::bevy_replicon::prelude::*;
use bevy_net_session_test::bevy_net_session::NetSessionPlugin;
use bevy_net_session_test::{connect, net_builder};
use serde::{Deserialize, Serialize};

#[derive(Event, Serialize, Deserialize, Clone)]
struct Ping;

// One peer: the strict headless app + replicon + the session plugin + your shared protocol.
let peer = || {
    net_builder(NetSessionPlugin { protocol_version: 1, ..Default::default() })
        .setup(|app| {
            app.add_client_event::<Ping>(Channel::Ordered);
        })
        .build()
};
let (mut host, mut client) = (peer(), peer());
connect(&mut host, &mut client); // hosts on a free port, joins it, steps both until joined

client.world_mut().commands().client_trigger(Ping);
TestApp::step_together(&mut [&mut host, &mut client]);
```

## What it does

- `net_builder(NetSessionPlugin { .. })`: a `TestAppBuilder` for one peer: bevy_headless_test's
  strict app + replicon + the session plugin + a 1 ms real pause per frame. The networking
  crates' own system pairs are allowed (`NET_THIRD_PARTY`); your systems and the session layer's
  stay checked.
- `host(&mut host, max_clients) -> SocketAddr`: host on a free UDP port on 127.0.0.1.
- `join(&mut host, &mut client, addr, payload, max_frames) -> u32`: join and step both apps in
  lock-step until the client is joined and the host has its peer. A refused join fails the test
  with the `JoinFailed` reason (`VersionMismatch`, `Rejected`, ...).
- `connect(&mut host, &mut client) -> SocketAddr`: both, with an empty payload.
- `LOOPBACK_PAUSE`, `DEFAULT_JOIN_FRAMES`, and re-exports of `bevy_net_session` and
  `bevy_headless_test`.

It uses real sockets on 127.0.0.1 only, no outside network; a join takes a handful of frames.
The helpers panic with a message on failure: that is how a test fails.

## Quick start

```toml
[dev-dependencies]
bevy_net_session_test = "0.1"
```

Your game already depends on `bevy_net_session`; Cargo unifies the two, so the test peers run
exactly the session layer your game ships.

## Compatibility

| bevy_net_session_test | Bevy | bevy_net_session | bevy_headless_test | Rust |
|---|---|---|---|---|
| 0.1 | 0.20.0 | 0.2 | 0.2 | 1.97.1+ |

Before 0.1, these helpers were the `loopback` module of bevy_headless_test 0.1 (feature
`net_session`). Moving: `bevy_headless_test::loopback::X` is now `bevy_net_session_test::X`.

## License

Licensed under either of

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE) or <https://www.apache.org/licenses/LICENSE-2.0>)
- MIT license ([LICENSE-MIT](LICENSE-MIT) or <https://opensource.org/licenses/MIT>)

at your option.

## Contributing

Unless you explicitly state otherwise, any contribution intentionally submitted for inclusion in
the work by you, as defined in the Apache-2.0 license, shall be dual licensed as above, without
any additional terms or conditions.
