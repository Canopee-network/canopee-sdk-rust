# Canopee SDK — Rust

Client SDK for a [Canopee](https://github.com/Canopee-network/canopee) node's
Unix socket: identity, storage, user records, devices, pairing, DHT, pub/sub
and capabilities.

This is the **canonical SDK**. The `canopee-node`, `canopee-protocol`,
`canopee-identity`, `canopee-storage` and `canopee-config` crates on which it
depends are vendored under `vendor/` (same sources as the main repository) so
the crate builds standalone. Other SDKs — TypeScript, Go, Python — are tested
against the golden fixture produced from this crate.

## Quick start

```rust
use canopee_sdk::{CanopeeClient, ObjectType};

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    let client = CanopeeClient::connect().await?;
    let id = client.put(b"hi".to_vec(), None).await?;   // ObjectId
    let obj = client.get(&id).await?;                   // Object
    Ok(())
}
```

`CanopeeClient::connect()` picks the socket from `$CANOPEE_APP_ROOT/.canopee/node.sock`
(falling back to `~/.canopee/node.sock`). Every `NodeCommand`/`NodeResponse`
pair of the node has a typed method on the client — `put`, `put_object`,
`concat`, `get`, `list`, `delete_object`, `set_name`, `export_object`,
`import_object`, `status`, `identity`, `shutdown`, `dh_public_key`,
`save_profile`, `load_profile`, `save_contact_list`, `load_contact_list`,
`save_home_index`, `load_home_index`, `set_home_entry_shared`,
`export_identity`, `import_identity`, `publish_pointer`, `publish_app_pointer`,
`resolve_app_pointer`, `resolve_pointer`, `claim_username`, `resolve_username`,
`show_username`, `resolve_profile`, `device`, `device_list`,
`resolve_owner_device`, `add_device`, `remove_device`, `initiate_pairing`,
`complete_pairing`, `sync_from_peer`, `sync_device_list`, `dial`,
`listen_via_relay`, `publish`, `subscribe`, `peers`, `relay_reservations`,
`find_providers`, `fetch_object`, `announce`, `put_object_public`,
`share_object`, `grant_capability`, `list_capabilities`, `revoke_capability`,
`check_capability`, `check_access`, `start_serve_session`,
`stop_serve_session`.

A node error surfaces as `anyhow` error containing the node's message.

## Testing the wire format

The socket speaks **bincode 1.x legacy**. To regenerate the golden fixture
(the `fixtures/golden.json` shared by every SDK's offline tests):

```sh
CANOPEE_GOLDEN_DIR=$PWD/fixtures cargo +1.95.0 test --test golden_vectors
```

Running plain `cargo test` runs the same generator in no-op mode plus the
crate's unit tests. Regenerating is deterministic except for vectors that embed
a freshly-minted signing identity (AppPointerRecord / Capability / PairingData),
which rotate keys on every run; compare those by shape, not bytes.

## Layout

- `src/` — the SDK (socket framing, typed client, subscription stream).
- `vendor/` — `canopee-protocol` (wire types), `canopee-storage`,
  `canopee-identity`, `canopee-config`, vendored from the main Canopee repo.
- `tests/golden_vectors.rs` — golden fixture generator (env-gated).
- `fixtures/golden.json` — canonical 139-vector wire fixture.

## License

MIT