**bookclerk_workerd**

# Module: bookclerk_workerd

## Contents

**Modules**

- [`bridge_http`](#bridge_http) - Minimal HTTP/1.1 client for the workerd bridge (JSON control plane, Cap'n
- [`bridge_stdio`](#bridge_stdio) - Bookclerk Cap'n Proto stdio adapter over the workerd HTTP bridge.
- [`config`](#config) - Generate a real workerd Cap'n Proto config for one plugin isolate.
- [`egress`](#egress) - Host egress policy for workerd guests (wraps shared [`EgressPolicy`](../bookclerk_plugin_manifest/egress.md#bookclerk_plugin_manifestegressegresspolicy)).
- [`ensure`](#ensure) - Download / refresh the pinned Cloudflare `workerd` binary.
- [`grant`](#grant) - Operator consent grant overrides delivered to `bookclerk-workerd` via env.
- [`granted`](#granted) - Host-granted HTTP reverse channel for workerd isolates: `JobRunner.job`
- [`granted_http`](#granted_http) - Sans-IO HTTP/1.1 parsing for the granted reverse channel.
- [`inherited_link`](#inherited_link) - Open a host-inherited sibling link (`fd:<n>` / `handle:<n>`) as a Tokio stream.
- [`invoke`](#invoke) - `POST /invoke`: the Cap'n Proto data plane between the launcher and the
- [`pin`](#pin) - Pinned Cloudflare `workerd` release metadata shared with `bookclerk-dev`.
- [`socket_proxy`](#socket_proxy) - Host-mediated TCP proxy for native-behind-workerd guests.
- [`unix_bind`](#unix_bind) - Unix-domain listeners that stay under `sockaddr_un` limits.

---

## Module: bridge_http

Minimal HTTP/1.1 client for the workerd bridge (JSON control plane, Cap'n
Proto `/invoke` bodies, and streamed object bodies).



## Module: bridge_stdio

Bookclerk Cap'n Proto stdio adapter over the workerd HTTP bridge.

Toward the host this process is a [`PluginWorker`](../bookclerk_plugin_abi/roles.md#bookclerk_plugin_abirolespluginworker). The control plane
(`describe` / `open` / `shutdown`) is a JSON policy handshake with the
adapter isolate; every ABI *method* call on an exported entrypoint is one
`POST /invoke` carrying Cap'n Proto bytes ([`crate::invoke`](invoke.md)), so the typed
clients from `bookclerk-plugin-abi` (`ContentSourceClient`,
`EventConsumerClient`, `JobRunnerClient`, `DatabaseClient`, …) are reused
unchanged over the [`InvokeClient`](invoke.md#bookclerk_workerdinvokeinvokeclient) hook. Object bodies stream over the
dedicated `/destination/get` / `/destination/put` routes. The exported
[`Entrypoints`](../bookclerk_plugin_abi/roles.md#bookclerk_plugin_abirolesentrypoints) follow the verified `plugin.toml` capabilities (the host
allowlist), never a guest-declared widening. See `docs/workerd-bridge.md`.

With a `Backend::Native` guest the isolate stays the control plane
(`describe` merge, `open` policy, `shutdown`) while every entrypoint
capability is the guest's own typed Cap'n Proto client, forwarded to the
host without entering JavaScript memory.



## Module: config

Generate a real workerd Cap'n Proto config for one plugin isolate.



## Module: egress

Host egress policy for workerd guests (wraps shared [`EgressPolicy`](../bookclerk_plugin_manifest/egress.md#bookclerk_plugin_manifestegressegresspolicy)).



## Module: ensure

Download / refresh the pinned Cloudflare `workerd` binary.



## Module: grant

Operator consent grant overrides delivered to `bookclerk-workerd` via env.

The plugin host injects these after resolving the effective grant at spawn.
Values always clamp to host [`WorkerdLimits`](../bookclerk_plugin_manifest/types.md#bookclerk_plugin_manifesttypesworkerdlimits) caps and never widen beyond
the manifest-derived egress policy / limits.



## Module: granted

Host-granted HTTP reverse channel for workerd isolates: `JobRunner.job`
streams (`Source` / `Destination` / `ProgressSink`), the job's
`Cancellation` long poll (`/cancel`), named database sessions
(`/db/execute`), and the `EVENTS` outbox publisher (`/events/publish`).

HTTP accept/read/write runs on the multi-thread runtime (`tokio::spawn`) so
it is not starved by the Cap'n Proto vat `LocalSet`. Stub calls stay on the
vat thread via a command channel.



## Module: granted_http

Sans-IO HTTP/1.1 parsing for the granted reverse channel.

Every byte on the granted channel comes from a plugin isolate, so the
request head, the chunked body framing and the query string are parsed
here over plain byte slices, with no socket in sight. [`crate::granted`](granted.md)
feeds these from its reader; unit tests and the `granted_http` fuzz
target feed them directly, so all three drive the same code
(`docs/adr/untrusted-input-fuzzing.md`).

Limits enforced here are structural (head size, chunk-size digits,
trailer count). Body byte caps belong to the route and stay in
[`crate::granted`](granted.md).

Parsing is strict rather than lenient: the only client is the plugin
runtime's own `fetch`, so anything RFC 9112 does not require accepting
(bare LF, whitespace before a chunk size, a second transfer coding) is
rejected and answered with `400 Bad Request` instead of being guessed at.



## Module: inherited_link

Open a host-inherited sibling link (`fd:<n>` / `handle:<n>`) as a Tokio stream.



## Module: invoke

`POST /invoke`: the Cap'n Proto data plane between the launcher and the
adapter isolate.

[`InvokeClient`](invoke.md#bookclerk_workerdinvokeinvokeclient) is a Cap'n Proto `ClientHook`. Every method call made
through a typed client built from it (`ContentSourceClient`,
`EventConsumerClient`, `JobRunnerClient`, `DatabaseClient`, …) becomes one
HTTP request whose body is the `$Params` struct as an unpacked
single-segment message and whose `200` reply body is the `$Results` struct
— the same envelopes the generated `generated-wire.ts` codecs read and
write in the isolate. No JSON projection of any ABI struct exists on this
path; see `docs/workerd-bridge.md`.

Unpacked messages carry no capability table, so interface-typed fields are
written as capability indexes and the table travels beside the message as
the `X-Bookclerk-Caps` JSON header. Host-served capabilities are
[`GrantCap`](invoke.md#bookclerk_workerdinvokegrantcap)s — grant tokens the isolate presents on the granted channel —
and the only isolate-served kind is an `AdapterDatabaseSession`, which the
reply names by object id and later calls address with `X-Bookclerk-Target`.



## Module: pin

Pinned Cloudflare `workerd` release metadata shared with `bookclerk-dev`.

Source of truth: `../workerd-pin.json` (do not edit this file by hand).
Bumped by `scripts/bump-workerd-pin.py` (CI: `.github/workflows/workerd-pin-bump.yml`)
with a 7-day publish cooldown for supply-chain safety.
Regenerated by `scripts/sync-workerd-pin.py`.



## Module: socket_proxy

Host-mediated TCP proxy for native-behind-workerd guests.

Native plugins must not open ambient `AF_INET` sockets. They speak HTTP
CONNECT over an inherited multiplexed link ([`spawn_link`](socket_proxy.md#bookclerk_workerdsocket_proxyspawn_link)); the launcher
applies the same [`EgressPolicy`](../bookclerk_plugin_manifest/egress.md#bookclerk_plugin_manifestegressegresspolicy) as workerd `fetch()`/`connect()` and
splices bytes. [`spawn_unix`](socket_proxy.md#bookclerk_workerdsocket_proxyspawn_unix) remains for unit tests that bind a pathname
listener.



## Module: unix_bind

Unix-domain listeners that stay under `sockaddr_un` limits.

A host's temp or plugin-state directory can sit under a long prefix (a
checkout under `/home/runner/work/bookclerk/bookclerk/` on GitHub Actions is
already ~38 bytes). Plugin scratch plus a workerd leaf used to overflow
Linux's 108-byte `sun_path`.

Linux GRANTED (workerd ↔ launcher) stays `unix-abstract:` — workerd is a
child of this launcher, not a sibling guest. Other Unix: bind a pathname
under the host-chosen session dir and pass a **relative** `unix:granted.sock`
to workerd (cwd is that directory).

Native-behind-workerd no longer binds a named socket-proxy listener. The
host delivers an inherited duplex (`fd:` / `handle:`) and
[`crate::socket_proxy::spawn_link`](socket_proxy.md#bookclerk_workerdsocket_proxyspawn_link) multiplexes CONNECT streams over it.

Do not fall back to `std::env::temp_dir()`: inside `bookclerk-workerd` that
is the process scratch (`TMPDIR`).



