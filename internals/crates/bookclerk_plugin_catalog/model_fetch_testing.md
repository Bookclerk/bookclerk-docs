# bookclerk_plugin_catalog::model_fetch_testing

Test support for model downloads: a tiny model and a registry on
loopback that serves it the way the real one does (a token endpoint,
then blobs by digest with `Range` support).

Compiled for this crate's tests and, behind the `test-support`
feature, for the tests of the crates that drive downloads. Never part
of a product build.

| Item | Kind | Summary |
| --- | --- | --- |
| [`Pack`](#pack) | struct | A tiny model: its files by name, `model.json` among them. |
| [`serve`](#serve) | function | A registry on loopback that hands out the token `t0ken` and serves `blobs` to requests that carry it. `cut` truncates the first answer for the blob with that digest, to exercise a resumed download. |
| [`registry`](#registry) | function | `serve`, as the registry `offers_json` names reached on loopback. |
| [`offers_json`](#offers_json) | function | A model list that offers `pack` as `tiny`, the default. |
| [`Seen`](#seen) | type alias | What the fake registry was asked: request line and whether the request carried the token. |

## Structs

### Pack

```rust
pub struct Pack {
    pub files: BTreeMap<String, Vec<u8>>,
}
```

A tiny model: its files by name, `model.json` among them.

**Fields**

- `files`: File content by name.

#### Pack::new

```rust
pub fn new(model_id: &str) -> Self
```

A model with three small files and the description that lists
them.

##### Panics

When the description cannot be written as JSON, which a test
fixture never produces.

#### Pack::offer

```rust
pub fn offer(&self) -> ModelOffer
```

The offer that pins this model.

#### Pack::blobs

```rust
pub fn blobs(&self) -> BTreeMap<String, Vec<u8>>
```

Content by `sha256:<hex>` blob name.

## Functions

### serve

```rust
pub fn serve(blobs: BTreeMap<String, Vec<u8>>, cut: Option<String>) -> (String, Seen)
```

A registry on loopback that hands out the token `t0ken` and serves
`blobs` to requests that carry it. `cut` truncates the first answer
for the blob with that digest, to exercise a resumed download.

#### Returns

The registry's address (`http://127.0.0.1:<port>`) and what it was
asked.

#### Panics

When no loopback port can be bound.

### registry

```rust
pub fn registry(
    blobs: BTreeMap<String, Vec<u8>>,
    cut: Option<String>,
) -> (ModelRegistry, Seen)
```

[`serve`](#serve), as the registry [`offers_json`](#offers_json) names reached on loopback.

#### Panics

When no loopback port can be bound, or the fixture list stops
parsing.

### offers_json

```rust
pub fn offers_json(pack: &Pack) -> Vec<u8>
```

A model list that offers `pack` as `tiny`, the default.

#### Panics

When `pack` has no `model.json` or the list cannot be written as
JSON.

## Type aliases

### Seen

```rust
pub type Seen = Arc<Mutex<Vec<(String, bool)>>>;
```

What the fake registry was asked: request line and whether the
request carried the token.
