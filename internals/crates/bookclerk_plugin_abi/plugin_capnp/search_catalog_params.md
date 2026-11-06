# bookclerk_plugin_abi::plugin_capnp::search_catalog_params

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

#### Reader::get_query

```rust
pub fn get_query(self) -> Result<Reader<'a>>
```

#### Reader::has_query

```rust
pub fn has_query(&self) -> bool
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

#### Reader::get_page

```rust
pub fn get_page(self) -> u32
```

#### Reader::get_sort

```rust
pub fn get_sort(self) -> Result<CatalogSort, NotInSchema>
```

#### Reader::get_field

```rust
pub fn get_field(self) -> Result<CatalogField, NotInSchema>
```

#### Reader::get_language

```rust
pub fn get_language(self) -> Result<Reader<'a>>
```

#### Reader::has_language

```rust
pub fn has_language(&self) -> bool
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

#### Builder::get_query

```rust
pub fn get_query(self) -> Result<Builder<'a>>
```

#### Builder::set_query

```rust
pub fn set_query(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_query

```rust
pub fn init_query(self, size: u32) -> Builder<'a>
```

#### Builder::has_query

```rust
pub fn has_query(&self) -> bool
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

#### Builder::get_page

```rust
pub fn get_page(self) -> u32
```

#### Builder::set_page

```rust
pub fn set_page(&mut self, value: u32)
```

#### Builder::get_sort

```rust
pub fn get_sort(self) -> Result<CatalogSort, NotInSchema>
```

#### Builder::set_sort

```rust
pub fn set_sort(&mut self, value: CatalogSort)
```

#### Builder::get_field

```rust
pub fn get_field(self) -> Result<CatalogField, NotInSchema>
```

#### Builder::set_field

```rust
pub fn set_field(&mut self, value: CatalogField)
```

#### Builder::get_language

```rust
pub fn get_language(self) -> Result<Builder<'a>>
```

#### Builder::set_language

```rust
pub fn set_language(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_language

```rust
pub fn init_language(self, size: u32) -> Builder<'a>
```

#### Builder::has_language

```rust
pub fn has_language(&self) -> bool
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
