# bookclerk_node_proto

Wire protocol between a Bookclerk host node and its owner cluster.

A host connects to `GET /api/node/v1/connect` with the WebSocket
subprotocol [`SUBPROTOCOL`](bookclerk_node_proto.md#bookclerk_node_protosubprotocol) and proves itself with the Ed25519 key it
enrolled (H-4 design, section 3):

1. host → owner: `Body::Hello` `{node_id, incarnation,
   software_version, resume}`
2. owner → host: `Body::Challenge` `{nonce, cluster_id}`
3. host → owner: `Body::Auth` `{sig}`, an Ed25519 signature over
   [`auth_transcript`](signature.md#bookclerk_node_protosignatureauth_transcript)
4. owner → host: `Body::Welcome`, or a close with
   [`CLOSE_AUTH_FAILED`](bookclerk_node_proto.md#bookclerk_node_protoclose_auth_failed) / [`CLOSE_REVOKED`](bookclerk_node_proto.md#bookclerk_node_protoclose_revoked).

Frames are JSON text of at most [`MAX_FRAME_BYTES`](bookclerk_node_proto.md#bookclerk_node_protomax_frame_bytes); binary frames are a
protocol error because bulk data travels over HTTP.

After the handshake, a hello that listed held fences is answered with
`Body::Resumed`. Work is pulled: the host sends `Body::Claim` and
gets `Body::Work` (the job, its fence, and its secrets sealed to the
node's box key with [`seal_secrets`](sealed.md#bookclerk_node_protosealedseal_secrets)) or `Body::NoWork`; it then
sends `Body::Renew`, `Body::Complete` or `Body::Fail` under that
fence, and may write a refreshed store token back with
`Body::SecretUpdate`. The owner forwards the host's `node.<id>` queue
as `Body::Wake` frames, each answered with `Body::Ack`.

A claim may carry a [`ClaimFilter`](frames.md#bookclerk_node_protoframesclaimfilter) (the kinds the host runs and the
plugins its operator consented to); a job handed back unrun goes with
`Body::Release`. To act on wakes the host reads
`Body::DeploymentsList` and `Body::InstanceConfigGet` (`SECRETS`
sealed with [`seal_instance_secrets`](sealed.md#bookclerk_node_protosealedseal_instance_secrets)), reports
`Body::Observation`s, and fetches stored archives over HTTP at
[`ARTIFACTS_PATH`](bookclerk_node_proto.md#bookclerk_node_protoartifacts_path) with a request signed by [`sign_artifact_request`](signature.md#bookclerk_node_protosignaturesign_artifact_request).

A scan run on a host returns its rows in fenced `Body::ScanResults`
batches; the owner validates and writes them. An acquire asks
`Body::AcquirePlan` for its titles, then for each title asks
`Body::UploadGrant` and streams the files over HTTP at
[`UPLOADS_PATH`](bookclerk_node_proto.md#bookclerk_node_protouploads_path) with the token sealed in `Body::UploadGranted`; the
owner streams them on to its output destinations without staging them,
and the job completes when the last title's upload commits.

A branch shares the titles of its own library with the library it
joined (`Body::CatalogBegin`, `Body::CatalogSync`,
`Body::CatalogUpload`; see [`CatalogTitle`](catalog.md#bookclerk_node_protocatalogcatalogtitle)): titles carry no account
or user field, and files reuse the upload endpoint.

A host that leaves sends `Body::NodeLeaving`; the owner revokes the
node at once. Joining with a sign-in instead of a link is the OAuth
code + PKCE flow of the built-in client [`BRANCH_CLIENT_ID`](join.md#bookclerk_node_protojoinbranch_client_id) ([`OwnerInfo`](join.md#bookclerk_node_protojoinownerinfo),
[`OAuthEnrollRequest`](join.md#bookclerk_node_protojoinoauthenrollrequest)). An Owner of the library may instead let the
install join as a clerk (an owner node, `ConnectionType::Clerk`): its
redeem answer carries a [`ClerkBootstrap`](clerk.md#bookclerk_node_protoclerkclerkbootstrap) sealed to the node's box key.

This crate is pure: no I/O, no clock, no randomness. The daemon (owner)
and the host client share it so both sides sign and check the same
bytes.

## Modules

### [`bookclerk_node_proto`](bookclerk_node_proto.md)

*14 constants*

### [`catalog`](catalog.md)

*4 functions, 4 structs, 6 constants*

### [`clerk`](clerk.md)

*1 struct, 2 enums, 3 functions, 5 constants*

### [`frames`](frames.md)

*15 structs, 2 enums, 2 functions, 9 constants*

### [`join`](join.md)

*3 structs, 4 functions, 9 constants*

### [`keys`](keys.md)

*1 constant, 1 enum, 4 functions*

### [`sealed`](sealed.md)

*1 enum, 14 functions, 3 structs, 6 constants*

### [`signature`](signature.md)

*1 enum, 10 functions, 2 structs, 5 constants*

