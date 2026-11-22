# bookclerk_plugin_abi::ai

The checks the host runs on model listings, model runs and their replies.

A plugin serves models on its `ai` entrypoint ([`crate::AiModels`](README.md#aimodels)) and
any plugin with the grant calls them through the `AI` binding
([`crate::AiRunner`](README.md#airunner)). Three things cross a trust boundary, and each is
checked by sans-IO code that the fuzz targets and the host share:

- **A model plugin's listing** ([`check_models`](#check_models)): the models it serves,
  each with a task, a revision and the facts a caller needs.
- **A caller's run** on the `AI` binding ([`check_binding_run_params`](#check_binding_run_params)):
  a model name ([`AiModelName`](#aimodelname)) and an input for exactly one task. The
  host checks the same input again before a model plugin sees it
  ([`check_entrypoint_run_params`](#check_entrypoint_run_params)).
- **A model plugin's reply** to a run ([`check_run_output`](#check_run_output)): for text
  embeddings, one finite vector per input text, of the length and
  revision the model declared.

Nothing here loads or runs a model.

| Item | Kind | Summary |
| --- | --- | --- |
| [`AiError`](#aierror) | enum | Why a model listing, run or reply was refused. |
| [`AiModelName`](#aimodelname) | enum | A model as a caller names it on the `AI` binding. |
| [`task_alias`](#task_alias) | function | The name of a task in a task alias (`text-embeddings`). |
| [`is_ai_model_id`](#is_ai_model_id) | function | Whether `id` is a model id as a plugin lists it: a lowercase letter or digit, then lowercase letters, digits, `.`, `_` and `-`, short enough that the longest plugin id still fits before it in `MAX_AI_MODEL_NAME_BYTES`. |
| [`check_model`](#check_model) | function | Checks one listed model. |
| [`check_models`](#check_models) | function | Checks a model plugin's listing: at most `MAX_AI_MODELS` well-formed models, each id once. |
| [`check_input`](#check_input) | function | Checks the input of one run and says which task it is for. |
| [`check_binding_run_params`](#check_binding_run_params) | function | Checks a caller's run on the `AI` binding. |
| [`check_entrypoint_run_params`](#check_entrypoint_run_params) | function | Checks a run before a model plugin sees it on its `ai` entrypoint. |
| [`embedding_vectors`](#embedding_vectors) | function | Splits checked text-embedding output into its vectors. |
| [`pack_embedding_vectors`](#pack_embedding_vectors) | function | Packs vectors into the wire form of `AiTextEmbeddingsOutput::vectors`: little-endian 32-bit floats, concatenated. |
| [`check_run_output`](#check_run_output) | function | Checks a model plugin's reply to a run against the input it answers and the model's listing. |
| [`decode_ai_run_reply_bytes`](#decode_ai_run_reply_bytes) | function | Decodes an `AiRunReply` from an unpacked Cap'n Proto message, the form a model plugin's reply crosses the plugin boundary in. |
| [`encode_ai_run_reply_bytes`](#encode_ai_run_reply_bytes) | function | Encodes an `AiRunReply`, the inverse of `decode_ai_run_reply_bytes`. |
| [`decode_ai_run_params_bytes`](#decode_ai_run_params_bytes) | function | Decodes `AiRunParams` from an unpacked Cap'n Proto message, the form a plugin's run on the `AI` binding crosses the plugin boundary in. |
| [`encode_ai_run_params_bytes`](#encode_ai_run_params_bytes) | function | Encodes `AiRunParams`, the inverse of `decode_ai_run_params_bytes`. |
| [`decode_ai_models_reply_bytes`](#decode_ai_models_reply_bytes) | function | Decodes an `AiModelsReply` from an unpacked Cap'n Proto message, the form a model plugin's listing crosses the plugin boundary in. |
| [`encode_ai_models_reply_bytes`](#encode_ai_models_reply_bytes) | function | Encodes an `AiModelsReply`, the inverse of `decode_ai_models_reply_bytes`. |
| [`AI_ALIAS_NAMESPACE`](#ai_alias_namespace) | constant | The name space of task aliases: `@library/text-embeddings` names the model the library chose for the task. No plugin serving models may use it as its id. |
| [`MAX_AI_REVISION_BYTES`](#max_ai_revision_bytes) | constant | Longest model revision digest, in bytes. Revisions are printable ASCII without spaces. |
| [`MAX_AI_LANGUAGE_TAG_BYTES`](#max_ai_language_tag_bytes) | constant | Longest language tag on a model, in bytes (the longest well-formed BCP 47 tag). |

## Enums

### AiError

```rust
pub enum AiError {
    BadModelName,
    BadModelId,
    UnknownAlias,
    TooManyModels,
    DuplicateModel(String),
    BadRevision,
    BadDimensions,
    BadMaxTokens,
    BadLanguages,
    NoInput,
    TooManyTexts,
    BadText(usize),
    VectorCount,
    NotFinite(usize),
    BadFallbacks,
    DimensionsChanged,
    RevisionChanged,
    UnexpectedModelName,
}
```

Why a model listing, run or reply was refused.

**Variants**

- `BadModelName`: A model name is not `@<plugin id>/<model id>` or a task alias, or
  is over [`MAX_AI_MODEL_NAME_BYTES`](README.md#max_ai_model_name_bytes).
- `BadModelId`: A model id is not `[a-z0-9][a-z0-9._-]*`, or its full name would be
  over [`MAX_AI_MODEL_NAME_BYTES`](README.md#max_ai_model_name_bytes).
- `UnknownAlias`: A task alias names no known task.
- `TooManyModels`: A listing has more than [`MAX_AI_MODELS`](README.md#max_ai_models) models.
- `DuplicateModel`: A listing names the same model id twice.
- `BadRevision`: A revision is empty, too long, or holds a space or a non-printable
  byte.
- `BadDimensions`: A vector length is zero, over [`MAX_AI_EMBED_DIMENSIONS`](README.md#max_ai_embed_dimensions), or
  missing on a text-embedding model.
- `BadMaxTokens`: A text-embedding model declares no token limit, or zero.
- `BadLanguages`: A model's language list is too long, repeats a tag or holds a
  malformed one.
- `NoInput`: A run carries no input for any task.
- `TooManyTexts`: A text-embedding input has more than [`MAX_AI_EMBED_TEXTS`](README.md#max_ai_embed_texts) texts.
- `BadText`: The text at this index is empty or over
  [`MAX_AI_EMBED_TEXT_BYTES`](README.md#max_ai_embed_text_bytes).
- `VectorCount`: The reply's vector bytes are not one vector of the declared length
  per input text.
- `NotFinite`: The vector at this index holds a number that is not finite.
- `BadFallbacks`: The fallback indexes are not ascending, repeat, or point past the
  input.
- `DimensionsChanged`: The reply's vector length differs from the model's listing.
- `RevisionChanged`: The reply's revision differs from the model's listing.
- `UnexpectedModelName`: A model plugin's reply named a model; only the host does.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `From<AiError>`, `PartialEq`

### AiModelName

```rust
pub enum AiModelName {
    Model { plugin: String, model: String },
    Alias(AiTask),
}
```

A model as a caller names it on the `AI` binding.

**Variants**

- `Model`: One model of one plugin: `@<plugin id>/<model id>`.
  - `plugin`: Id of the plugin that serves the model.
  - `model`: The model's id within that plugin.
- `Alias`: The model the library chose for a task: `@library/<task>`.

#### AiModelName::parse

```rust
pub fn parse(name: &str) -> Result<Self, AiError>
```

Parses a model name.

##### Arguments

* `name` - The name as a caller wrote it.

##### Returns

The plugin and model it names, or the task it is an alias for.

##### Errors

[`AiError::BadModelName`](#aierror) when the name is malformed or too long,
and [`AiError::UnknownAlias`](#aierror) when it is `@library/` followed by
something that is not a task.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Hash`, `PartialEq`

## Functions

### task_alias

```rust
pub fn task_alias(task: AiTask) -> &'static str
```

The name of a task in a task alias (`text-embeddings`).

#### Arguments

* `task` - The task.

#### Returns

The alias segment after `@library/`.

### is_ai_model_id

```rust
pub fn is_ai_model_id(id: &str) -> bool
```

Whether `id` is a model id as a plugin lists it: a lowercase letter or
digit, then lowercase letters, digits, `.`, `_` and `-`, short enough
that the longest plugin id still fits before it in
[`MAX_AI_MODEL_NAME_BYTES`](README.md#max_ai_model_name_bytes).

#### Arguments

* `id` - The model id, without its `@<plugin id>/` prefix.

#### Returns

True when the id is well formed.

### check_model

```rust
pub fn check_model(model: &AiModel) -> Result<(), AiError>
```

Checks one listed model.

#### Arguments

* `model` - The model as the plugin listed it.

#### Errors

The first rule the model breaks.

### check_models

```rust
pub fn check_models(list: &AiModelList) -> Result<(), AiError>
```

Checks a model plugin's listing: at most [`MAX_AI_MODELS`](README.md#max_ai_models) well-formed
models, each id once.

#### Arguments

* `list` - The reply to `AiModels.models`.

#### Errors

The first rule the listing breaks.

### check_input

```rust
pub fn check_input(input: &AiInput) -> Result<AiTask, AiError>
```

Checks the input of one run and says which task it is for.

#### Arguments

* `input` - The run's input.

#### Returns

The task whose field is set.

#### Errors

[`AiError::NoInput`](#aierror) when no task's field is set, otherwise the first
rule that task's input breaks.

### check_binding_run_params

```rust
pub fn check_binding_run_params(
    params: &AiRunParams,
) -> Result<(AiModelName, AiTask), AiError>
```

Checks a caller's run on the `AI` binding.

#### Arguments

* `params` - The params of `AiRunner.run`.

#### Returns

The parsed model name and the task the input is for. An alias for
another task than the input's is refused.

#### Errors

The first rule the name or the input breaks.

### check_entrypoint_run_params

```rust
pub fn check_entrypoint_run_params(params: &AiRunParams) -> Result<AiTask, AiError>
```

Checks a run before a model plugin sees it on its `ai` entrypoint.

#### Arguments

* `params` - The params of `AiModels.run`; `model` is the plugin's own
  model id.

#### Returns

The task the input is for.

#### Errors

The first rule the model id or the input breaks.

### embedding_vectors

```rust
pub fn embedding_vectors(output: &AiTextEmbeddingsOutput) -> Vec<Vec<f32>>
```

Splits checked text-embedding output into its vectors.

#### Arguments

* `output` - Output that passed [`check_run_output`](#check_run_output).

#### Returns

One vector per input text, in input order. Empty when `dimensions` is
zero.

### pack_embedding_vectors

```rust
pub fn pack_embedding_vectors(vectors: &[Vec<f32>]) -> Vec<u8>
```

Packs vectors into the wire form of [`AiTextEmbeddingsOutput::vectors`]:
little-endian 32-bit floats, concatenated.

#### Arguments

* `vectors` - The vectors, all of one length.

#### Returns

The bytes.

### check_run_output

```rust
pub fn check_run_output(
    model: &AiModel,
    input: &AiInput,
    reply: &AiRunOk,
) -> Result<(), AiError>
```

Checks a model plugin's reply to a run against the input it answers and
the model's listing.

For text embeddings: the vector length is the one the model listed,
there is exactly one vector per input text, every number is finite, the
revision is the model's, and the fallback indexes are ascending, unique
and inside the input.

#### Arguments

* `model` - The model as its plugin listed it (checked by
  [`check_model`](#check_model)).
* `input` - The input the reply answers (checked by [`check_input`](#check_input)).
* `reply` - The success payload of `AiModels.run`.

#### Errors

The first rule the reply breaks.

### decode_ai_run_reply_bytes

```rust
pub fn decode_ai_run_reply_bytes(bytes: &[u8]) -> Result<Result<AiRunOk, PluginError>>
```

Decodes an `AiRunReply` from an unpacked Cap'n Proto message, the form
a model plugin's reply crosses the plugin boundary in.

#### Arguments

* `bytes` - The message bytes.

#### Returns

The reply: the output, or the plugin's typed error.

#### Errors

Returns the Cap'n Proto error the reader reports.

### encode_ai_run_reply_bytes

```rust
pub fn encode_ai_run_reply_bytes(reply: &Result<AiRunOk, PluginError>) -> Result<Vec<u8>>
```

Encodes an `AiRunReply`, the inverse of [`decode_ai_run_reply_bytes`](#decode_ai_run_reply_bytes).

#### Arguments

* `reply` - The reply to encode.

#### Returns

The message bytes.

#### Errors

Returns the Cap'n Proto error the builder reports.

### decode_ai_run_params_bytes

```rust
pub fn decode_ai_run_params_bytes(bytes: &[u8]) -> Result<AiRunParams>
```

Decodes `AiRunParams` from an unpacked Cap'n Proto message, the form a
plugin's run on the `AI` binding crosses the plugin boundary in.

#### Arguments

* `bytes` - The message bytes.

#### Returns

The run.

#### Errors

Returns the Cap'n Proto error the reader reports.

### encode_ai_run_params_bytes

```rust
pub fn encode_ai_run_params_bytes(params: &AiRunParams) -> Result<Vec<u8>>
```

Encodes `AiRunParams`, the inverse of [`decode_ai_run_params_bytes`](#decode_ai_run_params_bytes).

#### Arguments

* `params` - The run to encode.

#### Returns

The message bytes.

#### Errors

Returns the Cap'n Proto error the builder reports.

### decode_ai_models_reply_bytes

```rust
pub fn decode_ai_models_reply_bytes(
    bytes: &[u8],
) -> Result<Result<AiModelList, PluginError>>
```

Decodes an `AiModelsReply` from an unpacked Cap'n Proto message, the
form a model plugin's listing crosses the plugin boundary in.

#### Arguments

* `bytes` - The message bytes.

#### Returns

The listing, or the plugin's typed error.

#### Errors

Returns the Cap'n Proto error the reader reports.

### encode_ai_models_reply_bytes

```rust
pub fn encode_ai_models_reply_bytes(
    reply: &Result<AiModelList, PluginError>,
) -> Result<Vec<u8>>
```

Encodes an `AiModelsReply`, the inverse of
[`decode_ai_models_reply_bytes`](#decode_ai_models_reply_bytes).

#### Arguments

* `reply` - The listing to encode.

#### Returns

The message bytes.

#### Errors

Returns the Cap'n Proto error the builder reports.

## Constants

### AI_ALIAS_NAMESPACE

```rust
pub const AI_ALIAS_NAMESPACE: &str = "library";
```

The name space of task aliases: `@library/text-embeddings` names the
model the library chose for the task. No plugin serving models may use
it as its id.

### MAX_AI_REVISION_BYTES

```rust
pub const MAX_AI_REVISION_BYTES: usize = 128;
```

Longest model revision digest, in bytes. Revisions are printable ASCII
without spaces.

### MAX_AI_LANGUAGE_TAG_BYTES

```rust
pub const MAX_AI_LANGUAGE_TAG_BYTES: usize = 35;
```

Longest language tag on a model, in bytes (the longest well-formed
BCP 47 tag).
