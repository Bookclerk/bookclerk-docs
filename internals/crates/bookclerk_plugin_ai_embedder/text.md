# bookclerk_plugin_ai_embedder::text

From a text to the model's token ids.

[`TextEncoder`](#textencoder) reproduces what the model's published tokenizer does
around SentencePiece: the text prefix for the text's role, the split on
literal pieces, the token limit and the framing pieces. The ids it
gives for a text are the published model's ids.

| Item | Kind | Summary |
| --- | --- | --- |
| [`TextEncoder`](#textencoder) | struct | A model's tokenizer with the framing its description asks for. |
| [`TextError`](#texterror) | enum | Why a tokenizer could not be set up or a text not encoded. |
| [`Role`](#role) | enum | What a text is to the model; some models put a different prefix before a search query than before a document. |

## Structs

### TextEncoder

```rust
pub struct TextEncoder {
    /* private fields */
}
```

A model's tokenizer with the framing its description asks for.

#### TextEncoder::new

```rust
pub fn new(
    model: Vec<u8>,
    spec: &TokenizerSpec,
    max_tokens: u32,
) -> Result<Self, TextError>
```

Sets a tokenizer up for a model.

##### Arguments

* `model` - The tokenizer file's bytes, already checked against the
  model description.
* `spec` - The tokenizer part of the model description.
* `max_tokens` - The model's token limit, framing included.

##### Errors

When the bytes are not a tokenizer model, or the description names a
piece the vocabulary does not have.

#### TextEncoder::body_limit

```rust
pub fn body_limit(&self) -> usize
```

Most ids of the text itself, the framing not counted.

#### TextEncoder::required_ids

```rust
pub fn required_ids(&self) -> impl Iterator<Item = u32> + '_
```

Ids the framing and the fallback always need: the framing pieces,
the unknown token and the word boundary.

#### TextEncoder::body_ids

```rust
pub fn body_ids(&self, text: &str, role: Role) -> Result<Vec<u32>, TextError>
```

The ids of a text, without the framing pieces.

##### Arguments

* `text` - The text as the caller gave it.
* `role` - Which text prefix goes before it.

##### Returns

At most [`TextEncoder::body_limit`](#textencoderbody_limit) ids; a longer text is cut off.

##### Errors

When the tokenizer runtime fails on the text.

#### TextEncoder::frame

```rust
pub fn frame(&self, body: Vec<u32>) -> Vec<u32>
```

Puts the framing pieces around a text's ids.

#### TextEncoder::spell

```rust
pub fn spell(&self, id: u32) -> Vec<u32>
```

Spells a token with single-character pieces, for a token whose
table row is not loaded.

##### Arguments

* `id` - A token id the tokenizer gave.

##### Returns

One id per character of the token's piece: the word boundary for a
boundary mark, the character's own piece when the vocabulary has
one, the unknown token otherwise. An id outside the vocabulary
gives the unknown token alone.

#### TextEncoder::unknown_id

```rust
pub fn unknown_id(&self) -> u32
```

Id of the unknown token.

Implements: `Debug`

## Enums

### TextError

```rust
pub enum TextError {
    Tokenizer(TokenizerError),
    MissingPiece(String),
    NoUnknownPiece,
}
```

Why a tokenizer could not be set up or a text not encoded.

**Variants**

- `Tokenizer`: The tokenizer file is not a SentencePiece Lite model, or the
  runtime failed on a text.
- `MissingPiece`: The model description names a piece the vocabulary does not have.
- `NoUnknownPiece`: The vocabulary has no unknown-token piece to fall back on.

Implements: `Clone`, `Debug`, `Display`, `Eq`, `Error`, `From<TextError>`, `From<TokenizerError>`, `PartialEq`

### Role

```rust
pub enum Role {
    Document,
    Query,
}
```

What a text is to the model; some models put a different prefix before
a search query than before a document.

**Variants**

- `Document`: Stored content.
- `Query`: A search query matched against documents.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `PartialEq`
