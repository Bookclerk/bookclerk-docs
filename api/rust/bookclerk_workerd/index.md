# bookclerk_workerd

Library surface for the pinned Cloudflare `workerd` helper and egress policy.

## Audience

The `bookclerk-workerd` launcher binary and host packaging (`cargo ensure-workerd`).
Guest authors do not link this crate; they run inside the isolate it configures.

See `docs/plugins.md` for jail / workerd isolation requirements.

## Re-exports

- [`bridge_http::generate_bridge_token`](bridge_http.md#generate_bridge_token)
- [`config::adapter_binding_plan`](config.md#adapter_binding_plan)
- [`config::generated_backend_proxy_plan`](config.md#generated_backend_proxy_plan)
- [`config::materialize_native_backend`](config.md#materialize_native_backend)
- [`config::BindingSpec`](config.md#bindingspec)
- [`config::BindingTarget`](config.md#bindingtarget)
- [`config::EntrypointSource`](config.md#entrypointsource)
- [`config::GeneratedConfig`](config.md#generatedconfig)
- [`config::ListenSpec`](config.md#listenspec)
- [`config::WORKERD_SERVE_EXPERIMENTAL`](config.md#workerd_serve_experimental)
- [`egress::EgressProxy`](egress.md#egressproxy)
- [`ensure::ensure_workerd`](ensure.md#ensure_workerd)
- [`ensure::workerd_bin_path`](ensure.md#workerd_bin_path)
- [`grant::OperatorGrantEnv`](grant.md#operatorgrantenv)
- [`pin::binary_name`](pin.md#binary_name)
- [`pin::BUNDLED_WORKERD_COMPAT_DATE`](pin.md#bundled_workerd_compat_date)
- [`pin::WORKERD_RELEASE_TAG`](pin.md#workerd_release_tag)
- [`pin::WORKERD_VERSION_STAMP`](pin.md#workerd_version_stamp)

## Modules

- [`bridge_http`](bridge_http.md): Minimal HTTP/1.1 client for the workerd bridge (JSON control plane, Cap'n Proto `/invoke` bodies, and streamed object bodies).
- [`bridge_stdio`](bridge_stdio.md): Bookclerk Cap'n Proto stdio adapter over the workerd HTTP bridge.
- [`config`](config.md): Generate a real workerd Cap'n Proto config for one plugin isolate.
- [`egress`](egress.md): Host egress policy for workerd guests (wraps shared `EgressPolicy`).
- [`ensure`](ensure.md): Download / refresh the pinned Cloudflare `workerd` binary.
- [`grant`](grant.md): Operator consent grant overrides delivered to `bookclerk-workerd` via env.
- [`granted`](granted.md): Host-granted HTTP reverse channel for workerd isolates: `JobRunner.job` streams (`Source` / `Destination` / `ProgressSink`), the job's `Cancellation` long poll (`/cancel`), named database sessions (`/db/execute`), and the `EVENTS` outbox publisher (`/events/publish`).
- [`granted_http`](granted_http.md): Sans-IO HTTP/1.1 parsing for the granted reverse channel.
- [`inherited_link`](inherited_link.md): Open a host-inherited sibling link (`fd:<n>` / `handle:<n>`) as a Tokio stream.
- [`invoke`](invoke.md): `POST /invoke`: the Cap'n Proto data plane between the launcher and the adapter isolate.
- [`pin`](pin.md): Pinned Cloudflare `workerd` release metadata shared with `bookclerk-dev`.
- [`socket_proxy`](socket_proxy.md): Host-mediated TCP proxy for native-behind-workerd guests.
- [`unix_bind`](unix_bind.md): Unix-domain listeners that stay under `sockaddr_un` limits.
