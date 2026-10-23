**bookclerk > format_out**

# Module: format_out

## Contents

**Enums**

- [`OutputFormat`](#outputformat) - Global CLI `--format` choice for human text vs pretty JSON.

**Functions**

- [`emit`](#emit) - Print a serializable value as JSON, or run `text` for human output.
- [`err_line`](#err_line) - One operator-facing stderr line (warnings/progress, not a diagnostic log).
- [`line`](#line) - One operator-facing stdout line (command output, not a diagnostic log).

---

## bookclerk::format_out::OutputFormat

*Enum*

Global CLI `--format` choice for human text vs pretty JSON.

**Variants:**
- `Text` - Human-readable lines (default).
- `Json` - Pretty-printed JSON via [`emit`](format_out.md#bookclerkformat_outemit).

**Methods:**

- `fn is_json(self: Self) -> bool` - True when the caller asked for JSON instead of text.

**Traits:** Copy, Eq

**Trait Implementations:**

- **ValueEnum**
  - `fn value_variants<'a>() -> &'a [Self]`
  - `fn to_possible_value<'a>(self: &Self) -> ::std::option::Option<clap::builder::PossibleValue>`
- **Clone**
  - `fn clone(self: &Self) -> OutputFormat`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &OutputFormat) -> bool`
- **Default**
  - `fn default() -> OutputFormat`



## bookclerk::format_out::emit

*Function*

Print a serializable value as JSON, or run `text` for human output.

```rust
fn emit<T, impl FnOnce()>(format: OutputFormat, value: &T, text: impl Trait) -> anyhow::Result<()>
```



## bookclerk::format_out::err_line

*Function*

One operator-facing stderr line (warnings/progress, not a diagnostic log).

```rust
fn err_line<impl AsRef<str>>(text: impl Trait)
```



## bookclerk::format_out::line

*Function*

One operator-facing stdout line (command output, not a diagnostic log).

```rust
fn line<impl AsRef<str>>(text: impl Trait)
```



