# bookclerk_plugin_abi::plugin_capnp::db_timing

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

#### Reader::get_attempt_elapsed_us

```rust
pub fn get_attempt_elapsed_us(self) -> u64
```

#### Reader::get_db_execution_us

```rust
pub fn get_db_execution_us(self) -> u64
```

#### Reader::get_db_timing_source

```rust
pub fn get_db_timing_source(self) -> Result<Reader<'a>>
```

#### Reader::has_db_timing_source

```rust
pub fn has_db_timing_source(&self) -> bool
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

#### Builder::get_attempt_elapsed_us

```rust
pub fn get_attempt_elapsed_us(self) -> u64
```

#### Builder::set_attempt_elapsed_us

```rust
pub fn set_attempt_elapsed_us(&mut self, value: u64)
```

#### Builder::get_db_execution_us

```rust
pub fn get_db_execution_us(self) -> u64
```

#### Builder::set_db_execution_us

```rust
pub fn set_db_execution_us(&mut self, value: u64)
```

#### Builder::get_db_timing_source

```rust
pub fn get_db_timing_source(self) -> Result<Builder<'a>>
```

#### Builder::set_db_timing_source

```rust
pub fn set_db_timing_source(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_db_timing_source

```rust
pub fn init_db_timing_source(self, size: u32) -> Builder<'a>
```

#### Builder::has_db_timing_source

```rust
pub fn has_db_timing_source(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
