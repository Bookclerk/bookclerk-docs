**bookclerk_config > media**

# Module: media

## Contents

**Structs**

- [`MediaConfig`](#mediaconfig) - `[media]` — decode, encode, and packaging worker pool.

---

## bookclerk_config::media::MediaConfig

*Struct*

`[media]` — decode, encode, and packaging worker pool.

**Fields:**
- `workers: usize` - Maximum codec jobs running at once. `0` derives one worker per core,
- `isolation: crate::isolation::Isolation` - How strictly workers must be confined.
- `worker_bin: Option<std::path::PathBuf>` - Explicit path to `bookclerk-media-worker`. Normally left unset, in which

**Methods:**

- `fn apply_env_overrides(self: & mut Self)` - Apply `BOOKCLERK_MEDIA_*` environment overrides.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> MediaConfig`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &MediaConfig) -> bool`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Default**
  - `fn default() -> MediaConfig`



