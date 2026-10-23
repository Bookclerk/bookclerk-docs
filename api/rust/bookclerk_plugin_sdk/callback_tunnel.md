**bookclerk_plugin_sdk > callback_tunnel**

# Module: callback_tunnel

## Contents

**Structs**

- [`TunnelGuest`](#tunnelguest) - Guest end of the callback tunnel: accepts logical connections from the host.
- [`TunnelHost`](#tunnelhost) - Host end of the callback tunnel: opens logical connections toward the guest.

**Type Aliases**

- [`TunnelStream`](#tunnelstream) - One multiplexed logical connection implementing `AsyncRead` + `AsyncWrite`.

---

## bookclerk_plugin_sdk::callback_tunnel::TunnelGuest

*Struct*

Guest end of the callback tunnel: accepts logical connections from the host.

**Methods:**

- `fn new<R, W>(reader: R, writer: W) -> Self` - Spawns reader/writer tasks on the guest half of a duplex IPC link.
- `fn accept(self: & mut Self) -> Result<TunnelStream>` - Waits for the next logical connection opened by `TunnelHost::open`.



## bookclerk_plugin_sdk::callback_tunnel::TunnelHost

*Struct*

Host end of the callback tunnel: opens logical connections toward the guest.

**Methods:**

- `fn new<R, W>(reader: R, writer: W) -> Self` - Spawns reader/writer tasks on the host half of a duplex IPC link.
- `fn open(self: &Self) -> Result<TunnelStream>` - Opens one logical connection toward the guest and returns its stream.



## bookclerk_plugin_sdk::callback_tunnel::TunnelStream

*Type Alias*: `crate::mux::MuxStream`

One multiplexed logical connection implementing `AsyncRead` + `AsyncWrite`.



