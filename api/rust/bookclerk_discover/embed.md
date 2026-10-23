**bookclerk_discover > embed**

# Module: embed

## Contents

**Structs**

- [`CosineHit`](#cosinehit) - One cosine-similarity neighbor from an embedding query.
- [`HashEmbedder`](#hashembedder) - Deterministic offline embedder for discovery similarity.
- [`OnnxEmbedder`](#onnxembedder) - ONNX MiniLM embedder (fastembed). Loaded on demand; drop to free RAM.

**Functions**

- [`default_embedding_model_id`](#default_embedding_model_id) - Preferred model id for this build (ONNX MiniLM; runtime may fall back).
- [`embed_dirty_works`](#embed_dirty_works) - Embed works whose text hash changed (or are missing).
- [`embedding_model_id`](#embedding_model_id) - Resolve the configured model id.
- [`open_embedder`](#open_embedder) - Open an embedder for the current build / config.
- [`text_for_work`](#text_for_work) - Build the text blob embedded for a work.

**Traits**

- [`Embedder`](#embedder) - Produce dense vectors for recommendation / similarity search.

**Constants**

- [`MODEL_ALL_MINILM_L6_V2_Q`](#model_all_minilm_l6_v2_q) - Quantized MiniLM-L6-v2 — ~22 MB ONNX, ~50 MB RAM, 384 dims.
- [`MODEL_LOCAL_HASH_V1`](#model_local_hash_v1) - Local hash embedder id used when ONNX is unavailable / disabled.

---

## bookclerk_discover::embed::CosineHit

*Struct*

One cosine-similarity neighbor from an embedding query.

**Fields:**
- `target_id: String` - Library entity id of the similar work (`works.id`).
- `score: f32` - Ranking score (higher is better); units are algorithm-specific.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> CosineHit`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_discover::embed::Embedder

*Trait*

Produce dense vectors for recommendation / similarity search.

**Methods:**

- `model_id`: Stable embedding model id stored alongside vectors in SQLite.
- `dimensions`: Length of each embedding vector produced by this backend.
- `embed`: Embeds each input text into a dense float vector of `Self::dimensions` length.



## bookclerk_discover::embed::HashEmbedder

*Struct*

Deterministic offline embedder for discovery similarity.

Not semantic like MiniLM — used when ONNX fails to load or when
`prefer_onnx` is false. Still enables the full pipeline (hash → store →
cosine) for CI and constrained hosts.

**Methods:**

- `fn new(dims: usize) -> Self` - Creates a hash embedder with the given vector length (clamped to 8..=384).

**Trait Implementations:**

- **Embedder**
  - `fn model_id(self: &Self) -> &str`
  - `fn dimensions(self: &Self) -> usize`
  - `fn embed(self: & mut Self, texts: &[String]) -> std::result::Result<Vec<Vec<f32>>, DiscoverError>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> HashEmbedder`



## bookclerk_discover::embed::MODEL_ALL_MINILM_L6_V2_Q

*Constant*: `&str`

Quantized MiniLM-L6-v2 — ~22 MB ONNX, ~50 MB RAM, 384 dims.



## bookclerk_discover::embed::MODEL_LOCAL_HASH_V1

*Constant*: `&str`

Local hash embedder id used when ONNX is unavailable / disabled.



## bookclerk_discover::embed::OnnxEmbedder

*Struct*

ONNX MiniLM embedder (fastembed). Loaded on demand; drop to free RAM.

**Methods:**

- `fn open(cache_dir: &Path, intra_threads: usize) -> std::result::Result<Self, DiscoverError>` - Download (if needed) and load quantized MiniLM into `cache_dir`.

**Trait Implementations:**

- **Embedder**
  - `fn model_id(self: &Self) -> &str`
  - `fn dimensions(self: &Self) -> usize`
  - `fn embed(self: & mut Self, texts: &[String]) -> std::result::Result<Vec<Vec<f32>>, DiscoverError>`



## bookclerk_discover::embed::default_embedding_model_id

*Function*

Preferred model id for this build (ONNX MiniLM; runtime may fall back).

```rust
fn default_embedding_model_id() -> &'static str
```



## bookclerk_discover::embed::embed_dirty_works

*Function*

Embed works whose text hash changed (or are missing).

# Arguments

* `library` - Open library store used for reads/writes.
* `embedder` - Embedding backend that produces dense vectors.

# Returns

On success, the inner `usize` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn embed_dirty_works(library: &bookclerk_library::LibraryStore, embedder: & mut dyn Embedder) -> std::result::Result<usize, DiscoverError>
```



## bookclerk_discover::embed::embedding_model_id

*Function*

Resolve the configured model id.

# Arguments

* `configured` - Configured model id from config/env; empty selects the build default.

# Returns

String result for this operation.

```rust
fn embedding_model_id(configured: &str) -> &str
```



## bookclerk_discover::embed::open_embedder

*Function*

Open an embedder for the current build / config.

When `prefer_onnx` is true, tries quantized MiniLM via fastembed/ort and
**warns + falls back** to the local-hash embedder if load/download fails
(missing glibc symbols, offline host, corrupt cache, …).

# Arguments

* `models_dir` - Directory that holds (or will hold) embedding model files.
* `intra_threads` - ONNX intra-op thread count (clamped to at least 1).
* `prefer_onnx` - When true, try MiniLM ONNX first and fall back to hash embeddings on failure.

# Returns

On success, the inner `Box<dyn Embedder>` value.

# Errors

Returns an error when the underlying I/O, parse, network, or store operation fails.

```rust
fn open_embedder(models_dir: &std::path::Path, intra_threads: usize, prefer_onnx: bool) -> std::result::Result<Box<dyn Embedder>, DiscoverError>
```



## bookclerk_discover::embed::text_for_work

*Function*

Build the text blob embedded for a work.

# Arguments

* `work` - Library work row whose metadata is turned into embedding text.

# Returns

String result for this operation.

```rust
fn text_for_work(work: &bookclerk_library::WorkRecord) -> String
```



