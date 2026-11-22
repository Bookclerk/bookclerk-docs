# bookclerk_model_pack

Layout of an embedding model on disk.

A text-embedding model has a vocabulary table with one row per token,
and that table is most of the download: 98 MB of bekko-a8m's 130 MB.
A library uses a handful of languages, so the table is shipped in
parts. One model directory holds:

| File | What it is |
| --- | --- |
| `model.json` | The [`PackManifest`]: what the model is and the size and SHA-256 of every other file. |
| graph (`core.onnx`) | The published ONNX graph with its token tables turned into graph inputs. |
| tokenizer (`tokenizer.spm.fb`) | The whole vocabulary as a SentencePiece Lite model. |
| base rows (`base.rows`) | Table rows every text needs: special tokens, single characters, bytes. |
| language rows (`<tag>.rows`) | The other rows the texts of one language use. |

The model plugin tokenizes with the whole vocabulary, looks each token's
row up in the row packs it has, and passes only those rows to the
graph. Token ids for a text are therefore the published model's own ids
whichever packs are present, and the output for a text whose rows are
all present is bit-identical to the published model's.

Everything here is sans-IO: functions take bytes and return validated
values, so the plugin, the pack builder's tests and the fuzzers run the
same code. See `docs/internals/model-packs.md`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`GraphSpec`](#graphspec) | struct | The cut ONNX graph and the names it uses. |
| [`LanguageTag`](#languagetag) | struct | A language tag naming a language pack: lowercase letters and digits in `-`-separated parts of 1 to 8 characters, at most 35 bytes (`en`, `pt-br`). |
| [`PackFile`](#packfile) | struct | One file of a model directory, with what its content must be. |
| [`PackFileName`](#packfilename) | struct | A file name inside a model directory: 1 to 64 lowercase letters, digits, `.`, `_` and `-`, starting with a letter or digit. |
| [`PackManifest`](#packmanifest) | struct | `model.json`: one embedding model, cut for row packs. |
| [`Sha256Hex`](#sha256hex) | struct | A SHA-256 digest as 64 lowercase hexadecimal characters. |
| [`TableSpec`](#tablespec) | struct | One token table the graph takes as an input: `[tokens, width]`. |
| [`TokenizerSpec`](#tokenizerspec) | struct | The tokenizer and how a text is framed for the model. |
| [`Rows`](#rows) | struct | A checked row pack held in memory. |
| [`Element`](#element) | enum | Element type of a token table. |
| [`Pooling`](#pooling) | enum | How the graph's output becomes one vector per text. |
| [`PackError`](#packerror) | enum | Why a model directory's file was refused. |
| [`encode_rows`](#encode_rows) | function | Writes a row pack. |
| [`revision`](#revision) | function | The revision of a loaded model: a digest of its description, which lists the digest of every file, and of the languages loaded. |
| [`MANIFEST_FILE`](#manifest_file) | constant | Name of the model description inside a model directory. |
| [`MAX_LANGUAGES`](#max_languages) | constant | Most language packs one model lists. |
| [`MAX_LITERAL_PIECES`](#max_literal_pieces) | constant | Most literal pieces a tokenizer lists (bekko-a8m has 142). |
| [`MAX_MANIFEST_BYTES`](#max_manifest_bytes) | constant | Most bytes of `model.json` that are read. |
| [`MAX_TABLES`](#max_tables) | constant | Most token tables one model has (bekko-a8m has two, Harrier three). |
| [`PACK_FORMAT`](#pack_format) | constant | Layout version this crate reads and writes (`format` in `model.json`). |
| [`MAX_ROWS`](#max_rows) | constant | Most rows one pack holds. Vocabularies stay well under this (bekko-a8m has 256,000 tokens). |
| [`MAX_ROW_BYTES`](#max_row_bytes) | constant | Most bytes one row has (bekko-a8m's is 388, Harrier's 410). |
| [`ROWS_HEADER_BYTES`](#rows_header_bytes) | constant | Bytes before the token ids: the magic and the two counts. |
| [`ROWS_MAGIC`](#rows_magic) | constant | First eight bytes of every row pack. |

## Structs

### GraphSpec

```rust
pub struct GraphSpec {
    pub model: PackFile,
    pub ids_input: String,
    pub mask_input: String,
    pub output: String,
    pub pooling: Pooling,
}
```

The cut ONNX graph and the names it uses.

**Fields**

- `model`: The graph file.
- `ids_input`: Input that takes token positions: 64-bit integers `[1, tokens]`,
  numbered `0..tokens` because the tables passed hold one row per
  token of the text, in order.
- `mask_input`: Input that takes the attention mask: 64-bit ones, `[1, tokens]`.
- `output`: Output read after a run: 32-bit floats.
- `pooling`: How that output becomes the text's vector.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### LanguageTag

```rust
pub struct LanguageTag(/* private */);
```

A language tag naming a language pack: lowercase letters and digits in
`-`-separated parts of 1 to 8 characters, at most 35 bytes (`en`,
`pt-br`).

#### LanguageTag::as_str

```rust
pub fn as_str(&self) -> &str
```

The tag as text.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `From<LanguageTag>`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`, `TryFrom<String>`

### PackFile

```rust
pub struct PackFile {
    pub file: PackFileName,
    pub bytes: u64,
    pub sha256: Sha256Hex,
}
```

One file of a model directory, with what its content must be.

**Fields**

- `file`: Name inside the model directory.
- `bytes`: Length in bytes.
- `sha256`: SHA-256 of the content.

#### PackFile::describe

```rust
pub fn describe(file: PackFileName, content: &[u8]) -> Self
```

Describes `content` under the name `file`.

#### PackFile::verify

```rust
pub fn verify(&self, content: &[u8]) -> Result<(), PackError>
```

Checks that `content` is the file the description lists.

##### Errors

[`PackError::FileMismatch`](#packerror) when the length or the digest differs.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### PackFileName

```rust
pub struct PackFileName(/* private */);
```

A file name inside a model directory: 1 to 64 lowercase letters,
digits, `.`, `_` and `-`, starting with a letter or digit.

The grammar leaves no way to name a parent directory, a hidden file or
a path, so joining it to the model directory stays inside it.

#### PackFileName::as_str

```rust
pub fn as_str(&self) -> &str
```

The name as text.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `From<PackFileName>`, `Hash`, `Ord`, `PartialEq`, `PartialOrd`, `Serialize`, `TryFrom<String>`

### PackManifest

```rust
pub struct PackManifest {
    pub format: u32,
    pub model_id: String,
    pub dimensions: u32,
    pub max_tokens: u32,
    pub graph: GraphSpec,
    pub tokenizer: TokenizerSpec,
    pub tables: Vec<TableSpec>,
    pub base_rows: PackFile,
    pub languages: BTreeMap<LanguageTag, PackFile>,
}
```

`model.json`: one embedding model, cut for row packs.

**Fields**

- `format`: Layout version; [`PACK_FORMAT`](#pack_format).
- `model_id`: Model id within the plugin (`bekko-a8m`).
- `dimensions`: Numbers per vector.
- `max_tokens`: Tokens read from one text, counting the framing pieces; the rest of
  the text is cut off.
- `graph`: The cut graph.
- `tokenizer`: The tokenizer.
- `tables`: The token tables, in the order their rows are concatenated in a row
  pack.
- `base_rows`: Rows every text needs.
- `languages`: The row pack of each language, present on disk or not.

#### PackManifest::parse

```rust
pub fn parse(bytes: &[u8]) -> Result<Self, PackError>
```

Reads and checks a `model.json`.

##### Arguments

* `bytes` - The file's content.

##### Errors

[`PackError::ManifestTooLarge`](#packerror), [`PackError::ManifestSyntax`](#packerror),
[`PackError::UnsupportedFormat`](#packerror) or the first
[`PackError::ManifestField`](#packerror) rule broken.

#### PackManifest::to_json

```rust
pub fn to_json(&self) -> Result<Vec<u8>, PackError>
```

Writes the description as JSON that [`PackManifest::parse`](#packmanifestparse) reads.

##### Errors

The first rule the description breaks, as [`PackManifest::parse`](#packmanifestparse)
would report it.

#### PackManifest::row_bytes

```rust
pub fn row_bytes(&self) -> usize
```

Bytes of one row of a row pack: every table's row, concatenated.

Within 1 to [`MAX_ROW_BYTES`](#max_row_bytes) for a checked description.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### Sha256Hex

```rust
pub struct Sha256Hex(/* private */);
```

A SHA-256 digest as 64 lowercase hexadecimal characters.

#### Sha256Hex::of

```rust
pub fn of(bytes: &[u8]) -> Self
```

The digest of `bytes`.

#### Sha256Hex::as_str

```rust
pub fn as_str(&self) -> &str
```

The digest as text.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `From<Sha256Hex>`, `Hash`, `PartialEq`, `Serialize`, `TryFrom<String>`

### TableSpec

```rust
pub struct TableSpec {
    pub input: String,
    pub element: Element,
    pub width: u32,
}
```

One token table the graph takes as an input: `[tokens, width]`.

**Fields**

- `input`: Graph input that takes the rows.
- `element`: Element type.
- `width`: Elements per row.

#### TableSpec::row_bytes

```rust
pub fn row_bytes(&self) -> Option<usize>
```

Bytes one row of this table takes, or `None` on overflow.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### TokenizerSpec

```rust
pub struct TokenizerSpec {
    pub model: PackFile,
    pub literal_pieces: Vec<String>,
    pub prefix_pieces: Vec<String>,
    pub suffix_pieces: Vec<String>,
    pub document_prefix: String,
    pub query_prefix: String,
}
```

The tokenizer and how a text is framed for the model.

**Fields**

- `model`: The SentencePiece Lite model with the whole vocabulary.
- `literal_pieces`: Pieces matched in the text before the tokenizer runs, longest match
  first; each becomes its own token and the text between them is
  tokenized segment by segment. A segment that starts with a space
  loses that space (a segment of only that space becomes the word
  boundary piece), as the published tokenizer does.
- `prefix_pieces`: Pieces put before the text's tokens (`<bos>`).
- `suffix_pieces`: Pieces put after the text's tokens (`<eos>`).
- `document_prefix`: Text put before a document before tokenizing.
- `query_prefix`: Text put before a search query before tokenizing.

Implements: `Clone`, `Debug`, `Deserialize<'de>`, `Eq`, `PartialEq`, `Serialize`

### Rows

```rust
pub struct Rows {
    /* private fields */
}
```

A checked row pack held in memory.

Built by [`Rows::parse`](#rowsparse); afterwards every lookup is a binary search
over the id column and cannot fail on the pack's content.

#### Rows::parse

```rust
pub fn parse(bytes: Vec<u8>, row_bytes: usize) -> Result<Self, PackError>
```

Checks a row pack and takes ownership of it.

##### Arguments

* `bytes` - The whole file.
* `row_bytes` - Bytes per row the model's tables add up to.

##### Errors

The first rule of the layout the file breaks; see [`PackError`](#packerror).

#### Rows::len

```rust
pub fn len(&self) -> usize
```

Rows in the pack.

#### Rows::is_empty

```rust
pub fn is_empty(&self) -> bool
```

Whether the pack has no rows.

#### Rows::row_bytes

```rust
pub fn row_bytes(&self) -> usize
```

Bytes per row.

#### Rows::ids

```rust
pub fn ids(&self) -> impl Iterator<Item = u32> + '_
```

The token ids in the pack, ascending.

#### Rows::row

```rust
pub fn row(&self, id: u32) -> Option<&[u8]>
```

The row of token `id`.

##### Arguments

* `id` - A token id of the model's vocabulary.

##### Returns

The row's [`Rows::row_bytes`](#rowsrow_bytes) bytes, or `None` when the pack does not
hold the token.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Enums

### Element

```rust
pub enum Element {
    I8,
    U8,
    F32,
}
```

Element type of a token table.

**Variants**

- `I8`: Signed 8-bit integers.
- `U8`: Unsigned 8-bit integers.
- `F32`: Little-endian IEEE 754 32-bit floats.

#### Element::size

```rust
pub fn size(self) -> usize
```

Bytes one element takes.

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `PartialEq`, `Serialize`

### Pooling

```rust
pub enum Pooling {
    Mean,
    None,
}
```

How the graph's output becomes one vector per text.

**Variants**

- `Mean`: The output has one vector per token (`[1, tokens, dimensions]`);
  the text's vector is their mean.
- `None`: The output already is the text's vector (`[1, dimensions]`).

Implements: `Clone`, `Copy`, `Debug`, `Deserialize<'de>`, `Eq`, `Hash`, `PartialEq`, `Serialize`

### PackError

```rust
pub enum PackError {
    ManifestTooLarge,
    ManifestSyntax(String),
    UnsupportedFormat(u32),
    ManifestField { field: &'static str, reason: &'static str },
    FileMismatch(String),
    RowsTruncated,
    RowsMagic,
    RowsWidth { expected: usize, found: usize },
    RowsTooLarge,
    RowsLength,
    RowsOrder,
}
```

Why a model directory's file was refused.

**Variants**

- `ManifestTooLarge`: `model.json` is longer than [`MAX_MANIFEST_BYTES`](#max_manifest_bytes).
- `ManifestSyntax`: `model.json` is not the JSON document this crate reads.
- `UnsupportedFormat`: `model.json` was written for another layout version.
- `ManifestField`: A field of `model.json` breaks a rule; the text names the field.
  - `field`: Dotted path of the field (`graph.output`).
  - `reason`: The rule it breaks.
- `FileMismatch`: A file's length or SHA-256 differs from what `model.json` lists.
- `RowsTruncated`: A row pack is shorter than its header.
- `RowsMagic`: A row pack does not start with [`ROWS_MAGIC`](#rows_magic).
- `RowsWidth`: A row pack's rows are not the length the model's tables give.
  - `expected`: Row length the model's tables add up to.
  - `found`: Row length the pack's header states.
- `RowsTooLarge`: A row pack's row count or row length is over the limits.
- `RowsLength`: A row pack's length differs from what its header implies.
- `RowsOrder`: A row pack's token ids are not strictly ascending.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

## Functions

### encode_rows

```rust
pub fn encode_rows(row_bytes: usize, rows: &[(u32, &[u8])]) -> Result<Vec<u8>, PackError>
```

Writes a row pack.

#### Arguments

* `row_bytes` - Bytes per row.
* `rows` - Token id and row, in strictly ascending id order.

#### Returns

The file, which [`Rows::parse`](#rowsparse) accepts.

#### Errors

[`PackError::RowsTooLarge`](#packerror) over the limits, [`PackError::RowsOrder`](#packerror)
when the ids do not ascend, and [`PackError::RowsWidth`](#packerror) for a row of
another length.

### revision

```rust
pub fn revision(manifest_bytes: &[u8], languages: &[LanguageTag]) -> String
```

The revision of a loaded model: a digest of its description, which
lists the digest of every file, and of the languages loaded.

Two outputs are comparable only when their revisions match, so the
revision changes when the weights, the tokenizer or the set of loaded
language packs change.

#### Arguments

* `manifest_bytes` - The bytes of `model.json`.
* `languages` - Tags of the language packs loaded, in any order.

#### Returns

32 lowercase hexadecimal characters.

## Constants

### MANIFEST_FILE

```rust
pub const MANIFEST_FILE: &str = "model.json";
```

Name of the model description inside a model directory.

### MAX_LANGUAGES

```rust
pub const MAX_LANGUAGES: usize = 64;
```

Most language packs one model lists.

### MAX_LITERAL_PIECES

```rust
pub const MAX_LITERAL_PIECES: usize = 1024;
```

Most literal pieces a tokenizer lists (bekko-a8m has 142).

### MAX_MANIFEST_BYTES

```rust
pub const MAX_MANIFEST_BYTES: usize = _;
```

Most bytes of `model.json` that are read.

### MAX_TABLES

```rust
pub const MAX_TABLES: usize = 8;
```

Most token tables one model has (bekko-a8m has two, Harrier three).

### PACK_FORMAT

```rust
pub const PACK_FORMAT: u32 = 1;
```

Layout version this crate reads and writes (`format` in `model.json`).

### MAX_ROWS

```rust
pub const MAX_ROWS: usize = 4_000_000;
```

Most rows one pack holds. Vocabularies stay well under this (bekko-a8m
has 256,000 tokens).

### MAX_ROW_BYTES

```rust
pub const MAX_ROW_BYTES: usize = _;
```

Most bytes one row has (bekko-a8m's is 388, Harrier's 410).

### ROWS_HEADER_BYTES

```rust
pub const ROWS_HEADER_BYTES: usize = 16;
```

Bytes before the token ids: the magic and the two counts.

### ROWS_MAGIC

```rust
pub const ROWS_MAGIC: [u8; 8] = _;
```

First eight bytes of every row pack.
