**bookclerk_config > isolation**

# Module: isolation

## Contents

**Enums**

- [`Isolation`](#isolation) - What to do when a child process cannot be confined.

---

## bookclerk_config::isolation::Isolation

*Enum*

What to do when a child process cannot be confined.

Shared by every tier that runs untrusted work in a child: `[media]` for
codec workers and `[plugins]` for storefront guests. The tiers differ in what
they reach for, not in how an operator wants a missing jail handled.

**Variants:**
- `Required` - Refuse to run the work when the jail does not engage. The default: a
- `BestEffort` - Confine where the platform allows and log what did not engage. Use on
- `Off` - Run with no jail at all. Development only.

**Methods:**

- `fn as_str(self: Self) -> &'static str` - Canonical spelling, as written in `config.toml`.
- `fn parse(value: &str) -> Option<Self>` - Parse a config or environment value.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> Isolation`
- **Default**
  - `fn default() -> Isolation`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &Isolation) -> bool`



