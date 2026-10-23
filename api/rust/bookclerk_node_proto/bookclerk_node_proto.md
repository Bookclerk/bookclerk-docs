**bookclerk_node_proto**

# Module: bookclerk_node_proto

## Contents

**Constants**

- [`ARTIFACTS_PATH`](#artifacts_path) - Path prefix of the owner's artifact endpoint: `GET
- [`CLOSE_AUTH_FAILED`](#close_auth_failed) - Close code: the handshake failed (unknown node, bad signature, or a
- [`CLOSE_PROTOCOL`](#close_protocol) - Close code: a frame broke the protocol after the handshake (binary,
- [`CLOSE_REVOKED`](#close_revoked) - Close code: the node's credential is revoked.
- [`CLOSE_SUPERSEDED`](#close_superseded) - Close code: a newer authenticated connection for the same node replaced
- [`FRAME_VERSION`](#frame_version) - Frame envelope version (`v`).
- [`HANDSHAKE_TIMEOUT_SECS`](#handshake_timeout_secs) - Seconds the owner waits for each handshake frame.
- [`IDLE_TIMEOUT_SECS`](#idle_timeout_secs) - Seconds without any inbound frame before a side closes the connection.
- [`MAX_FRAME_BYTES`](#max_frame_bytes) - Largest frame either side accepts (1 MiB).
- [`PING_INTERVAL_SECS`](#ping_interval_secs) - Seconds between pings from each side.
- [`SEND_QUEUE_FRAMES`](#send_queue_frames) - Frames a side queues for sending before it stops forwarding wakes and
- [`SUBPROTOCOL`](#subprotocol) - WebSocket subprotocol a host must offer.
- [`UPLOADS_PATH`](#uploads_path) - Path prefix of the owner's upload endpoint (H-4 design, section 6):
- [`UPLOAD_AUTH_SCHEME`](#upload_auth_scheme) - `Authorization` scheme of the upload endpoint.

---

## bookclerk_node_proto::ARTIFACTS_PATH

*Constant*: `&str`

Path prefix of the owner's artifact endpoint: `GET
/api/node/v1/artifacts/<archive_sha256>` with a signed `Authorization`
([`sign_artifact_request`](signature.md#bookclerk_node_protosignaturesign_artifact_request)) streams one stored archive to a host that
runs it.



## bookclerk_node_proto::CLOSE_AUTH_FAILED

*Constant*: `u16`

Close code: the handshake failed (unknown node, bad signature, or a
malformed or late handshake frame).



## bookclerk_node_proto::CLOSE_PROTOCOL

*Constant*: `u16`

Close code: a frame broke the protocol after the handshake (binary,
oversized, malformed, or the wrong version).



## bookclerk_node_proto::CLOSE_REVOKED

*Constant*: `u16`

Close code: the node's credential is revoked.



## bookclerk_node_proto::CLOSE_SUPERSEDED

*Constant*: `u16`

Close code: a newer authenticated connection for the same node replaced
this one.



## bookclerk_node_proto::FRAME_VERSION

*Constant*: `u32`

Frame envelope version (`v`).



## bookclerk_node_proto::HANDSHAKE_TIMEOUT_SECS

*Constant*: `u64`

Seconds the owner waits for each handshake frame.



## bookclerk_node_proto::IDLE_TIMEOUT_SECS

*Constant*: `u64`

Seconds without any inbound frame before a side closes the connection.



## bookclerk_node_proto::MAX_FRAME_BYTES

*Constant*: `usize`

Largest frame either side accepts (1 MiB).



## bookclerk_node_proto::PING_INTERVAL_SECS

*Constant*: `u64`

Seconds between pings from each side.



## bookclerk_node_proto::SEND_QUEUE_FRAMES

*Constant*: `usize`

Frames a side queues for sending before it stops forwarding wakes and
waits for the peer to read.



## bookclerk_node_proto::SUBPROTOCOL

*Constant*: `&str`

WebSocket subprotocol a host must offer.



## bookclerk_node_proto::UPLOADS_PATH

*Constant*: `&str`

Path prefix of the owner's upload endpoint (H-4 design, section 6):
`PUT|HEAD <prefix><upload_id>/files/<n>` streams one declared file and
`POST <prefix><upload_id>/commit` commits the title, each with
`Authorization: Bookclerk-Upload <token>` ([`UPLOAD_AUTH_SCHEME`](bookclerk_node_proto.md#bookclerk_node_protoupload_auth_scheme)).



## bookclerk_node_proto::UPLOAD_AUTH_SCHEME

*Constant*: `&str`

`Authorization` scheme of the upload endpoint.



