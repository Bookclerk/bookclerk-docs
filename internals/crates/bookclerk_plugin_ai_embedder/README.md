# bookclerk_plugin_ai_embedder

Text-embedding model plugin.

The platform `embedder` guest serves text-embedding models on the `ai`
entrypoint (`docs/adr/ai-models.md`). It has no network access and no
settings: it serves every model directory it finds under `models/` in
its data directory, with the language packs present there. The host
puts verified model files in that directory and removes them.

A run goes through three modules:

1. [`text`](text.md) turns a text into the model's token ids with the whole
   vocabulary (`bookclerk-sentencepiece`).
2. [`model`](model.md) looks each token's table rows up in the row packs on disk
   (`bookclerk-model-pack`) and spells a token whose row is missing with
   smaller pieces.
3. [`engine`](engine.md) runs the cut ONNX graph on those rows with the pinned ONNX
   Runtime shared library that sits beside the plugin's executable.

The layout of a model directory is described in
`docs/internals/model-packs.md`.

| Item | Kind | Summary |
| --- | --- | --- |
| [`ID`](#id) | constant | Plugin id advertised in `describe()`; models are named `@embedder/<model id>`. |
| [`MODELS_DIR`](#models_dir) | constant | Directory under the plugin's data directory that holds one directory per model. |
| [`RUNTIME_LIBRARY`](#runtime_library) | constant | Name of the ONNX Runtime shared library beside the plugin executable on this platform (the same name `bookclerk-onnxruntime` installs). |

## Modules

- [`engine`](engine.md): Running a cut graph with ONNX Runtime.
- [`model`](model.md): A model directory: what is in it, and the model loaded from it.
- [`plugin`](plugin.md): The plugin's `ai` entrypoint over the model directories on disk.
- [`text`](text.md): From a text to the model's token ids.

## Constants

### ID

```rust
pub const ID: &str = "embedder";
```

Plugin id advertised in `describe()`; models are named `@embedder/<model id>`.

### MODELS_DIR

```rust
pub const MODELS_DIR: &str = "models";
```

Directory under the plugin's data directory that holds one directory
per model.

### RUNTIME_LIBRARY

```rust
pub const RUNTIME_LIBRARY: &str = _;
```

Name of the ONNX Runtime shared library beside the plugin executable on
this platform (the same name `bookclerk-onnxruntime` installs).
