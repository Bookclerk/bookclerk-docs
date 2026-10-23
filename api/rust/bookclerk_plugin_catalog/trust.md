**bookclerk_plugin_catalog > trust**

# Module: trust

## Contents

**Structs**

- [`TrustPolicy`](#trustpolicy) - Trust policy for plugin installation (digest-required artifacts).

---

## bookclerk_plugin_catalog::trust::TrustPolicy

*Struct*

Trust policy for plugin installation (digest-required artifacts).

`Self::allow_unverified_publisher` means the operator accepted a package
that has no independent publisher authenticity proof. Archive SHA-256 is
still required. Bookclerk does **not** verify publisher signatures.

**Fields:**
- `allow_unverified_publisher: bool` - Allow community packages without independent publisher authenticity
- `refuse_yanked: bool` - Refuse yanked versions (always true for unattended).
- `warn_on_unsigned_os: bool` - When true, warn instead of refuse on missing OS code signature.

**Methods:**

- `fn allow_unverified_publisher() -> Self` - Interactive default: allow community packages after an explicit flag.
- `fn check_unverified_publisher_allowed(self: &Self) -> Result<()>` - Returns `Ok(())` when community packages without publisher authenticity

**Traits:** Eq

**Trait Implementations:**

- **Default**
  - `fn default() -> Self`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> TrustPolicy`
- **PartialEq**
  - `fn eq(self: &Self, other: &TrustPolicy) -> bool`



