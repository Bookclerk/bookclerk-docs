**bookclerk_integrations > hooks**

# Module: hooks

## Contents

**Functions**

- [`emit_book_acquired`](#emit_book_acquired) - Look up the acquired book and publish `book_acquired` to the durable outbox.

---

## bookclerk_integrations::hooks::emit_book_acquired

*Function*

Look up the acquired book and publish `book_acquired` to the durable outbox.

Prefer `bookclerk_library::LibraryStore::set_acquire_status`, which publishes
in the same transaction as the acquire state change. This helper is a
fail-closed catch-up for callers that already committed the book row.

# Errors

Returns when the library lookup or outbox insert fails, including a missing
book row (so the caller cannot report success without an outbox record).

```rust
fn emit_book_acquired(library: &bookclerk_library::LibraryStore, product_or_uuid: &str, storage_key: &str) -> bookclerk_library::Result<bookclerk_library::PublishDomainEventOutcome>
```



