**bookclerk_plugin_source_graphicaudio > options**

# Module: options

## Contents

**Enums**

- [`GraphicAudioAccess`](#graphicaudioaccess) - How GraphicAudio fetches owned audio (`[sources.graphicaudio] access`).
- [`GraphicAudioBitrate`](#graphicaudiobitrate) - GraphicAudio device encode preference (`[sources.graphicaudio] bitrate`).
- [`GraphicAudioContainer`](#graphicaudiocontainer) - Preferred GraphicAudio ZIP SKU when `GraphicAudioAccess::Zip`.

---

## bookclerk_plugin_source_graphicaudio::options::GraphicAudioAccess

*Enum*

How GraphicAudio fetches owned audio (`[sources.graphicaudio] access`).

Default is `Self::Web` (Browser Player). ZIP downloads and Access App
device registration are opt-in — do not assume the user purchased a ZIP
SKU or wants a device slot consumed.

**Variants:**
- `Web` - Magento Browser Player (`/library`) — no ZIP attempts, no device slot.
- `Zip` - Magento ZIP download (`My Downloadable Products`) — ≤3 attempts; opt-in.
- `Device` - Access App API (`/access`) — registers/uses a device activation; opt-in.

**Methods:**

- `fn parse(raw: &str) -> Option<Self>` - Parse a config / env / CLI token (`web`, `zip`, `device`, plus aliases).
- `fn from_env() -> Option<Self>` - Env override `BOOKCLERK_GA_ACCESS` or legacy `BOOKCLERK_GA_FETCH`.

**Traits:** Copy, Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &GraphicAudioAccess) -> bool`
- **Default**
  - `fn default() -> GraphicAudioAccess`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> GraphicAudioAccess`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_graphicaudio::options::GraphicAudioBitrate

*Enum*

GraphicAudio device encode preference (`[sources.graphicaudio] bitrate`).

**Variants:**
- `Hi` - Higher bitrate / Hi URL when available.
- `Lo` - Lower bitrate / Lo URL.

**Methods:**

- `fn parse(raw: &str) -> Option<Self>` - Parse.
- `fn prefers_hi(self: Self) -> bool` - Prefers hi.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Default**
  - `fn default() -> GraphicAudioBitrate`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &GraphicAudioBitrate) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> GraphicAudioBitrate`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_source_graphicaudio::options::GraphicAudioContainer

*Enum*

Preferred GraphicAudio ZIP SKU when `GraphicAudioAccess::Zip`.

**Variants:**
- `Auto` - Prefer M4B, then MP3, then FLAC (default).
- `M4b` - M4b variant.
- `Mp3` - MP3 variant.
- `Flac` - Flac variant.

**Methods:**

- `fn parse(raw: &str) -> Option<Self>` - Parse.
- `fn format_rank(self: Self, option_label: &str) -> u8` - Rank used when sorting Magento downloadable options (lower = better).

**Traits:** Eq, Copy

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &GraphicAudioContainer) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> GraphicAudioContainer`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> GraphicAudioContainer`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



