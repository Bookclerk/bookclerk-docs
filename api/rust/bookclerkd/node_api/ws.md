**bookclerkd > node_api > ws**

# Module: node_api::ws

## Contents

**Functions**

- [`close_node_sessions`](#close_node_sessions) - Asks every connection of `node_id` on this node, live or still in the
- [`get_connect`](#get_connect) - `GET /api/node/v1/connect`: upgrade to the host-node socket after the

---

## bookclerkd::node_api::ws::close_node_sessions

*Function*

Asks every connection of `node_id` on this node, live or still in the
handshake, to close with `code`. Called for `node.revoked` wakes.

```rust
fn close_node_sessions(node_id: &str, code: u16)
```



## bookclerkd::node_api::ws::get_connect

*Function*

`GET /api/node/v1/connect`: upgrade to the host-node socket after the
transport check. 403 `transport_insecure` for a refused connection,
400 `subprotocol_required` without [`SUBPROTOCOL`](../../bookclerk_node_proto/bookclerk_node_proto.md#bookclerk_node_protosubprotocol).

```rust
fn get_connect(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, request: axum::extract::Request) -> axum::response::Response
```



