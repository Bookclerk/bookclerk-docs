**bookclerk_plugin_sdk > http > redirect**

# Module: http::redirect

## Contents

**Enums**

- [`Policy`](#policy) - How many hops to follow after the initial request.

---

## bookclerk_plugin_sdk::http::redirect::Policy

*Enum*

How many hops to follow after the initial request.

**Variants:**
- `None` - Do not follow `Location`.
- `Limited(usize)` - Follow at most `n` hops.
- `StopOnQueryContains(&'static str)` - Follow at most 10 hops, but stop when the current URL's query

**Methods:**

- `fn none() -> Self` - `Policy::None`.
- `fn limited(max: usize) -> Self` - `Policy::Limited`.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Default**
  - `fn default() -> Self`
- **Clone**
  - `fn clone(self: &Self) -> Policy`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &Policy) -> bool`



