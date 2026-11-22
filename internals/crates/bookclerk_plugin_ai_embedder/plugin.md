# bookclerk_plugin_ai_embedder::plugin

The plugin's `ai` entrypoint over the model directories on disk.

## Structs

### EmbedderRoot

```rust
pub struct EmbedderRoot {
    /* private fields */
}
```

Root capability of the `embedder` guest.

#### EmbedderRoot::new

```rust
pub fn new(models_dir: Option<PathBuf>, library: Option<PathBuf>) -> Self
```

A plugin over explicit locations.

##### Arguments

* `models_dir` - Directory with one directory per model.
* `library` - The ONNX Runtime shared library.

#### EmbedderRoot::from_environment

```rust
pub fn from_environment() -> Self
```

The plugin as the host starts it: models under `models/` in its
data directory (its `HOME`), ONNX Runtime beside its executable.

Implements: `Clone`, `Debug`, `PluginWorker`
