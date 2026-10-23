**bookclerk_media > metadata**

# Module: metadata

## Contents

**Structs**

- [`FixupRequest`](#fixuprequest) - Request to fix up audiobook metadata after decrypt / download.

**Functions**

- [`bookclerk_tool_tag`](#bookclerk_tool_tag) - `bookclerk <version>` string embedded in encoder / freeform TOOL tags.
- [`fixup_audiobook`](#fixup_audiobook) - Apply metadata tags, optional cover embed, and optional chapters.

**Constants**

- [`BOOKCLERK_TOOL_NAME`](#bookclerk_tool_name) - Product / CLI name written into tool attribution tags.

---

## bookclerk_media::metadata::BOOKCLERK_TOOL_NAME

*Constant*: `&str`

Product / CLI name written into tool attribution tags.



## bookclerk_media::metadata::FixupRequest

*Struct*

Request to fix up audiobook metadata after decrypt / download.

**Fields:**
- `input: std::path::PathBuf` - Absolute path of the clear media file to tag (copied/written to output).
- `output: std::path::PathBuf` - Absolute path where the tagged file should be written.
- `title: String` - Primary title embedded in the container tags.
- `author: Option<String>` - Author / artist tag when known.
- `narrator: Option<String>` - Narrator tag when known (ABS / iTunes freeform).
- `cover: Option<std::path::PathBuf>` - Optional cover image path to embed (JPEG/PNG).
- `chapters: Vec<(String, u64)>` - Chapter titles + start offsets in milliseconds (embedded for M4B).
- `replace_chapters: bool` - When true, replace any existing M4B chapter list/track with `Self::chapters`
- `subtitle: Option<String>` - Subtitle / album subtitle when known.
- `publisher: Option<String>` - Publisher imprint for the publisher tag.
- `year: Option<String>` - Publish year as a string (e.g. `"2011"`).
- `genre: Option<String>` - Genre string; multiple genres should use `;` separators for ABS.
- `series: Option<String>` - Series name written to Tone / ABS freeform atoms when set.
- `series_index: Option<String>` - Series sequence string (e.g. `"1"`) for ABS freeform atoms.
- `asin: Option<String>` - Amazon ASIN written to freeform metadata when set.
- `isbn: Option<String>` - ISBN written to freeform metadata when set.
- `description: Option<String>` - Long description / comment body when set.
- `language: Option<String>` - Content language tag when set.
- `tool: Option<String>` - Tool attribution (`bookclerk 0.1.0`). Defaults to [`bookclerk_tool_tag`](metadata.md#bookclerk_mediametadatabookclerk_tool_tag) when `None`.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> FixupRequest`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_media::metadata::bookclerk_tool_tag

*Function*

`bookclerk <version>` string embedded in encoder / freeform TOOL tags.

```rust
fn bookclerk_tool_tag() -> String
```



## bookclerk_media::metadata::fixup_audiobook

*Function*

Apply metadata tags, optional cover embed, and optional chapters.

Runs in a confined media worker; see the [crate](bookclerk_media.md) documentation.

# Errors

Returns `MediaError::InputMissing` when the input does not exist,
`MediaError::OutputMissing` when the worker reported success but wrote
nothing, and propagates tag-writing and worker failures otherwise.

```rust
fn fixup_audiobook(req: FixupRequest) -> crate::error::Result<crate::MediaOutcome>
```



