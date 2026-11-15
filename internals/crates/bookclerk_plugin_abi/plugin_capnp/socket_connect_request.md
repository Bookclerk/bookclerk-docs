# bookclerk_plugin_abi::plugin_capnp::socket_connect_request

| Item | Kind | Summary |
| --- | --- | --- |
| [`Owned`](#owned) | struct |  |
| [`Reader`](#reader) | struct |  |
| [`Builder`](#builder) | struct |  |
| [`Pipeline`](#pipeline) | struct |  |

## Structs

### Owned

```rust
pub struct Owned(/* private */);
```

Implements: `Clone`, `Copy`, `Introspect`, `Owned`, `OwnedStruct`, `Pipelined`, `SetterInput<Owned>`

### Reader

```rust
pub struct Reader<'a> {
    /* private fields */
}
```

#### Reader::reborrow

```rust
pub fn reborrow(&self) -> Reader<'_>
```

#### Reader::total_size

```rust
pub fn total_size(&self) -> Result<MessageSize>
```

#### Reader::get_url

```rust
pub fn get_url(self) -> Result<Reader<'a>>
```

#### Reader::has_url

```rust
pub fn has_url(&self) -> bool
```

#### Reader::get_protocols

```rust
pub fn get_protocols(self) -> Result<Reader<'a>>
```

#### Reader::has_protocols

```rust
pub fn has_protocols(&self) -> bool
```

#### Reader::get_headers

```rust
pub fn get_headers(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_headers

```rust
pub fn has_headers(&self) -> bool
```

#### Reader::get_auto_responses

```rust
pub fn get_auto_responses(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_auto_responses

```rust
pub fn has_auto_responses(&self) -> bool
```

#### Reader::get_attachment

```rust
pub fn get_attachment(self) -> Result<Reader<'a>>
```

#### Reader::has_attachment

```rust
pub fn has_attachment(&self) -> bool
```

Implements: `Clone`, `Copy`, `Debug`, `From<Reader<'a>>`, `From<StructReader<'a>>`, `FromPointerReader<'a>`, `HasTypeId`, `Imbue<'a>`, `IntoInternalStructReader<'a>`, `SetterInput<Owned>`

### Builder

```rust
pub struct Builder<'a> {
    /* private fields */
}
```

#### Builder::into_reader

```rust
pub fn into_reader(self) -> Reader<'a>
```

#### Builder::reborrow

```rust
pub fn reborrow(&mut self) -> Builder<'_>
```

#### Builder::reborrow_as_reader

```rust
pub fn reborrow_as_reader(&self) -> Reader<'_>
```

#### Builder::total_size

```rust
pub fn total_size(&self) -> Result<MessageSize>
```

#### Builder::get_url

```rust
pub fn get_url(self) -> Result<Builder<'a>>
```

#### Builder::set_url

```rust
pub fn set_url(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_url

```rust
pub fn init_url(self, size: u32) -> Builder<'a>
```

#### Builder::has_url

```rust
pub fn has_url(&self) -> bool
```

#### Builder::get_protocols

```rust
pub fn get_protocols(self) -> Result<Builder<'a>>
```

#### Builder::set_protocols

```rust
pub fn set_protocols(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_protocols

```rust
pub fn init_protocols(self, size: u32) -> Builder<'a>
```

#### Builder::has_protocols

```rust
pub fn has_protocols(&self) -> bool
```

#### Builder::get_headers

```rust
pub fn get_headers(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_headers

```rust
pub fn set_headers(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_headers

```rust
pub fn init_headers(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_headers

```rust
pub fn has_headers(&self) -> bool
```

#### Builder::get_auto_responses

```rust
pub fn get_auto_responses(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_auto_responses

```rust
pub fn set_auto_responses(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_auto_responses

```rust
pub fn init_auto_responses(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_auto_responses

```rust
pub fn has_auto_responses(&self) -> bool
```

#### Builder::get_attachment

```rust
pub fn get_attachment(self) -> Result<Builder<'a>>
```

#### Builder::set_attachment

```rust
pub fn set_attachment(&mut self, value: Reader<'_>)
```

#### Builder::init_attachment

```rust
pub fn init_attachment(self, size: u32) -> Builder<'a>
```

#### Builder::has_attachment

```rust
pub fn has_attachment(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
