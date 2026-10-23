**bookclerk_naming > engine**

# Module: engine

## Contents

**Structs**

- [`TemplatePart`](#templatepart) - A single evaluated fragment of a template.

---

## bookclerk_naming::engine::TemplatePart

*Struct*

A single evaluated fragment of a template.

**Fields:**
- `value: String` - The rendered text.
- `is_literal: bool` - `true` for literal template text, `false` for a resolved tag value.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &TemplatePart) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> TemplatePart`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



