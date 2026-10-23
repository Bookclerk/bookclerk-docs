**bookclerk_config > naming_profile**

# Module: naming_profile

## Contents

**Structs**

- [`NamingProfileTemplates`](#namingprofiletemplates) - Concrete templates contributed by a [`NamingProfile`](naming_profile.md#bookclerk_confignaming_profilenamingprofile).
- [`ResolvedNamingTemplates`](#resolvednamingtemplates) - Resolved folder / file / chapter-file templates after applying profile

**Enums**

- [`NamingProfile`](#namingprofile) - Preset path-template profile.

---

## bookclerk_config::naming_profile::NamingProfile

*Enum*

Preset path-template profile.

Select via `output.naming_profile`. Individual `folder_template` /
`file_template` / `chapter_file_template` values override the profile when
set.

**Variants:**
- `Audiobookshelf` - [Audiobookshelf](https://audiobookshelf.org/docs/documentation/libraries/book-library/directory-structure/)
- `Classic` - Previous Libation desktop defaults (`Templates.*.DefaultTemplate`).

**Methods:**

- `fn all() -> &'static [NamingProfile]` - All known profiles (for CLI listing).
- `fn as_str(self: Self) -> &'static str` - Stable config / CLI id (`audiobookshelf`, `classic`).
- `fn description(self: Self) -> &'static str` - Short human description of the profile's layout intent.
- `fn parse(value: &str) -> Option<Self>` - Parse a profile id (case-insensitive). Accepts common aliases.
- `fn templates(self: Self) -> NamingProfileTemplates` - Built-in folder / file / chapter-file templates for this profile.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Default**
  - `fn default() -> NamingProfile`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &NamingProfile) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> NamingProfile`



## bookclerk_config::naming_profile::NamingProfileTemplates

*Struct*

Concrete templates contributed by a [`NamingProfile`](naming_profile.md#bookclerk_confignaming_profilenamingprofile).

**Fields:**
- `folder: &'static str` - Default folder-path template for this profile (Libation tag syntax).
- `file: &'static str` - Default file-stem template for this profile (no extension).
- `chapter_file: &'static str` - Default per-chapter file-stem template when splitting by chapter.

**Traits:** Copy, Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &NamingProfileTemplates) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> NamingProfileTemplates`



## bookclerk_config::naming_profile::ResolvedNamingTemplates

*Struct*

Resolved folder / file / chapter-file templates after applying profile
defaults and optional per-field overrides.

**Fields:**
- `folder: String` - Folder-path template after profile defaults and optional overrides.
- `file: String` - File-stem template after profile defaults and optional overrides.
- `chapter_file: String` - Chapter file-stem template after profile defaults and optional overrides.

**Methods:**

- `fn resolve(profile: NamingProfile, folder: Option<&str>, file: Option<&str>, chapter_file: Option<&str>) -> Self` - Resolve templates: each explicit override wins over the profile default.

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> ResolvedNamingTemplates`
- **PartialEq**
  - `fn eq(self: &Self, other: &ResolvedNamingTemplates) -> bool`



