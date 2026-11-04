# bookclerk_plugin_abi::plugin_capnp::scheduled_controller

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

#### Reader::get_cron

```rust
pub fn get_cron(self) -> Result<Reader<'a>>
```

#### Reader::has_cron

```rust
pub fn has_cron(&self) -> bool
```

#### Reader::get_scheduled_time

```rust
pub fn get_scheduled_time(self) -> u64
```

#### Reader::get_fire_id

```rust
pub fn get_fire_id(self) -> Result<Reader<'a>>
```

#### Reader::has_fire_id

```rust
pub fn has_fire_id(&self) -> bool
```

#### Reader::get_attempt

```rust
pub fn get_attempt(self) -> u32
```

#### Reader::get_deadline_unix_ms

```rust
pub fn get_deadline_unix_ms(self) -> u64
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

#### Builder::get_cron

```rust
pub fn get_cron(self) -> Result<Builder<'a>>
```

#### Builder::set_cron

```rust
pub fn set_cron(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_cron

```rust
pub fn init_cron(self, size: u32) -> Builder<'a>
```

#### Builder::has_cron

```rust
pub fn has_cron(&self) -> bool
```

#### Builder::get_scheduled_time

```rust
pub fn get_scheduled_time(self) -> u64
```

#### Builder::set_scheduled_time

```rust
pub fn set_scheduled_time(&mut self, value: u64)
```

#### Builder::get_fire_id

```rust
pub fn get_fire_id(self) -> Result<Builder<'a>>
```

#### Builder::set_fire_id

```rust
pub fn set_fire_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_fire_id

```rust
pub fn init_fire_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_fire_id

```rust
pub fn has_fire_id(&self) -> bool
```

#### Builder::get_attempt

```rust
pub fn get_attempt(self) -> u32
```

#### Builder::set_attempt

```rust
pub fn set_attempt(&mut self, value: u32)
```

#### Builder::get_deadline_unix_ms

```rust
pub fn get_deadline_unix_ms(self) -> u64
```

#### Builder::set_deadline_unix_ms

```rust
pub fn set_deadline_unix_ms(&mut self, value: u64)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
