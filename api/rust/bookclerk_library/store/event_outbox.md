**bookclerk_library > store > event_outbox**

# Module: store::event_outbox

## Contents

**Enums**

- [`EventPruneStep`](#eventprunestep) - One delete of `LibraryStore::prune_event_deliveries`, in run order.

**Functions**

- [`deliver_after_for`](#deliver_after_for) - RFC 3339 earliest delivery time for `spec`; empty when it may deliver now.
- [`inject_dispatch_page_failures`](#inject_dispatch_page_failures) - Fail after the next `n` successful dispatch pages (tests only).
- [`prepare_publish_domain_event`](#prepare_publish_domain_event) - Mint an event id when empty and reject oversized or blank publish fields.
- [`set_dispatch_chunk_for_test`](#set_dispatch_chunk_for_test) - Override subscribers-per-plan for dispatch paging tests.

---

## bookclerk_library::store::event_outbox::EventPruneStep

*Enum*

One delete of `LibraryStore::prune_event_deliveries`, in run order.

**Variants:**
- `TerminalDeliveries` - Acked and rejected deliveries past `retention_days`.
- `DeadLetters` - Dead letters past `dead_letter_retention_days`.
- `StaleSubscriberNodes` - Per-node subscriber heartbeats past their TTL.
- `EmptyEvents` - Dispatched events past `retention_days` with no deliveries left.

**Methods:**


**Traits:** Copy, Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> EventPruneStep`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &EventPruneStep) -> bool`



## bookclerk_library::store::event_outbox::deliver_after_for

*Function*

RFC 3339 earliest delivery time for `spec`; empty when it may deliver now.

```rust
fn deliver_after_for(spec: &crate::models::PublishDomainEventSpec, now: &str) -> String
```



## bookclerk_library::store::event_outbox::inject_dispatch_page_failures

*Function*

Fail after the next `n` successful dispatch pages (tests only).

```rust
fn inject_dispatch_page_failures(n: u32)
```



## bookclerk_library::store::event_outbox::prepare_publish_domain_event

*Function*

Mint an event id when empty and reject oversized or blank publish fields.

# Errors

Returns `LibraryError::Other` when the payload exceeds 64 KiB, when
`dedup_key` is blank, when a domain event (bus row) has no `event_type`
or a queue message has one, when the queue name or content type is not
recognised, or when `source` is not empty and fails the source grammar.

```rust
fn prepare_publish_domain_event(spec: crate::models::PublishDomainEventSpec) -> crate::error::Result<crate::models::PublishDomainEventSpec>
```



## bookclerk_library::store::event_outbox::set_dispatch_chunk_for_test

*Function*

Override subscribers-per-plan for dispatch paging tests.

```rust
fn set_dispatch_chunk_for_test(chunk: Option<usize>)
```



