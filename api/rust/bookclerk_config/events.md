**bookclerk_config > events**

# Module: events

## Contents

**Structs**

- [`EventsConfig`](#eventsconfig) - `[events]` — outbox retention and local delivery-worker concurrency.

---

## bookclerk_config::events::EventsConfig

*Struct*

`[events]` — outbox retention and local delivery-worker concurrency.

**Fields:**
- `retention_days: u64` - Days to retain acked/rejected deliveries and parent events with no
- `dead_letter_retention_days: u64` - Days to retain `dead_letter` deliveries (independent of `Self::retention_days`).
- `concurrency: u32` - Local delivery workers (`[events.concurrency]`; default 1).

**Methods:**

- `fn apply_env_overrides(self: & mut Self)` - Apply `BOOKCLERK_EVENTS_*` environment overrides.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> EventsConfig`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &EventsConfig) -> bool`
- **Default**
  - `fn default() -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`



