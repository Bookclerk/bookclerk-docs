**bookclerk_plugin_abi > host_rpc**

# Module: host_rpc

## Contents

**Structs**

- [`HostAdapterDatabaseSessionClient`](#hostadapterdatabasesessionclient) - Host-side client for `begin` on an open adapter session capability.

---

## bookclerk_plugin_abi::host_rpc::HostAdapterDatabaseSessionClient

*Struct*

Host-side client for `begin` on an open adapter session capability.

**Methods:**

- `fn from_session_client(client: adapter_database_session_capnp::Client) -> Self` - Casts a public adapter session capability to the host-private interface.

**Trait Implementations:**

- **HostAdapterDatabaseSession**
  - `fn begin(self: &'life0 Self, isolation: crate::IsolationReq) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn execute(self: &'life0 Self, request: crate::host_envelope::AdapterExecuteRequest) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



