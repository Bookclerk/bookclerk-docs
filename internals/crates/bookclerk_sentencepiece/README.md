# bookclerk_sentencepiece

Safe wrapper over the SentencePiece Lite tokenizer runtime.

A tokenizer turns text into the token ids a transformer model was
trained on. [SentencePiece Lite](https://github.com/google/sentencepiece/tree/master/lite)
is Google's small C++ runtime for SentencePiece models stored as
FlatBuffers (`.spm.fb`): it gives the same ids as full SentencePiece and
needs neither Protocol Buffers nor Abseil.

The C++ sources are vendored, unchanged, under
`third_party/sentencepiece-lite` at the commits its `README.md` pins, and
are compiled by this crate's build script. They are Apache-2.0; the
license texts sit beside them.

The runtime parses the model bytes and the text it is given. This crate
is meant for the jailed model plugin, which is where that parsing
belongs; a host process must not link it (see
`docs/adr/ai-models.md`).

```rust
use bookclerk_sentencepiece::Tokenizer;

let model = std::fs::read("tokenizer.spm.fb")?;
let tokenizer = Tokenizer::from_bytes(model)?;
let encoding = tokenizer.encode("A detective in Victorian London", 512)?;
assert!(encoding.ids.len() <= 512);
```

| Item | Kind | Summary |
| --- | --- | --- |
| [`Encoding`](#encoding) | struct | The token ids of one text. |
| [`Tokenizer`](#tokenizer) | struct | A loaded SentencePiece Lite model. |
| [`TokenizerError`](#tokenizererror) | enum | Why a tokenizer could not be opened or a text could not be encoded. |
| [`SpecialPiece`](#specialpiece) | enum | A piece with a fixed role that most models reserve an id for. |

## Structs

### Encoding

```rust
pub struct Encoding {
    pub ids: Vec<u32>,
    pub total: usize,
}
```

The token ids of one text.

**Fields**

- `ids`: Token ids in text order, cut to the limit [`Tokenizer::encode`](#tokenizerencode) was
  given. Each is below [`Tokenizer::vocab_size`](#tokenizervocab_size).
- `total`: How many ids the whole text encodes to. Greater than `ids.len()`
  when the text was cut.

#### Encoding::truncated

```rust
pub fn truncated(&self) -> bool
```

Whether the text had more tokens than the limit allowed.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

### Tokenizer

```rust
pub struct Tokenizer {
    /* private fields */
}
```

A loaded SentencePiece Lite model.

Holds the model bytes for as long as the runtime reads from them.

#### Tokenizer::from_bytes

```rust
pub fn from_bytes(model: Vec<u8>) -> Result<Self, TokenizerError>
```

Loads a model from the bytes of a `.spm.fb` file.

##### Arguments

* `model` - The whole file. The runtime verifies the FlatBuffer
  before using it.

##### Errors

[`TokenizerError::InvalidModel`](#tokenizererror) when the runtime refuses the bytes.

#### Tokenizer::vocab_size

```rust
pub fn vocab_size(&self) -> usize
```

Number of pieces in the model; every id is below it.

#### Tokenizer::special_id

```rust
pub fn special_id(&self, piece: SpecialPiece) -> Option<u32>
```

The id the model reserves for `piece`, or `None` when it has none.

#### Tokenizer::piece_to_id

```rust
pub fn piece_to_id(&self, piece: &str) -> Option<u32>
```

The id of the piece whose text is exactly `piece`, or `None` when
the model has no such piece.

A piece's text is what the model stores: a word-initial piece starts
with `\u{2581}` (`▁`) and a byte piece reads `<0x41>`.

#### Tokenizer::id_to_piece

```rust
pub fn id_to_piece(&self, id: u32) -> Option<&str>
```

The text of piece `id`, or `None` when `id` is not below
[`Tokenizer::vocab_size`](#tokenizervocab_size) or the model stores text that is not
UTF-8.

#### Tokenizer::encode

```rust
pub fn encode(&self, text: &str, limit: usize) -> Result<Encoding, TokenizerError>
```

Encodes `text` into token ids, keeping at most `limit` of them.

The ids are the model's own: no beginning or end marker is added.

##### Arguments

* `text` - The text to encode.
* `limit` - The most ids to return; the rest of the text is dropped
  and [`Encoding::total`](#encoding) says how many there were.

##### Errors

[`TokenizerError::EncodeFailed`](#tokenizererror) when the runtime fails, and
[`TokenizerError::IdOutOfRange`](#tokenizererror) when it returns an id outside the
vocabulary (a model that disagrees with itself).

Implements: `Debug`, `Drop`, `Send`

## Enums

### TokenizerError

```rust
pub enum TokenizerError {
    InvalidModel,
    EncodeFailed,
    IdOutOfRange,
}
```

Why a tokenizer could not be opened or a text could not be encoded.

**Variants**

- `InvalidModel`: The bytes are not a SentencePiece Lite model the runtime accepts.
- `EncodeFailed`: The runtime failed to encode the text.
- `IdOutOfRange`: The runtime returned a token id outside the model's vocabulary.

Implements: `Clone`, `Copy`, `Debug`, `Display`, `Eq`, `Error`, `PartialEq`

### SpecialPiece

```rust
pub enum SpecialPiece {
    Unknown,
    Begin,
    End,
    Padding,
}
```

A piece with a fixed role that most models reserve an id for.

**Variants**

- `Unknown`: Stands in for text no piece covers.
- `Begin`: Marks the beginning of a text.
- `End`: Marks the end of a text.
- `Padding`: Fills a batch out to one length.

Implements: `Clone`, `Copy`, `Debug`, `Eq`, `Hash`, `PartialEq`
