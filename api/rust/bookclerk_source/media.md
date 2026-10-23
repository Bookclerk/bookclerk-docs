**bookclerk_source > media**

# Module: media

## Contents

**Functions**

- [`audio_extension`](#audio_extension) - Prefer byte sniffing, then `Content-Type`, then URL path; `default` when all fail.
- [`extension_from_bytes`](#extension_from_bytes) - Guess an audio extension from magic bytes (`infer`), with AAC/MP3 fallbacks.
- [`extension_from_content_type`](#extension_from_content_type) - Map a `Content-Type` header (parameters stripped) to an audio extension.
- [`extension_from_url`](#extension_from_url) - Guess an audio file extension from a URL path (query/fragment stripped).

---

## bookclerk_source::media::audio_extension

*Function*

Prefer byte sniffing, then `Content-Type`, then URL path; `default` when all fail.

```rust
fn audio_extension(url: &str, bytes: Option<&[u8]>, content_type: Option<&str>, default: &'static str) -> &'static str
```



## bookclerk_source::media::extension_from_bytes

*Function*

Guess an audio extension from magic bytes (`infer`), with AAC/MP3 fallbacks.

```rust
fn extension_from_bytes(bytes: &[u8]) -> Option<&'static str>
```



## bookclerk_source::media::extension_from_content_type

*Function*

Map a `Content-Type` header (parameters stripped) to an audio extension.

```rust
fn extension_from_content_type(content_type: &str) -> Option<&'static str>
```



## bookclerk_source::media::extension_from_url

*Function*

Guess an audio file extension from a URL path (query/fragment stripped).

```rust
fn extension_from_url(url: &str) -> Option<&'static str>
```



