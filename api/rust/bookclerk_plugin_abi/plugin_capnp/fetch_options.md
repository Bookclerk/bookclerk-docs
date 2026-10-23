# bookclerk_plugin_abi::plugin_capnp::fetch_options

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

#### Reader::get_widevine

```rust
pub fn get_widevine(self) -> bool
```

#### Reader::get_xhe_aac

```rust
pub fn get_xhe_aac(self) -> bool
```

#### Reader::get_widevine_cdm_path

```rust
pub fn get_widevine_cdm_path(self) -> Result<Reader<'a>>
```

#### Reader::has_widevine_cdm_path

```rust
pub fn has_widevine_cdm_path(&self) -> bool
```

#### Reader::get_widevine_cdm_provider

```rust
pub fn get_widevine_cdm_provider(self) -> Result<Reader<'a>>
```

#### Reader::has_widevine_cdm_provider

```rust
pub fn has_widevine_cdm_provider(&self) -> bool
```

#### Reader::get_download_cover

```rust
pub fn get_download_cover(self) -> bool
```

#### Reader::get_download_pdf

```rust
pub fn get_download_pdf(self) -> bool
```

#### Reader::get_cover_size

```rust
pub fn get_cover_size(self) -> Result<Reader<'a>>
```

#### Reader::has_cover_size

```rust
pub fn has_cover_size(&self) -> bool
```

#### Reader::get_chapter_layout

```rust
pub fn get_chapter_layout(self) -> Result<Reader<'a>>
```

#### Reader::has_chapter_layout

```rust
pub fn has_chapter_layout(&self) -> bool
```

#### Reader::get_strip_audible_brand_audio

```rust
pub fn get_strip_audible_brand_audio(self) -> bool
```

#### Reader::get_download_clips_bookmarks

```rust
pub fn get_download_clips_bookmarks(self) -> bool
```

#### Reader::get_retain_aax_file

```rust
pub fn get_retain_aax_file(self) -> bool
```

#### Reader::get_download_speed_limit_kbps

```rust
pub fn get_download_speed_limit_kbps(self) -> u32
```

#### Reader::get_save_metadata_json

```rust
pub fn get_save_metadata_json(self) -> bool
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

#### Builder::get_widevine

```rust
pub fn get_widevine(self) -> bool
```

#### Builder::set_widevine

```rust
pub fn set_widevine(&mut self, value: bool)
```

#### Builder::get_xhe_aac

```rust
pub fn get_xhe_aac(self) -> bool
```

#### Builder::set_xhe_aac

```rust
pub fn set_xhe_aac(&mut self, value: bool)
```

#### Builder::get_widevine_cdm_path

```rust
pub fn get_widevine_cdm_path(self) -> Result<Builder<'a>>
```

#### Builder::set_widevine_cdm_path

```rust
pub fn set_widevine_cdm_path(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_widevine_cdm_path

```rust
pub fn init_widevine_cdm_path(self, size: u32) -> Builder<'a>
```

#### Builder::has_widevine_cdm_path

```rust
pub fn has_widevine_cdm_path(&self) -> bool
```

#### Builder::get_widevine_cdm_provider

```rust
pub fn get_widevine_cdm_provider(self) -> Result<Builder<'a>>
```

#### Builder::set_widevine_cdm_provider

```rust
pub fn set_widevine_cdm_provider(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_widevine_cdm_provider

```rust
pub fn init_widevine_cdm_provider(self, size: u32) -> Builder<'a>
```

#### Builder::has_widevine_cdm_provider

```rust
pub fn has_widevine_cdm_provider(&self) -> bool
```

#### Builder::get_download_cover

```rust
pub fn get_download_cover(self) -> bool
```

#### Builder::set_download_cover

```rust
pub fn set_download_cover(&mut self, value: bool)
```

#### Builder::get_download_pdf

```rust
pub fn get_download_pdf(self) -> bool
```

#### Builder::set_download_pdf

```rust
pub fn set_download_pdf(&mut self, value: bool)
```

#### Builder::get_cover_size

```rust
pub fn get_cover_size(self) -> Result<Builder<'a>>
```

#### Builder::set_cover_size

```rust
pub fn set_cover_size(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_cover_size

```rust
pub fn init_cover_size(self, size: u32) -> Builder<'a>
```

#### Builder::has_cover_size

```rust
pub fn has_cover_size(&self) -> bool
```

#### Builder::get_chapter_layout

```rust
pub fn get_chapter_layout(self) -> Result<Builder<'a>>
```

#### Builder::set_chapter_layout

```rust
pub fn set_chapter_layout(&mut self, value: impl SetterInput<Owned>)
```

#### Builder::init_chapter_layout

```rust
pub fn init_chapter_layout(self, size: u32) -> Builder<'a>
```

#### Builder::has_chapter_layout

```rust
pub fn has_chapter_layout(&self) -> bool
```

#### Builder::get_strip_audible_brand_audio

```rust
pub fn get_strip_audible_brand_audio(self) -> bool
```

#### Builder::set_strip_audible_brand_audio

```rust
pub fn set_strip_audible_brand_audio(&mut self, value: bool)
```

#### Builder::get_download_clips_bookmarks

```rust
pub fn get_download_clips_bookmarks(self) -> bool
```

#### Builder::set_download_clips_bookmarks

```rust
pub fn set_download_clips_bookmarks(&mut self, value: bool)
```

#### Builder::get_retain_aax_file

```rust
pub fn get_retain_aax_file(self) -> bool
```

#### Builder::set_retain_aax_file

```rust
pub fn set_retain_aax_file(&mut self, value: bool)
```

#### Builder::get_download_speed_limit_kbps

```rust
pub fn get_download_speed_limit_kbps(self) -> u32
```

#### Builder::set_download_speed_limit_kbps

```rust
pub fn set_download_speed_limit_kbps(&mut self, value: u32)
```

#### Builder::get_save_metadata_json

```rust
pub fn get_save_metadata_json(self) -> bool
```

#### Builder::set_save_metadata_json

```rust
pub fn set_save_metadata_json(&mut self, value: bool)
```

Implements: `From<Builder<'a>>`, `From<StructBuilder<'a>>`, `FromPointerBuilder<'a>`, `HasStructSize`, `HasTypeId`, `ImbueMut<'a>`

### Pipeline

```rust
pub struct Pipeline {
    /* private fields */
}
```

Implements: `FromTypelessPipeline`
