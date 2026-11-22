# bookclerk_plugin_ai_embedder::model

A model directory: what is in it, and the model loaded from it.

[`inspect`](#inspect) is cheap: it reads `model.json` and looks at file sizes, so
the plugin can list its models without loading one. [`load`](#load) reads and
verifies every file against the SHA-256 the description lists, then
builds the tokenizer, the row packs and the graph.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Listing`](#listing) | struct | What `inspect` found in a model directory. |
| [`Embedding`](#embedding) | struct | One text's vector and whether a pack was missing for it. |
| [`LoadedModel`](#loadedmodel) | struct | A model in memory. |
| [`ModelError`](#modelerror) | enum | Why a model directory could not be listed, loaded or run. |
| [`inspect`](#inspect) | function | Looks at a model directory without loading the model. |
| [`load`](#load) | function | Loads the model a listing describes. |

## Structs

### Listing

```rust
pub struct Listing {
    /* private fields */
}
```

What [`inspect`](#inspect) found in a model directory.

#### Listing::revision

```rust
pub fn revision(&self) -> &str
```

Revision of the model with the language packs present.

#### Listing::info

```rust
pub fn info(&self) -> AiModel
```

The model as the plugin lists it to the host.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### Embedding

```rust
pub struct Embedding {
    pub vector: Vec<f32>,
    pub fallback: bool,
}
```

One text's vector and whether a pack was missing for it.

**Fields**

- `vector`: The vector, scaled to length one.
- `fallback`: True when a token of the text had no row in the loaded packs and
  was spelled with smaller pieces.

Implements: `Clone`, `Debug`, `PartialEq`

### LoadedModel

```rust
pub struct LoadedModel {
    /* private fields */
}
```

A model in memory.

#### LoadedModel::revision

```rust
pub fn revision(&self) -> &str
```

Revision of the listing the model was loaded from.

#### LoadedModel::dimensions

```rust
pub fn dimensions(&self) -> u32
```

Numbers per vector.

#### LoadedModel::embed

```rust
pub fn embed(&self, text: &str, role: Role) -> Result<Embedding, ModelError>
```

Embeds one text.

##### Arguments

* `text` - The text.
* `role` - Document or query.

##### Errors

When the tokenizer or the graph fails, or the output is not finite.

Implements: `Debug`

## Enums

### ModelError

```rust
pub enum ModelError {
    Io { file: String, reason: String },
    Pack(PackError),
    Name { directory: String, model: String },
    Incomplete(String),
    Text(TextError),
    Engine(EngineError),
    BaseRow(u32),
    NotFinite,
}
```

Why a model directory could not be listed, loaded or run.

**Variants**

- `Io`: A file could not be read.
  - `file`: File name inside the model directory.
  - `reason`: The operating system's reason.
- `Pack`: A file is not what the layout or the model description says.
- `Name`: The directory's name is not the model id its description gives.
  - `directory`: The directory's name.
  - `model`: `model_id` of its `model.json`.
- `Incomplete`: A file every text needs is missing or has another length.
- `Text`: The tokenizer could not be set up or failed on a text.
- `Engine`: ONNX Runtime could not load or run the graph.
- `BaseRow`: The base rows lack a token every text needs.
- `NotFinite`: The graph gave a number that is not finite.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `From<EngineError>`, `From<PackError>`, `From<TextError>`, `PartialEq`

## Functions

### inspect

```rust
pub fn inspect(dir: &Path) -> Result<Listing, ModelError>
```

Looks at a model directory without loading the model.

#### Arguments

* `dir` - `models/<model id>` in the plugin's data directory.

#### Returns

The description and the language packs present on disk.

#### Errors

When `model.json` is missing or invalid, the directory is named
differently from the model, or the graph, the tokenizer or the base rows
are missing or have another length than listed.

### load

```rust
pub fn load(
    listing: &Listing,
    environment: &Environment,
) -> Result<LoadedModel, ModelError>
```

Loads the model a listing describes.

#### Arguments

* `listing` - What [`inspect`](#inspect) found.
* `environment` - The process's ONNX Runtime.

#### Errors

When a file changed since the listing or fails its SHA-256, the
tokenizer or the graph cannot be set up, or the base rows lack a token
every text needs.
