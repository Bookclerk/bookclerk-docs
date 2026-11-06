# bookclerk_plugin_abi::plugin_capnp::named_kv

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

#### Reader::get_name

```rust
pub fn get_name(self) -> Result<Reader<'a>>
```

#### Reader::has_name

```rust
pub fn has_name(&self) -> bool
```

#### Reader::get_namespace

```rust
pub fn get_namespace(self) -> Result<Reader<'a>>
```

#### Reader::has_namespace

```rust
pub fn has_namespace(&self) -> bool
```

#### Reader::get_kv

```rust
pub fn get_kv(self) -> Result<Client>
```

#### Reader::has_kv

```rust
pub fn has_kv(&self) -> bool
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

#### Builder::get_name

```rust
pub fn get_name(self) -> Result<Builder<'a>>
```

#### Builder::set_name

```rust
pub fn set_name(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_name

```rust
pub fn init_name(self, size: u32) -> Builder<'a>
```

#### Builder::has_name

```rust
pub fn has_name(&self) -> bool
```

#### Builder::get_namespace

```rust
pub fn get_namespace(self) -> Result<Builder<'a>>
```

#### Builder::set_namespace

```rust
pub fn set_namespace(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_namespace

```rust
pub fn init_namespace(self, size: u32) -> Builder<'a>
```

#### Builder::has_namespace

```rust
pub fn has_namespace(&self) -> bool
```

#### Builder::get_kv

```rust
pub fn get_kv(self) -> Result<Client>
```

#### Builder::set_kv

```rust
pub fn set_kv(&mut self, value: Client)
```

#### Builder::has_kv

```rust
pub fn has_kv(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

#### Pipeline::get_kv

```rust
pub fn get_kv(&self) -> Client
```

Implements: `FromTypelessPipeline`
