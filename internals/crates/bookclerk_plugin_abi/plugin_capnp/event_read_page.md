# bookclerk_plugin_abi::plugin_capnp::event_read_page

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

#### Reader::get_events

```rust
pub fn get_events(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_events

```rust
pub fn has_events(&self) -> bool
```

#### Reader::get_cursor

```rust
pub fn get_cursor(self) -> Result<Reader<'a>>
```

#### Reader::has_cursor

```rust
pub fn has_cursor(&self) -> bool
```

#### Reader::get_more

```rust
pub fn get_more(self) -> bool
```

#### Reader::get_expired_event_ids

```rust
pub fn get_expired_event_ids(self) -> Result<Reader<'a>>
```

#### Reader::has_expired_event_ids

```rust
pub fn has_expired_event_ids(&self) -> bool
```

#### Reader::get_truncated

```rust
pub fn get_truncated(self) -> bool
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

#### Builder::get_events

```rust
pub fn get_events(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_events

```rust
pub fn set_events(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_events

```rust
pub fn init_events(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_events

```rust
pub fn has_events(&self) -> bool
```

#### Builder::get_cursor

```rust
pub fn get_cursor(self) -> Result<Builder<'a>>
```

#### Builder::set_cursor

```rust
pub fn set_cursor(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_cursor

```rust
pub fn init_cursor(self, size: u32) -> Builder<'a>
```

#### Builder::has_cursor

```rust
pub fn has_cursor(&self) -> bool
```

#### Builder::get_more

```rust
pub fn get_more(self) -> bool
```

#### Builder::set_more

```rust
pub fn set_more(&mut self, value: bool)
```

#### Builder::get_expired_event_ids

```rust
pub fn get_expired_event_ids(self) -> Result<Builder<'a>>
```

#### Builder::set_expired_event_ids

```rust
pub fn set_expired_event_ids(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_expired_event_ids

```rust
pub fn init_expired_event_ids(self, size: u32) -> Builder<'a>
```

#### Builder::has_expired_event_ids

```rust
pub fn has_expired_event_ids(&self) -> bool
```

#### Builder::get_truncated

```rust
pub fn get_truncated(self) -> bool
```

#### Builder::set_truncated

```rust
pub fn set_truncated(&mut self, value: bool)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
