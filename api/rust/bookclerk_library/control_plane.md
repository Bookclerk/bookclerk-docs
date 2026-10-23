**bookclerk_library > control_plane**

# Module: control_plane

## Contents

**Structs**

- [`ConfigurationDocument`](#configurationdocument) - A parsed domain document plus its revision metadata.
- [`ControlPlaneSession`](#controlplanesession) - Identity, secret root, and the authoritative events document for this process.
- [`EventsSettingsV1`](#eventssettingsv1) - Typed `[events]` document, schema version 1.
- [`NodeRuntimeSettingsV1`](#noderuntimesettingsv1) - Typed node-scoped runtime document, schema version 1.

**Enums**

- [`EventsReplace`](#eventsreplace) - Outcome of [`replace_events`](control_plane.md#bookclerk_librarycontrol_planereplace_events).
- [`NodeRuntimeReplace`](#noderuntimereplace) - Outcome of [`replace_node_runtime`](control_plane.md#bookclerk_librarycontrol_planereplace_node_runtime).

**Functions**

- [`bootstrap_control_plane`](#bootstrap_control_plane) - Enrolls this files directory and imports `[events]` and `[output]` when
- [`import_events_if_absent`](#import_events_if_absent) - Inserts `[events]` when the cluster document is absent.
- [`import_node_runtime_if_absent`](#import_node_runtime_if_absent) - Inserts a node runtime document when it is absent.
- [`load_events`](#load_events) - Loads the cluster events document.
- [`load_node_runtime`](#load_node_runtime) - Loads one node's runtime document.
- [`overlay_core_documents`](#overlay_core_documents) - Copies every authoritative cluster document in `session` onto `config`.
- [`overlay_events`](#overlay_events) - Copies the authoritative events document onto `config` for `cluster_id`.
- [`replace_events`](#replace_events) - Replaces the cluster events document.
- [`replace_node_runtime`](#replace_node_runtime) - Replaces one node's runtime document.
- [`update_node_runtime`](#update_node_runtime) - Applies `edit` to a node's runtime document (creating it with defaults

**Constants**

- [`DEFAULT_CPU_BUDGET_PERCENT`](#default_cpu_budget_percent) - Default CPU budget: leave a fifth of the cores to the rest of the node.
- [`DEFAULT_MEMORY_BUDGET_PERCENT`](#default_memory_budget_percent) - Default memory budget: leave a quarter of the memory to the rest of the node.
- [`EVENTS_NAMESPACE`](#events_namespace) - Namespace for the migrated `[events]` domain.
- [`EVENTS_SCHEMA_VERSION`](#events_schema_version) - Schema version of [`EventsSettingsV1`](control_plane.md#bookclerk_librarycontrol_planeeventssettingsv1).
- [`MAX_EVENTS_CONCURRENCY`](#max_events_concurrency) - Upper bound for `[events].concurrency`.
- [`MAX_EVENTS_RETENTION_DAYS`](#max_events_retention_days) - Upper bound for event retention fields, in days.
- [`NODE_RUNTIME_NAMESPACE`](#node_runtime_namespace) - Namespace for node-scoped runtime settings.
- [`NODE_RUNTIME_SCHEMA_VERSION`](#node_runtime_schema_version) - Schema version of [`NodeRuntimeSettingsV1`](control_plane.md#bookclerk_librarycontrol_planenoderuntimesettingsv1).

---

## bookclerk_library::control_plane::ConfigurationDocument

*Struct*

A parsed domain document plus its revision metadata.

**Generic Parameters:**
- T

**Fields:**
- `body: T` - Parsed body.
- `revision: i64` - Compare-and-swap revision.
- `schema_version: i64` - Domain schema version.
- `updated_at: String` - RFC 3339 commit time.
- `updated_by: String` - Audit identity of the last writer.
- `scope_type: String` - Scope type (`cluster` or `node`).
- `scope_id: String` - Scope id (`singleton` or a node id).
- `namespace: String` - Domain namespace.

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &ConfigurationDocument<T>) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> ConfigurationDocument<T>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::ControlPlaneSession

*Struct*

Identity, secret root, and the authoritative events document for this process.

**Fields:**
- `node: NodeRecord` - Registered node.
- `cluster_id: String` - Cluster id from the secret root.
- `events: ConfigurationDocument<EventsSettingsV1>` - Authoritative `[events]` document.
- `output: ConfigurationDocument<OutputSettingsV1>` - Authoritative shared `[output]` document.
- `node_runtime: ConfigurationDocument<NodeRuntimeSettingsV1>` - This node's runtime document. Other nodes' documents are not included.

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> ControlPlaneSession`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerk_library::control_plane::DEFAULT_CPU_BUDGET_PERCENT

*Constant*: `u8`

Default CPU budget: leave a fifth of the cores to the rest of the node.



## bookclerk_library::control_plane::DEFAULT_MEMORY_BUDGET_PERCENT

*Constant*: `u8`

Default memory budget: leave a quarter of the memory to the rest of the node.



## bookclerk_library::control_plane::EVENTS_NAMESPACE

*Constant*: `&str`

Namespace for the migrated `[events]` domain.



## bookclerk_library::control_plane::EVENTS_SCHEMA_VERSION

*Constant*: `i64`

Schema version of [`EventsSettingsV1`](control_plane.md#bookclerk_librarycontrol_planeeventssettingsv1).



## bookclerk_library::control_plane::EventsReplace

*Enum*

Outcome of [`replace_events`](control_plane.md#bookclerk_librarycontrol_planereplace_events).

**Variants:**
- `Applied(ConfigurationDocument<EventsSettingsV1>)` - The new document is committed.
- `Conflict{ current_revision: i64 }` - `expected_revision` was not current. The stored document is unchanged.
- `Replayed{ revision: i64 }` - This operation id already committed at `revision`.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> EventsReplace`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &EventsReplace) -> bool`



## bookclerk_library::control_plane::EventsSettingsV1

*Struct*

Typed `[events]` document, schema version 1.

**Fields:**
- `retention_days: u64` - Days to keep acked and rejected deliveries.
- `dead_letter_retention_days: u64` - Days to keep dead-lettered deliveries.
- `concurrency: u32` - Local delivery workers and the cluster in-flight cap.

**Methods:**

- `fn validate(self: &Self) -> Result<()>` - Rejects values outside the domain bounds.
- `fn from_config(config: &bookclerk_config::EventsConfig) -> Result<Self>` - Copies the transitional `[events]` table, rejecting values the domain forbids.
- `fn apply_to(self: &Self, config: & mut bookclerk_config::EventsConfig)` - Writes the document onto the in-memory events table.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> EventsSettingsV1`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &EventsSettingsV1) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerk_library::control_plane::MAX_EVENTS_CONCURRENCY

*Constant*: `u32`

Upper bound for `[events].concurrency`.



## bookclerk_library::control_plane::MAX_EVENTS_RETENTION_DAYS

*Constant*: `u64`

Upper bound for event retention fields, in days.



## bookclerk_library::control_plane::NODE_RUNTIME_NAMESPACE

*Constant*: `&str`

Namespace for node-scoped runtime settings.



## bookclerk_library::control_plane::NODE_RUNTIME_SCHEMA_VERSION

*Constant*: `i64`

Schema version of [`NodeRuntimeSettingsV1`](control_plane.md#bookclerk_librarycontrol_planenoderuntimesettingsv1).



## bookclerk_library::control_plane::NodeRuntimeReplace

*Enum*

Outcome of [`replace_node_runtime`](control_plane.md#bookclerk_librarycontrol_planereplace_node_runtime).

**Variants:**
- `Applied(ConfigurationDocument<NodeRuntimeSettingsV1>)` - The new document is committed.
- `Conflict{ current_revision: i64 }` - `expected_revision` was not current.
- `Replayed{ revision: i64 }` - This operation id already committed at `revision`.

**Traits:** Eq

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> NodeRuntimeReplace`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **PartialEq**
  - `fn eq(self: &Self, other: &NodeRuntimeReplace) -> bool`



## bookclerk_library::control_plane::NodeRuntimeSettingsV1

*Struct*

Typed node-scoped runtime document, schema version 1.

Desired state for one node, editable from any node in the cluster: an
operator label, and the budget the scheduler may commit on this node.
Budgets are shares of the capacity the node detects at heartbeat
(CPU cores and memory, the tighter of the machine and its cgroup).

**Fields:**
- `label: String` - Operator label shown for this node. Empty is allowed.
- `cpu_budget_percent: u8` - Share of detected CPU cores the scheduler may commit to plugin
- `memory_budget_percent: u8` - Share of detected memory the scheduler may commit (1..=100).
- `schedulable: bool` - False drains the node: the scheduler places nothing new here and

**Methods:**

- `fn new() -> Self` - Empty label, default budgets, schedulable.
- `fn validate(self: &Self) -> Result<()>` - Rejects control characters, labels longer than 64 bytes, and budgets

**Traits:** Eq

**Trait Implementations:**

- **PartialEq**
  - `fn eq(self: &Self, other: &NodeRuntimeSettingsV1) -> bool`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Clone**
  - `fn clone(self: &Self) -> NodeRuntimeSettingsV1`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Default**
  - `fn default() -> Self`



## bookclerk_library::control_plane::bootstrap_control_plane

*Function*

Enrolls this files directory and imports `[events]` and `[output]` when
their documents are absent.

`seed` is the effective `config.toml` plus environment configuration; only
its `events` and `output` tables are read, and only for a document that
does not exist yet. After this returns, callers must treat
`ControlPlaneSession::events` and `ControlPlaneSession::output` as
authority (see [`overlay_core_documents`](control_plane.md#bookclerk_librarycontrol_planeoverlay_core_documents)). A later TOML load or
environment override must not replace them without a compare-and-swap
write.

`scratch_dir` is [`bookclerk_config::Config::download_cache_dir`]: the
heartbeat records the byte size of its `acquire` and `acquire-pdf` trees.

# Errors

Returns an error when the secret root, cluster binding, schema, or a core
document cannot be established.

```rust
fn bootstrap_control_plane(store: &crate::store::LibraryStore, files_dir: &std::path::Path, scratch_dir: &std::path::Path, password: Option<&str>, seed: &bookclerk_config::Config) -> crate::error::Result<ControlPlaneSession>
```



## bookclerk_library::control_plane::import_events_if_absent

*Function*

Inserts `[events]` when the cluster document is absent.

An existing document is parsed and returned without reading the seed, so an
obsolete or out-of-range TOML/environment value cannot fail startup.

# Errors

Returns an error when the stored document is invalid, or when the document
is absent and the seed is invalid or the actor may not import.

```rust
fn import_events_if_absent(store: &crate::store::LibraryStore, actor: &ConfigActor, seed: &bookclerk_config::EventsConfig, operation_id: &str) -> crate::error::Result<ConfigurationDocument<EventsSettingsV1>>
```



## bookclerk_library::control_plane::import_node_runtime_if_absent

*Function*

Inserts a node runtime document when it is absent.

# Errors

Returns an error when the body is invalid or the actor may not import.

```rust
fn import_node_runtime_if_absent(store: &crate::store::LibraryStore, actor: &ConfigActor, node_id: &str, body: &NodeRuntimeSettingsV1, operation_id: &str) -> crate::error::Result<ConfigurationDocument<NodeRuntimeSettingsV1>>
```



## bookclerk_library::control_plane::load_events

*Function*

Loads the cluster events document.

# Errors

Returns an error when the document is missing, unsupported, or invalid.

```rust
fn load_events(store: &crate::store::LibraryStore) -> crate::error::Result<ConfigurationDocument<EventsSettingsV1>>
```



## bookclerk_library::control_plane::load_node_runtime

*Function*

Loads one node's runtime document.

# Errors

Returns an error when the document is missing, unsupported, or invalid.

```rust
fn load_node_runtime(store: &crate::store::LibraryStore, node_id: &str) -> crate::error::Result<ConfigurationDocument<NodeRuntimeSettingsV1>>
```



## bookclerk_library::control_plane::overlay_core_documents

*Function*

Copies every authoritative cluster document in `session` onto `config`.

```rust
fn overlay_core_documents(config: & mut bookclerk_config::Config, session: &ControlPlaneSession)
```



## bookclerk_library::control_plane::overlay_events

*Function*

Copies the authoritative events document onto `config` for `cluster_id`.

Revisions are only comparable within one cluster. Callers that switch
databases pass the new cluster id explicitly.

```rust
fn overlay_events(config: & mut bookclerk_config::Config, events: &ConfigurationDocument<EventsSettingsV1>, cluster_id: &str)
```



## bookclerk_library::control_plane::replace_events

*Function*

Replaces the cluster events document.

# Errors

Returns an error when validation, authorization, or the compare-and-swap fails.

```rust
fn replace_events(store: &crate::store::LibraryStore, actor: &ConfigActor, expected_revision: i64, body: &EventsSettingsV1, operation_id: &str) -> crate::error::Result<EventsReplace>
```



## bookclerk_library::control_plane::replace_node_runtime

*Function*

Replaces one node's runtime document.

# Errors

Returns an error when validation, authorization, or the compare-and-swap fails.

```rust
fn replace_node_runtime(store: &crate::store::LibraryStore, actor: &ConfigActor, node_id: &str, expected_revision: i64, body: &NodeRuntimeSettingsV1, operation_id: &str) -> crate::error::Result<NodeRuntimeReplace>
```



## bookclerk_library::control_plane::update_node_runtime

*Function*

Applies `edit` to a node's runtime document (creating it with defaults
when the node never wrote one) under compare-and-swap, retrying a few
times on a concurrent edit. Budgets reach the scheduler with the node's
next heartbeat.

# Errors

Returns an error when the actor may not write, the edited body is
invalid, or the document keeps changing underneath.

```rust
fn update_node_runtime<impl Fn(&mut NodeRuntimeSettingsV1)>(store: &crate::store::LibraryStore, actor: &ConfigActor, node_id: &str, edit: impl Trait) -> crate::error::Result<ConfigurationDocument<NodeRuntimeSettingsV1>>
```



