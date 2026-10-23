**bookclerk_config > journal**

# Module: journal

## Contents

**Structs**

- [`OsLogLayer`](#osloglayer) - Structured sink that writes **redacted** events to the platform log facility.

**Enums**

- [`OsLogFacility`](#oslogfacility) - Which OS facility was attached (if any).

**Functions**

- [`journald_available`](#journald_available) - Back-compat alias.
- [`os_log_available`](#os_log_available) - Probe whether an OS log facility appears available.

---

## bookclerk_config::journal::OsLogFacility

*Enum*

Which OS facility was attached (if any).

**Variants:**
- `Journald` - Linux systemd journal (`/run/systemd/journal/socket`).
- `OsLog` - Apple Unified Logging (`os_log`).
- `EventLog` - Windows Event Log Application channel.

**Traits:** Eq, Copy

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> OsLogFacility`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &OsLogFacility) -> bool`



## bookclerk_config::journal::OsLogLayer

*Struct*

Structured sink that writes **redacted** events to the platform log facility.

**Methods:**

- `fn new<impl Into<String>>(syslog_identifier: impl Trait) -> io::Result<Self>` - Connect to the platform facility. Returns `Err` when unavailable.
- `fn facility(self: &Self) -> OsLogFacility` - Which platform sink this layer is writing to.

**Trait Implementations:**

- **Layer**
  - `fn on_event(self: &Self, event: &Event, _ctx: Context<S>)`



## bookclerk_config::journal::journald_available

*Function*

Back-compat alias.

```rust
fn journald_available() -> bool
```



## bookclerk_config::journal::os_log_available

*Function*

Probe whether an OS log facility appears available.

```rust
fn os_log_available() -> bool
```



