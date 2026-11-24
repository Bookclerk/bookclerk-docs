# bookclerk_library::recovery_code

Single-use second-factor recovery codes (`docs/adr/notifications.md`
decision 8).

A patron gets [`RECOVERY_CODE_COUNT`](#recovery_code_count) codes when they set up their first
second factor, and can make a new set at any time. A code finishes a
password reset in place of the authenticator code or passkey the
account would otherwise ask for.

A code is 16 characters of Crockford base32, shown in four groups of
four (`7K3M-Q9TB-2XWD-HF6P`). The first group is the code's
*selector*: it is stored as typed and names which of the user's codes
to check, so a check runs Argon2id once rather than once per code. The
other twelve characters (60 bits) are the secret, stored only as an
Argon2id hash.

| Item | Kind | Summary |
| --- | --- | --- |
| [`RecoveryCode`](#recoverycode) | struct | One recovery code, as typed or as just made. Its `Debug` never shows the secret. |
| [`RECOVERY_CODE_COUNT`](#recovery_code_count) | constant | How many codes a set holds. |
| [`SELECTOR_CHARS`](#selector_chars) | constant | Characters in a code's selector. |
| [`SECRET_CHARS`](#secret_chars) | constant | Characters in a code's secret. |
| [`MAX_RECOVERY_CODE_INPUT`](#max_recovery_code_input) | constant | Longest text accepted as a code before it is normalized: 16 symbols plus room for separators and spaces a patron might type or paste. |

## Structs

### RecoveryCode

```rust
pub struct RecoveryCode {
    /* private fields */
}
```

One recovery code, as typed or as just made. Its `Debug` never shows
the secret.

#### RecoveryCode::generate

```rust
pub fn generate() -> Self
```

Makes a fresh code from the operating system's random source.

#### RecoveryCode::generate_set

```rust
pub fn generate_set() -> Vec<Self>
```

Makes a full set of [`RECOVERY_CODE_COUNT`](#recovery_code_count) codes whose selectors
all differ.

#### RecoveryCode::parse

```rust
pub fn parse(input: &str) -> Result<Self>
```

Reads a code as a patron typed it: case does not matter, spaces
and hyphens are ignored, and the look-alikes Crockford allows are
read as digits (`I` and `L` as `1`, `O` as `0`).

##### Errors

[`LibraryError::Invalid`](README.md#libraryerror) when the text is longer than
[`MAX_RECOVERY_CODE_INPUT`](#max_recovery_code_input) bytes or does not come to exactly 16
alphabet characters.

#### RecoveryCode::selector

```rust
pub fn selector(&self) -> &str
```

The selector, which names the stored code to check. It is not
secret on its own.

#### RecoveryCode::display

```rust
pub fn display(&self) -> Zeroizing<String>
```

The code as shown to the patron: four groups of four, hyphenated.

#### RecoveryCode::hash

```rust
pub fn hash(&self) -> Result<String>
```

Hashes the secret with Argon2id, for storage.

##### Errors

[`LibraryError::Other`](README.md#libraryerror) when hashing fails.

#### RecoveryCode::verify

```rust
pub fn verify(&self, stored_hash: &str) -> Result<bool>
```

Whether the secret matches a stored hash from [`Self::hash`](#recoverycodehash).

##### Errors

[`LibraryError::Other`](README.md#libraryerror) when the stored hash cannot be read.

Implements: `Clone`, `Debug`, `Eq`, `PartialEq`

## Constants

### RECOVERY_CODE_COUNT

```rust
pub const RECOVERY_CODE_COUNT: usize = 10;
```

How many codes a set holds.

### SELECTOR_CHARS

```rust
pub const SELECTOR_CHARS: usize = 4;
```

Characters in a code's selector.

### SECRET_CHARS

```rust
pub const SECRET_CHARS: usize = 12;
```

Characters in a code's secret.

### MAX_RECOVERY_CODE_INPUT

```rust
pub const MAX_RECOVERY_CODE_INPUT: usize = 40;
```

Longest text accepted as a code before it is normalized: 16 symbols
plus room for separators and spaces a patron might type or paste.
