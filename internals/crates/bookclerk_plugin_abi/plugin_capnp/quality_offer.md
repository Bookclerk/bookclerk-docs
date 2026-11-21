# bookclerk_plugin_abi::plugin_capnp::quality_offer

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

#### Reader::get_codec

```rust
pub fn get_codec(self) -> Result<OfferedCodec, NotInSchema>
```

#### Reader::get_aac_profile

```rust
pub fn get_aac_profile(self) -> Result<OfferedAacProfile, NotInSchema>
```

#### Reader::get_bitrate_kbps

```rust
pub fn get_bitrate_kbps(self) -> u32
```

#### Reader::get_sample_rate_hz

```rust
pub fn get_sample_rate_hz(self) -> u32
```

#### Reader::get_channels

```rust
pub fn get_channels(self) -> u32
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

#### Builder::get_codec

```rust
pub fn get_codec(self) -> Result<OfferedCodec, NotInSchema>
```

#### Builder::set_codec

```rust
pub fn set_codec(&mut self, value: OfferedCodec)
```

#### Builder::get_aac_profile

```rust
pub fn get_aac_profile(self) -> Result<OfferedAacProfile, NotInSchema>
```

#### Builder::set_aac_profile

```rust
pub fn set_aac_profile(&mut self, value: OfferedAacProfile)
```

#### Builder::get_bitrate_kbps

```rust
pub fn get_bitrate_kbps(self) -> u32
```

#### Builder::set_bitrate_kbps

```rust
pub fn set_bitrate_kbps(&mut self, value: u32)
```

#### Builder::get_sample_rate_hz

```rust
pub fn get_sample_rate_hz(self) -> u32
```

#### Builder::set_sample_rate_hz

```rust
pub fn set_sample_rate_hz(&mut self, value: u32)
```

#### Builder::get_channels

```rust
pub fn get_channels(self) -> u32
```

#### Builder::set_channels

```rust
pub fn set_channels(&mut self, value: u32)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
