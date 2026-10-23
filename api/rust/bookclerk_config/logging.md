**bookclerk_config > logging**

# Module: logging

## Contents

**Structs**

- [`LoggingHandle`](#logginghandle) - Result of installing the global subscriber.
- [`TracingOptions`](#tracingoptions) - Options for [`init_tracing_with`](logging.md#bookclerk_configlogginginit_tracing_with).

**Enums**

- [`LogFormat`](#logformat) - Output format for the stderr sink.

**Functions**

- [`init_tracing`](#init_tracing) - Convenience wrapper matching the historical CLI/daemon call site.
- [`init_tracing_with`](#init_tracing_with) - Install a global tracing subscriber (stderr + optional OS facility + diagnostics).
- [`strip_ansi_escapes`](#strip_ansi_escapes) - Remove ECMA-48/ANSI escape sequences so piped guest logs stay plain text.

---

## bookclerk_config::logging::LogFormat

*Enum*

Output format for the stderr sink.

**Variants:**
- `Text` - Human-oriented text (CLI default).
- `Json` - Structured JSON (daemon default / log aggregators).

**Traits:** Eq, Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> LogFormat`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &LogFormat) -> bool`
- **Default**
  - `fn default() -> LogFormat`



## bookclerk_config::logging::LoggingHandle

*Struct*

Result of installing the global subscriber.

Keep this value alive for the process lifetime so the stderr worker thread
stays running ([`WorkerGuard`](https://docs.rs/tracing_appender/latest/tracing_appender/non_blocking/struct.WorkerGuard.html)).

**Fields:**
- `diagnostics: crate::diagnostics::DiagnosticsHandle` - Diagnostics ring buffer / upload handle.
- `journald: bool` - Whether an OS facility sink was attached.
- `os_facility: Option<crate::journal::OsLogFacility>` - Which facility was attached (if any).



## bookclerk_config::logging::TracingOptions

*Struct*

Options for [`init_tracing_with`](logging.md#bookclerk_configlogginginit_tracing_with).

**Fields:**
- `format: LogFormat` - Stderr format (text vs JSON).
- `default_level: String` - Default `EnvFilter` directive when `BOOKCLERK_LOG` / `RUST_LOG` are unset.
- `syslog_identifier: String` - Identifier for journald / Event Log source / os_log category.
- `diagnostics: crate::settings::DiagnosticsConfig` - Opt-in crash / error-burst upload settings.
- `version: String` - Package version embedded in diagnostics uploads.
- `enable_journald: bool` - Attempt to attach an OS log facility (no-op when unavailable).

**Trait Implementations:**

- **Default**
  - `fn default() -> Self`
- **Clone**
  - `fn clone(self: &Self) -> TracingOptions`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_config::logging::init_tracing

*Function*

Convenience wrapper matching the historical CLI/daemon call site.

```rust
fn init_tracing(format: LogFormat, default_level: &str) -> LoggingHandle
```



## bookclerk_config::logging::init_tracing_with

*Function*

Install a global tracing subscriber (stderr + optional OS facility + diagnostics).

Filter precedence for stderr/OS facility: `BOOKCLERK_LOG` → `RUST_LOG` →
`opts.default_level`. The diagnostics ring buffer is **not** filtered by that
directive — it always records TRACE and above so uploads retain deep context.
Widen local verbosity with e.g. `BOOKCLERK_LOG=bookclerk=debug` when investigating.

Secrets are redacted on every sink via the public `register_secret` / `redact_str` helpers.

```rust
fn init_tracing_with(opts: TracingOptions) -> LoggingHandle
```



## bookclerk_config::logging::strip_ansi_escapes

*Function*

Remove ECMA-48/ANSI escape sequences so piped guest logs stay plain text.

`bookclerk-workerd` and other guests may emit colored `tracing_subscriber`
lines on stderr. The host re-logs those lines into JSON; leaving CSI codes
in place produces `\u001b[…` noise in `daemon.json_logs`.

```rust
fn strip_ansi_escapes(input: &str) -> std::borrow::Cow<str>
```



