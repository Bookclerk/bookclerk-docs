# bookclerk_workerd::inherited_link

Open a host-inherited sibling link (`fd:<n>` / `handle:<n>`) as a Tokio stream.

## Enums

### InheritedDuplex

```rust
pub enum InheritedDuplex {
    Unix(UnixStream),
}
```

Bidirectional end of a host-created RPC or proxy link.

**Variants**

- `Unix`: Unix `socketpair` end.

#### InheritedDuplex::open

```rust
pub fn open(spec: &str) -> Result<Self>
```

Open `spec` (`fd:<n>` or `handle:<n>`) as a Tokio duplex.

##### Errors

Returns an error when the spec is the wrong OS form or the descriptor
cannot be wrapped.

#### InheritedDuplex::into_split

```rust
pub fn into_split(
    self,
) -> (Box<dyn AsyncRead + Unpin + Send>, Box<dyn AsyncWrite + Unpin + Send>)
```

Split into read and write halves for Cap'n Proto or the mux server.

Implements: `AsyncRead`, `AsyncWrite`, `Debug`
