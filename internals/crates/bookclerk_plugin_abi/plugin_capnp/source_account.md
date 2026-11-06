# bookclerk_plugin_abi::plugin_capnp::source_account

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

#### Reader::get_account_id

```rust
pub fn get_account_id(self) -> Result<Reader<'a>>
```

#### Reader::has_account_id

```rust
pub fn has_account_id(&self) -> bool
```

#### Reader::get_source

```rust
pub fn get_source(self) -> Result<Reader<'a>>
```

#### Reader::has_source

```rust
pub fn has_source(&self) -> bool
```

#### Reader::get_marketplace

```rust
pub fn get_marketplace(self) -> Result<Reader<'a>>
```

#### Reader::has_marketplace

```rust
pub fn has_marketplace(&self) -> bool
```

#### Reader::get_label

```rust
pub fn get_label(self) -> Result<Reader<'a>>
```

#### Reader::has_label

```rust
pub fn has_label(&self) -> bool
```

#### Reader::get_scan_enabled

```rust
pub fn get_scan_enabled(self) -> bool
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

#### Builder::get_account_id

```rust
pub fn get_account_id(self) -> Result<Builder<'a>>
```

#### Builder::set_account_id

```rust
pub fn set_account_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_account_id

```rust
pub fn init_account_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_account_id

```rust
pub fn has_account_id(&self) -> bool
```

#### Builder::get_source

```rust
pub fn get_source(self) -> Result<Builder<'a>>
```

#### Builder::set_source

```rust
pub fn set_source(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_source

```rust
pub fn init_source(self, size: u32) -> Builder<'a>
```

#### Builder::has_source

```rust
pub fn has_source(&self) -> bool
```

#### Builder::get_marketplace

```rust
pub fn get_marketplace(self) -> Result<Builder<'a>>
```

#### Builder::set_marketplace

```rust
pub fn set_marketplace(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_marketplace

```rust
pub fn init_marketplace(self, size: u32) -> Builder<'a>
```

#### Builder::has_marketplace

```rust
pub fn has_marketplace(&self) -> bool
```

#### Builder::get_label

```rust
pub fn get_label(self) -> Result<Builder<'a>>
```

#### Builder::set_label

```rust
pub fn set_label(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_label

```rust
pub fn init_label(self, size: u32) -> Builder<'a>
```

#### Builder::has_label

```rust
pub fn has_label(&self) -> bool
```

#### Builder::get_scan_enabled

```rust
pub fn get_scan_enabled(self) -> bool
```

#### Builder::set_scan_enabled

```rust
pub fn set_scan_enabled(&mut self, value: bool)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
