**bookclerk_sandbox > spawn_diag**

# Module: spawn_diag

## Contents

**Functions**

- [`join_capped_lines`](#join_capped_lines) - Join `ring` and cap the copy at `total_bytes`.
- [`push_capped_line`](#push_capped_line) - Push `line` onto `ring`, dropping the oldest entries until both caps hold.
- [`record_spawn_diagnostic`](#record_spawn_diagnostic) - Remember one already-formatted line and emit it.
- [`redact_capped`](#redact_capped) - Redact secrets and keep at most `max_bytes` of the result.
- [`redact_diagnostic_text`](#redact_diagnostic_text) - Replace a 64-hex session challenge and the challenge env name.
- [`snapshot_spawn_diagnostics`](#snapshot_spawn_diagnostics) - Byte-capped snapshot of [`record_spawn_diagnostic`](spawn_diag.md#bookclerk_sandboxspawn_diagrecord_spawn_diagnostic) lines.
- [`spawn_diag_raw_stderr`](#spawn_diag_raw_stderr) - `true` when raw stage lines should be written to this process's stderr.
- [`spawn_diag_stderr_enabled`](#spawn_diag_stderr_enabled) - `true` when `BOOKCLERK_SPAWN_DIAG` is `1`, `true`, or `stderr`.
- [`truncate_utf8`](#truncate_utf8) - Copy `text` up to `max_bytes` without splitting a scalar value.

**Constants**

- [`SPAWN_DIAG_MAX_LINES`](#spawn_diag_max_lines) - Maximum lines in that ring. The byte cap is the budget; this only stops a
- [`SPAWN_DIAG_RECORD_BYTES`](#spawn_diag_record_bytes) - Maximum bytes stored for one diagnostic line, cut on a UTF-8 boundary.
- [`SPAWN_DIAG_TOTAL_BYTES`](#spawn_diag_total_bytes) - Maximum bytes retained in the process-wide diagnostic ring.

---

## bookclerk_sandbox::spawn_diag::SPAWN_DIAG_MAX_LINES

*Constant*: `usize`

Maximum lines in that ring. The byte cap is the budget; this only stops a
flood of tiny lines.



## bookclerk_sandbox::spawn_diag::SPAWN_DIAG_RECORD_BYTES

*Constant*: `usize`

Maximum bytes stored for one diagnostic line, cut on a UTF-8 boundary.



## bookclerk_sandbox::spawn_diag::SPAWN_DIAG_TOTAL_BYTES

*Constant*: `usize`

Maximum bytes retained in the process-wide diagnostic ring.



## bookclerk_sandbox::spawn_diag::join_capped_lines

*Function*

Join `ring` and cap the copy at `total_bytes`.

```rust
fn join_capped_lines(ring: &std::collections::VecDeque<String>, total_bytes: usize) -> String
```



## bookclerk_sandbox::spawn_diag::push_capped_line

*Function*

Push `line` onto `ring`, dropping the oldest entries until both caps hold.

```rust
fn push_capped_line(ring: & mut std::collections::VecDeque<String>, line: &str, record_bytes: usize, total_bytes: usize, max_lines: usize)
```



## bookclerk_sandbox::spawn_diag::record_spawn_diagnostic

*Function*

Remember one already-formatted line and emit it.

The ring and the tracing event are capped. Raw stderr follows
[`spawn_diag_raw_stderr`](spawn_diag.md#bookclerk_sandboxspawn_diagspawn_diag_raw_stderr).

```rust
fn record_spawn_diagnostic(message: &str)
```



## bookclerk_sandbox::spawn_diag::redact_capped

*Function*

Redact secrets and keep at most `max_bytes` of the result.

A 64-hex challenge or `BOOKCLERK_SESSION_CHALLENGE` that starts inside the cap and
continues past it is classified with a bounded lookahead, then replaced.
No prefix of that token is emitted. Bytes after the cap are not copied.
The cut stays on a UTF-8 boundary.

```rust
fn redact_capped(message: &str, max_bytes: usize) -> String
```



## bookclerk_sandbox::spawn_diag::redact_diagnostic_text

*Function*

Replace a 64-hex session challenge and the challenge env name.

Non-hex bytes are copied as Unicode scalar values. A multibyte character
is never widened by casting each byte to `char`. Diagnostic retention uses
[`redact_capped`](spawn_diag.md#bookclerk_sandboxspawn_diagredact_capped) so a token that crosses the byte cap is recognized before
any of its prefix is stored.

```rust
fn redact_diagnostic_text(message: &str) -> String
```



## bookclerk_sandbox::spawn_diag::snapshot_spawn_diagnostics

*Function*

Byte-capped snapshot of [`record_spawn_diagnostic`](spawn_diag.md#bookclerk_sandboxspawn_diagrecord_spawn_diagnostic) lines.

```rust
fn snapshot_spawn_diagnostics() -> String
```



## bookclerk_sandbox::spawn_diag::spawn_diag_raw_stderr

*Function*

`true` when raw stage lines should be written to this process's stderr.

Diagnostic mode is explicit. A jailed child also prints, because its
stderr is the host's capture pipe rather than the daemon log. The host
forwarder re-emits those lines as tracing and does not treat them as a
second daemon sink.

```rust
fn spawn_diag_raw_stderr() -> bool
```



## bookclerk_sandbox::spawn_diag::spawn_diag_stderr_enabled

*Function*

`true` when `BOOKCLERK_SPAWN_DIAG` is `1`, `true`, or `stderr`.

```rust
fn spawn_diag_stderr_enabled() -> bool
```



## bookclerk_sandbox::spawn_diag::truncate_utf8

*Function*

Copy `text` up to `max_bytes` without splitting a scalar value.

A 4-byte cap of `café` is `caf`. The truncated text never reinterprets the
leftover bytes of `é` as Latin-1.

```rust
fn truncate_utf8(text: &str, max_bytes: usize) -> &str
```



