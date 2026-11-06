# bookclerk_plugin_abi::plugin_capnp::expand_candidates_params

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

#### Reader::get_source

```rust
pub fn get_source(self) -> Result<Reader<'a>>
```

#### Reader::has_source

```rust
pub fn has_source(&self) -> bool
```

#### Reader::get_product_id

```rust
pub fn get_product_id(self) -> Result<Reader<'a>>
```

#### Reader::has_product_id

```rust
pub fn has_product_id(&self) -> bool
```

#### Reader::get_title

```rust
pub fn get_title(self) -> Result<Reader<'a>>
```

#### Reader::has_title

```rust
pub fn has_title(&self) -> bool
```

#### Reader::get_authors

```rust
pub fn get_authors(self) -> Result<Reader<'a>>
```

#### Reader::has_authors

```rust
pub fn has_authors(&self) -> bool
```

#### Reader::get_narrators

```rust
pub fn get_narrators(self) -> Result<Reader<'a>>
```

#### Reader::has_narrators

```rust
pub fn has_narrators(&self) -> bool
```

#### Reader::get_series

```rust
pub fn get_series(self) -> Result<Reader<'a>>
```

#### Reader::has_series

```rust
pub fn has_series(&self) -> bool
```

#### Reader::get_series_asin

```rust
pub fn get_series_asin(self) -> Result<Reader<'a>>
```

#### Reader::has_series_asin

```rust
pub fn has_series_asin(&self) -> bool
```

#### Reader::get_asin

```rust
pub fn get_asin(self) -> Result<Reader<'a>>
```

#### Reader::has_asin

```rust
pub fn has_asin(&self) -> bool
```

#### Reader::get_isbn

```rust
pub fn get_isbn(self) -> Result<Reader<'a>>
```

#### Reader::has_isbn

```rust
pub fn has_isbn(&self) -> bool
```

#### Reader::get_region

```rust
pub fn get_region(self) -> Result<Reader<'a>>
```

#### Reader::has_region

```rust
pub fn has_region(&self) -> bool
```

#### Reader::get_limit

```rust
pub fn get_limit(self) -> u32
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

#### Builder::get_product_id

```rust
pub fn get_product_id(self) -> Result<Builder<'a>>
```

#### Builder::set_product_id

```rust
pub fn set_product_id(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_product_id

```rust
pub fn init_product_id(self, size: u32) -> Builder<'a>
```

#### Builder::has_product_id

```rust
pub fn has_product_id(&self) -> bool
```

#### Builder::get_title

```rust
pub fn get_title(self) -> Result<Builder<'a>>
```

#### Builder::set_title

```rust
pub fn set_title(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_title

```rust
pub fn init_title(self, size: u32) -> Builder<'a>
```

#### Builder::has_title

```rust
pub fn has_title(&self) -> bool
```

#### Builder::get_authors

```rust
pub fn get_authors(self) -> Result<Builder<'a>>
```

#### Builder::set_authors

```rust
pub fn set_authors(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_authors

```rust
pub fn init_authors(self, size: u32) -> Builder<'a>
```

#### Builder::has_authors

```rust
pub fn has_authors(&self) -> bool
```

#### Builder::get_narrators

```rust
pub fn get_narrators(self) -> Result<Builder<'a>>
```

#### Builder::set_narrators

```rust
pub fn set_narrators(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_narrators

```rust
pub fn init_narrators(self, size: u32) -> Builder<'a>
```

#### Builder::has_narrators

```rust
pub fn has_narrators(&self) -> bool
```

#### Builder::get_series

```rust
pub fn get_series(self) -> Result<Builder<'a>>
```

#### Builder::set_series

```rust
pub fn set_series(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_series

```rust
pub fn init_series(self, size: u32) -> Builder<'a>
```

#### Builder::has_series

```rust
pub fn has_series(&self) -> bool
```

#### Builder::get_series_asin

```rust
pub fn get_series_asin(self) -> Result<Builder<'a>>
```

#### Builder::set_series_asin

```rust
pub fn set_series_asin(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_series_asin

```rust
pub fn init_series_asin(self, size: u32) -> Builder<'a>
```

#### Builder::has_series_asin

```rust
pub fn has_series_asin(&self) -> bool
```

#### Builder::get_asin

```rust
pub fn get_asin(self) -> Result<Builder<'a>>
```

#### Builder::set_asin

```rust
pub fn set_asin(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_asin

```rust
pub fn init_asin(self, size: u32) -> Builder<'a>
```

#### Builder::has_asin

```rust
pub fn has_asin(&self) -> bool
```

#### Builder::get_isbn

```rust
pub fn get_isbn(self) -> Result<Builder<'a>>
```

#### Builder::set_isbn

```rust
pub fn set_isbn(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_isbn

```rust
pub fn init_isbn(self, size: u32) -> Builder<'a>
```

#### Builder::has_isbn

```rust
pub fn has_isbn(&self) -> bool
```

#### Builder::get_region

```rust
pub fn get_region(self) -> Result<Builder<'a>>
```

#### Builder::set_region

```rust
pub fn set_region(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_region

```rust
pub fn init_region(self, size: u32) -> Builder<'a>
```

#### Builder::has_region

```rust
pub fn has_region(&self) -> bool
```

#### Builder::get_limit

```rust
pub fn get_limit(self) -> u32
```

#### Builder::set_limit

```rust
pub fn set_limit(&mut self, value: u32)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
