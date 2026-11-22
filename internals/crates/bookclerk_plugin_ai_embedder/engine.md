# bookclerk_plugin_ai_embedder::engine

Running a cut graph with ONNX Runtime.

ONNX Runtime is loaded once per process from the shared library beside
the plugin's executable ([`runtime`](#runtime)). A [`Graph`](#graph) is one model's
session; [`Graph::run`](#graphrun) takes the table rows of one text's tokens and
gives the graph's output numbers.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Graph`](#graph) | struct | One model's graph, ready to run. |
| [`EngineError`](#engineerror) | enum | Why the runtime could not be loaded or a graph not run. |
| [`runtime`](#runtime) | function | Loads ONNX Runtime from `library`, once per process. |

## Structs

### Graph

```rust
pub struct Graph {
    /* private fields */
}
```

One model's graph, ready to run.

#### Graph::load

```rust
pub fn load(
    environment: &Environment,
    bytes: &[u8],
    spec: &GraphSpec,
    tables: &[TableSpec],
    dimensions: u32,
) -> Result<Self, EngineError>
```

Loads a graph.

##### Arguments

* `environment` - The process's ONNX Runtime ([`runtime`](#runtime)).
* `bytes` - The graph file, already checked against the model
  description.
* `spec` - The graph part of the model description.
* `tables` - The model's tables.
* `dimensions` - Numbers per vector.

##### Errors

[`EngineError::Graph`](#engineerror) when ONNX Runtime refuses the file, and
[`EngineError::MissingName`](#engineerror) when it lacks a named input or output.

#### Graph::run

```rust
pub fn run(&self, rows: &[&[u8]]) -> Result<Vec<f32>, EngineError>
```

Runs the graph on one text.

##### Arguments

* `rows` - One row-pack row per token of the text, in order: every
  table's row concatenated, as a row pack stores it.

##### Returns

The text's vector before normalising: the graph's output, averaged
over the tokens when the model pools by mean.

##### Errors

[`EngineError::Rows`](#engineerror) for no rows or rows of the wrong length,
[`EngineError::Run`](#engineerror) when ONNX Runtime fails, and
[`EngineError::OutputShape`](#engineerror) when the output is not what the model
description promises.

Implements: `Debug`

## Enums

### EngineError

```rust
pub enum EngineError {
    Runtime(String),
    Graph(String),
    MissingName { kind: &'static str, name: String },
    Rows,
    Run(String),
    OutputShape { expected: usize, found: usize },
}
```

Why the runtime could not be loaded or a graph not run.

**Variants**

- `Runtime`: The ONNX Runtime shared library could not be loaded.
- `Graph`: The graph file is not a graph ONNX Runtime accepts.
- `MissingName`: The graph lacks an input or output the model description names.
  - `kind`: `input` or `output`.
  - `name`: The name the description gives.
- `Rows`: The rows passed do not fit the tables.
- `Run`: ONNX Runtime failed while running the graph.
- `OutputShape`: The output is not the shape the model description promises.
  - `expected`: Numbers the description promises for this text.
  - `found`: Numbers the graph gave.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `From<EngineError>`, `PartialEq`

## Functions

### runtime

```rust
pub fn runtime(library: &Path) -> Result<&'static Environment, EngineError>
```

Loads ONNX Runtime from `library`, once per process.

#### Arguments

* `library` - Path of the shared library. Later calls return the
  environment the first call made, whatever path they pass.

#### Errors

[`EngineError::Runtime`](#engineerror) when the library is missing or is not an ONNX
Runtime build the bindings accept; the failure is remembered.
