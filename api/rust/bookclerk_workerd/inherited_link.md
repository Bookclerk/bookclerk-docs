**bookclerk_workerd > inherited_link**

# Module: inherited_link

## Contents

**Enums**

- [`InheritedDuplex`](#inheritedduplex) - Bidirectional end of a host-created RPC or proxy link.

---

## bookclerk_workerd::inherited_link::InheritedDuplex

*Enum*

Bidirectional end of a host-created RPC or proxy link.

**Variants:**
- `Unix(tokio::net::UnixStream)` - Unix `socketpair` end.

**Methods:**

- `fn open(spec: &str) -> Result<Self>` - Open `spec` (`fd:<n>` or `handle:<n>`) as a Tokio duplex.
- `fn into_split(self: Self) -> (Box<dyn AsyncRead>, Box<dyn AsyncWrite>)` - Split into read and write halves for Cap'n Proto or the mux server.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **AsyncWrite**
  - `fn poll_write(self: std::pin::Pin<& mut Self>, cx: & mut std::task::Context, buf: &[u8]) -> std::task::Poll<std::io::Result<usize>>`
  - `fn poll_flush(self: std::pin::Pin<& mut Self>, cx: & mut std::task::Context) -> std::task::Poll<std::io::Result<()>>`
  - `fn poll_shutdown(self: std::pin::Pin<& mut Self>, cx: & mut std::task::Context) -> std::task::Poll<std::io::Result<()>>`
- **AsyncRead**
  - `fn poll_read(self: std::pin::Pin<& mut Self>, cx: & mut std::task::Context, buf: & mut tokio::io::ReadBuf) -> std::task::Poll<std::io::Result<()>>`



