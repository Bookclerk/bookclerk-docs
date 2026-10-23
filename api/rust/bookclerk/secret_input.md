**bookclerk > secret_input**

# Module: secret_input

## Contents

**Functions**

- [`read_secret_stdin`](#read_secret_stdin) - Reads a secret: on a terminal, prompts with `prompt` on stderr and reads

---

## bookclerk::secret_input::read_secret_stdin

*Function*

Reads a secret: on a terminal, prompts with `prompt` on stderr and reads
one line without echo; otherwise reads all of stdin.

# Arguments

* `prompt` - Text shown before the input on a terminal (for example
  `"enrollment link: "`).

# Returns

The raw text read, zeroized on drop. The caller trims it and keeps the
first line.

# Errors

Returns the read error, or the error of switching the terminal's echo
off (the secret is then not read, so it is never echoed).

```rust
fn read_secret_stdin(prompt: &str) -> io::Result<zeroize::Zeroizing<String>>
```



