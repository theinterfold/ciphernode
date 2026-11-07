# Interfold ciphernode (v2)

The v2 ciphernode of [The Interfold](https://github.com/theinterfold/interfold), kept apart from the
main repository so that its design can be read and changed on its own. Once the design is agreed,
the crates move back into `theinterfold/interfold` at the same paths.

Start with the design decisions:
[docs/architecture/node-v2-decisions.md](docs/architecture/node-v2-decisions.md).

## Crates

| Crate         | Path                       | Role                                                                                                 |
| ------------- | -------------------------- | ---------------------------------------------------------------------------------------------------- |
| `e3-protocol` | `crates/protocol`          | The node as one pure state machine: selection, failover, DKG, decryption, accusations.               |
| `e3-engine`   | `crates/ciphernode/engine` | Applies each input in one journal transaction with the effects it causes, and replays on restart.    |
| `e3-journal`  | `crates/state/journal`     | The journal, snapshots and outbox, in one redb file.                                                 |
| `e3-blobs`    | `crates/state/blobs`       | Large payloads, stored once per E3 by hash.                                                          |
| `e3-chain`    | `crates/ciphernode/chain`  | Reads the contracts' logs as chain facts and sends the operator's transactions.                      |
| `e3-p2p`      | `crates/ciphernode/p2p`    | Operator-signed messages and the blobs they name, over libp2p.                                       |
| `e3-jobs`     | `crates/ciphernode/jobs`   | DKG, decryption and proof jobs, run off the engine.                                                  |
| `e3-node`     | `crates/ciphernode/node`   | Wires the engine to the adapters. The `e3-node` binary runs one node from a JSON configuration file. |
| `e3-sim`      | `crates/testing/sim`       | Runs whole committees of nodes in one process.                                                       |

## Build and test

```sh
cargo test --workspace
```

The toolchain comes from `rust-toolchain.toml`. The first build fetches `theinterfold/interfold`
(about 1 GB) for the shared crates; Cargo keeps it for later builds.

## Shared crates

The crypto, ZK, contract-binding and configuration crates come from `theinterfold/interfold` at one
commit: the `rev` in `Cargo.toml`, the head of its `refactor/ciphernode-v2` branch. To move to
another commit, change `rev` on every line together. A branch or tag in `theinterfold/interfold`
must keep the pinned commit, or Cargo cannot fetch it.

`.cargo/config.toml` sets `E3_ZK_PROVER_SKIP_FIXTURE_BUILD=1`, so the prover's build script does not
compile its test fixtures with `nargo` and `bb`.

## What stays in theinterfold/interfold

The `interfold` CLI (`interfold start` runs this node), the contracts, the circuits and the
end-to-end tests (`tests/integration/v2.sh`) are on the `refactor/ciphernode-v2` branch of
`theinterfold/interfold`.

## License

LGPL-3.0-only. See [LICENSE.md](LICENSE.md).
