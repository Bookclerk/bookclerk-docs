# bookclerk_plugin_abi::plugin_capnp::metadata_file_record

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

#### Reader::get_present

```rust
pub fn get_present(self) -> Result<Reader<'a, MetadataField>>
```

#### Reader::has_present

```rust
pub fn has_present(&self) -> bool
```

#### Reader::get_title

```rust
pub fn get_title(self) -> Result<Reader<'a>>
```

#### Reader::has_title

```rust
pub fn has_title(&self) -> bool
```

#### Reader::get_subtitle

```rust
pub fn get_subtitle(self) -> Result<Reader<'a>>
```

#### Reader::has_subtitle

```rust
pub fn has_subtitle(&self) -> bool
```

#### Reader::get_description

```rust
pub fn get_description(self) -> Result<Reader<'a>>
```

#### Reader::has_description

```rust
pub fn has_description(&self) -> bool
```

#### Reader::get_publisher

```rust
pub fn get_publisher(self) -> Result<Reader<'a>>
```

#### Reader::has_publisher

```rust
pub fn has_publisher(&self) -> bool
```

#### Reader::get_published_at

```rust
pub fn get_published_at(self) -> Result<Reader<'a>>
```

#### Reader::has_published_at

```rust
pub fn has_published_at(&self) -> bool
```

#### Reader::get_language

```rust
pub fn get_language(self) -> Result<Reader<'a>>
```

#### Reader::has_language

```rust
pub fn has_language(&self) -> bool
```

#### Reader::get_genres

```rust
pub fn get_genres(self) -> Result<Reader<'a>>
```

#### Reader::has_genres

```rust
pub fn has_genres(&self) -> bool
```

#### Reader::get_tags

```rust
pub fn get_tags(self) -> Result<Reader<'a>>
```

#### Reader::has_tags

```rust
pub fn has_tags(&self) -> bool
```

#### Reader::get_abridgement

```rust
pub fn get_abridgement(self) -> Result<Abridgement, NotInSchema>
```

#### Reader::get_expression_kind

```rust
pub fn get_expression_kind(self) -> Result<ExpressionKind, NotInSchema>
```

#### Reader::get_identifiers

```rust
pub fn get_identifiers(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_identifiers

```rust
pub fn has_identifiers(&self) -> bool
```

#### Reader::get_credits

```rust
pub fn get_credits(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_credits

```rust
pub fn has_credits(&self) -> bool
```

#### Reader::get_credit_roles

```rust
pub fn get_credit_roles(self) -> Result<Reader<'a, CreditRole>>
```

#### Reader::has_credit_roles

```rust
pub fn has_credit_roles(&self) -> bool
```

#### Reader::get_series_entries

```rust
pub fn get_series_entries(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_series_entries

```rust
pub fn has_series_entries(&self) -> bool
```

#### Reader::get_chapters

```rust
pub fn get_chapters(self) -> Result<Reader<'a, Owned>>
```

#### Reader::has_chapters

```rust
pub fn has_chapters(&self) -> bool
```

#### Reader::get_duration_ms

```rust
pub fn get_duration_ms(self) -> u64
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

#### Builder::get_present

```rust
pub fn get_present(self) -> Result<Builder<'a, MetadataField>>
```

#### Builder::set_present

```rust
pub fn set_present(&mut self, value: impl SetterInput<Owned<MetadataField>>) -> Result<()>
```

#### Builder::init_present

```rust
pub fn init_present(self, size: u32) -> Builder<'a, MetadataField>
```

#### Builder::has_present

```rust
pub fn has_present(&self) -> bool
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

#### Builder::get_subtitle

```rust
pub fn get_subtitle(self) -> Result<Builder<'a>>
```

#### Builder::set_subtitle

```rust
pub fn set_subtitle(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_subtitle

```rust
pub fn init_subtitle(self, size: u32) -> Builder<'a>
```

#### Builder::has_subtitle

```rust
pub fn has_subtitle(&self) -> bool
```

#### Builder::get_description

```rust
pub fn get_description(self) -> Result<Builder<'a>>
```

#### Builder::set_description

```rust
pub fn set_description(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_description

```rust
pub fn init_description(self, size: u32) -> Builder<'a>
```

#### Builder::has_description

```rust
pub fn has_description(&self) -> bool
```

#### Builder::get_publisher

```rust
pub fn get_publisher(self) -> Result<Builder<'a>>
```

#### Builder::set_publisher

```rust
pub fn set_publisher(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_publisher

```rust
pub fn init_publisher(self, size: u32) -> Builder<'a>
```

#### Builder::has_publisher

```rust
pub fn has_publisher(&self) -> bool
```

#### Builder::get_published_at

```rust
pub fn get_published_at(self) -> Result<Builder<'a>>
```

#### Builder::set_published_at

```rust
pub fn set_published_at(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_published_at

```rust
pub fn init_published_at(self, size: u32) -> Builder<'a>
```

#### Builder::has_published_at

```rust
pub fn has_published_at(&self) -> bool
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

#### Builder::get_genres

```rust
pub fn get_genres(self) -> Result<Builder<'a>>
```

#### Builder::set_genres

```rust
pub fn set_genres(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_genres

```rust
pub fn init_genres(self, size: u32) -> Builder<'a>
```

#### Builder::has_genres

```rust
pub fn has_genres(&self) -> bool
```

#### Builder::get_tags

```rust
pub fn get_tags(self) -> Result<Builder<'a>>
```

#### Builder::set_tags

```rust
pub fn set_tags(&mut self, value: impl SetterInput<Owned>) -> Result<()>
```

#### Builder::init_tags

```rust
pub fn init_tags(self, size: u32) -> Builder<'a>
```

#### Builder::has_tags

```rust
pub fn has_tags(&self) -> bool
```

#### Builder::get_abridgement

```rust
pub fn get_abridgement(self) -> Result<Abridgement, NotInSchema>
```

#### Builder::set_abridgement

```rust
pub fn set_abridgement(&mut self, value: Abridgement)
```

#### Builder::get_expression_kind

```rust
pub fn get_expression_kind(self) -> Result<ExpressionKind, NotInSchema>
```

#### Builder::set_expression_kind

```rust
pub fn set_expression_kind(&mut self, value: ExpressionKind)
```

#### Builder::get_identifiers

```rust
pub fn get_identifiers(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_identifiers

```rust
pub fn set_identifiers(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_identifiers

```rust
pub fn init_identifiers(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_identifiers

```rust
pub fn has_identifiers(&self) -> bool
```

#### Builder::get_credits

```rust
pub fn get_credits(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_credits

```rust
pub fn set_credits(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_credits

```rust
pub fn init_credits(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_credits

```rust
pub fn has_credits(&self) -> bool
```

#### Builder::get_credit_roles

```rust
pub fn get_credit_roles(self) -> Result<Builder<'a, CreditRole>>
```

#### Builder::set_credit_roles

```rust
pub fn set_credit_roles(
    &mut self,
    value: impl SetterInput<Owned<CreditRole>>,
) -> Result<()>
```

#### Builder::init_credit_roles

```rust
pub fn init_credit_roles(self, size: u32) -> Builder<'a, CreditRole>
```

#### Builder::has_credit_roles

```rust
pub fn has_credit_roles(&self) -> bool
```

#### Builder::get_series_entries

```rust
pub fn get_series_entries(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_series_entries

```rust
pub fn set_series_entries(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_series_entries

```rust
pub fn init_series_entries(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_series_entries

```rust
pub fn has_series_entries(&self) -> bool
```

#### Builder::get_chapters

```rust
pub fn get_chapters(self) -> Result<Builder<'a, Owned>>
```

#### Builder::set_chapters

```rust
pub fn set_chapters(&mut self, value: Reader<'_, Owned>) -> Result<()>
```

#### Builder::init_chapters

```rust
pub fn init_chapters(self, size: u32) -> Builder<'a, Owned>
```

#### Builder::has_chapters

```rust
pub fn has_chapters(&self) -> bool
```

#### Builder::get_duration_ms

```rust
pub fn get_duration_ms(self) -> u64
```

#### Builder::set_duration_ms

```rust
pub fn set_duration_ms(&mut self, value: u64)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
