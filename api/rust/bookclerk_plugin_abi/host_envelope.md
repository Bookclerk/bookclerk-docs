**bookclerk_plugin_abi > host_envelope**

# Module: host_envelope

## Contents

**Structs**

- [`AdapterExecuteRequest`](#adapterexecuterequest) - Adapter execute payload: canonical SQL, structured proofs, optional receipt.
- [`CanonicalExecuteRequest`](#canonicalexecuterequest) - Host-desugared canonical SQL. Proofs are not yet bound.
- [`GuestReceiptPersist`](#guestreceiptpersist) - Host-stamped hint for adapters to persist guest replay payload before COMMIT.
- [`UnresolvedExecuteRequest`](#unresolvedexecuterequest) - Source SQL that has not been host-canonicalized.

---

## bookclerk_plugin_abi::host_envelope::AdapterExecuteRequest

*Struct*

Adapter execute payload: canonical SQL, structured proofs, optional receipt.

This is the trust boundary. Transport and adapter execution must call
`Self::require_proofs` and must not regenerate, repair, or substitute
proofs.

**Fields:**
- `request: crate::ExecuteRequest` - Canonical execute payload (no proofs).
- `guest_receipt: GuestReceiptPersist` - Host-stamped finalize hint for guest replay.
- `proofs: Vec<crate::sql_proof::ResolvedStatement>` - One resolved proof per statement, bound to that statement's SQL.
- `isolation: crate::IsolationReq` - Isolation the adapter must realize.

**Methods:**

- `fn new(request: ExecuteRequest, guest_receipt: GuestReceiptPersist) -> Self` - Builds an adapter execute request without proofs.
- `fn with_proofs(self: Self, proofs: Vec<ResolvedStatement>) -> Self` - Stamps 1:1 proofs (including receipt wrappers).
- `fn with_isolation(self: Self, isolation: IsolationReq) -> Self` - Sets isolation (default `IsolationReq::AtomicBatch`).
- `fn require_proofs(self: &Self) -> crate::Result<()>` - Fail closed unless proofs are 1:1 and hash-bound to each statement.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> AdapterExecuteRequest`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_abi::host_envelope::CanonicalExecuteRequest

*Struct*

Host-desugared canonical SQL. Proofs are not yet bound.

The SQL strings here are the exact text that must cross the adapter
boundary and that proofs must hash.

**Fields:**
- `request: crate::ExecuteRequest` - Already-desugared canonical statements (`?` placeholders).
- `guest_receipt: GuestReceiptPersist` - Host-stamped finalize hint for guest replay.
- `isolation: crate::IsolationReq` - Isolation the adapter must realize.

**Methods:**

- `fn from_desugared(request: ExecuteRequest) -> Self` - Wraps already-desugared SQL (for example after [`crate::desugar_execute_request`](sql_desugar.md#bookclerk_plugin_abisql_desugardesugar_execute_request)).
- `fn with_guest_receipt(self: Self, guest_receipt: GuestReceiptPersist) -> Self` - Host-stamped finalize hint for guest replay.
- `fn with_isolation(self: Self, isolation: IsolationReq) -> Self` - Isolation the adapter must realize.
- `fn bind_proofs(self: Self, proofs: Vec<ResolvedStatement>) -> crate::Result<AdapterExecuteRequest>` - Binds 1:1 proofs to the canonical SQL. Fail closed on mismatch.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> CanonicalExecuteRequest`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_plugin_abi::host_envelope::GuestReceiptPersist

*Struct*

Host-stamped hint for adapters to persist guest replay payload before COMMIT.

**Fields:**
- `guest_statement_len: u32` - Guest statement count inside the receipt wrap (excluding prune/select).
- `guest_request_hash: String` - Guest `requestHash` compared on replay.

**Methods:**

- `fn is_absent(self: &Self) -> bool` - True when the host did not stamp a guest-receipt finalize hint.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> GuestReceiptPersist`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &GuestReceiptPersist) -> bool`
- **Default**
  - `fn default() -> GuestReceiptPersist`



## bookclerk_plugin_abi::host_envelope::UnresolvedExecuteRequest

*Struct*

Source SQL that has not been host-canonicalized.

Construct this from planner output or SeaORM SQL, then
`Self::canonicalize` exactly once before proof generation.

**Fields:**
- `request: crate::ExecuteRequest` - Source statements (may still need `NULLS` / `NULLIF` desugar).
- `guest_receipt: GuestReceiptPersist` - Host-stamped finalize hint for guest replay.
- `isolation: crate::IsolationReq` - Isolation the adapter must realize.

**Methods:**

- `fn new(request: ExecuteRequest) -> Self` - Wraps source SQL with default isolation and no receipt hint.
- `fn with_guest_receipt(self: Self, guest_receipt: GuestReceiptPersist) -> Self` - Host-stamped finalize hint for guest replay.
- `fn with_isolation(self: Self, isolation: IsolationReq) -> Self` - Isolation the adapter must realize.
- `fn canonicalize(self: Self) -> CanonicalExecuteRequest` - Applies Bookclerk semantic desugars exactly once.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Clone**
  - `fn clone(self: &Self) -> UnresolvedExecuteRequest`



