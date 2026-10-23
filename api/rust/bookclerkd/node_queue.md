**bookclerkd > node_queue**

# Module: node_queue

## Contents

**Functions**

- [`config_wait`](#config_wait) - [`config_wakeup`](node_queue.md#bookclerkdnode_queueconfig_wakeup) as a [`WakeFuture`](node_queue.md#bookclerkdnode_queuewakefuture).
- [`config_wakeup`](#config_wakeup) - Waits until a configuration wakeup arrives.
- [`consumer_id`](#consumer_id) - Catalog id this node claims its own queue's deliveries under.
- [`deployment_wait`](#deployment_wait) - [`deployment_wakeup`](node_queue.md#bookclerkdnode_queuedeployment_wakeup) as a [`WakeFuture`](node_queue.md#bookclerkdnode_queuewakefuture).
- [`deployment_wakeup`](#deployment_wakeup) - Waits until a deployment wakeup arrives (a stored permit returns at once).
- [`node_id`](#node_id) - This node's id, or `None` when the identity file cannot be read.
- [`placement_wakeup`](#placement_wakeup) - Waits until a placement wakeup arrives.
- [`requirements_wakeup`](#requirements_wakeup) - Waits until a requirements-sweep wakeup arrives.
- [`run_node_delivery`](#run_node_delivery) - Handles one claimed delivery for this node's consumer: wake the matching
- [`spawn_woken_loop`](#spawn_woken_loop) - Runs `pass` now, then again after each wakeup or [`RECONCILE_BACKSTOP`](node_queue.md#bookclerkdnode_queuereconcile_backstop).
- [`upsert_catalog_row`](#upsert_catalog_row) - Heartbeats this node's consumer row: one subscription to `node.<node_id>`.
- [`wake_for`](#wake_for) - Wakes the reconcilers a message concerns. `notify_one` stores a permit,
- [`wake_requirements_sweep`](#wake_requirements_sweep) - Asks for a requirements sweep soon (after a deployment write that this

**Constants**

- [`RECONCILE_BACKSTOP`](#reconcile_backstop) - Backstop reconcile cadence when no node-queue wakeup arrives.

**Type Aliases**

- [`WakeFuture`](#wakefuture) - Boxed wakeup future for [`spawn_woken_loop`](node_queue.md#bookclerkdnode_queuespawn_woken_loop).

---

## bookclerkd::node_queue::RECONCILE_BACKSTOP

*Constant*: `std::time::Duration`

Backstop reconcile cadence when no node-queue wakeup arrives.



## bookclerkd::node_queue::WakeFuture

*Type Alias*: `std::pin::Pin<Box<dyn std::future::Future>>`

Boxed wakeup future for [`spawn_woken_loop`](node_queue.md#bookclerkdnode_queuespawn_woken_loop).



## bookclerkd::node_queue::config_wait

*Function*

[`config_wakeup`](node_queue.md#bookclerkdnode_queueconfig_wakeup) as a [`WakeFuture`](node_queue.md#bookclerkdnode_queuewakefuture).

```rust
fn config_wait() -> std::pin::Pin<Box<dyn std::future::Future>>
```



## bookclerkd::node_queue::config_wakeup

*Function*

Waits until a configuration wakeup arrives.

```rust
fn config_wakeup()
```



## bookclerkd::node_queue::consumer_id

*Function*

Catalog id this node claims its own queue's deliveries under.

```rust
fn consumer_id(files_dir: &std::path::Path) -> Option<String>
```



## bookclerkd::node_queue::deployment_wait

*Function*

[`deployment_wakeup`](node_queue.md#bookclerkdnode_queuedeployment_wakeup) as a [`WakeFuture`](node_queue.md#bookclerkdnode_queuewakefuture).

```rust
fn deployment_wait() -> std::pin::Pin<Box<dyn std::future::Future>>
```



## bookclerkd::node_queue::deployment_wakeup

*Function*

Waits until a deployment wakeup arrives (a stored permit returns at once).

```rust
fn deployment_wakeup()
```



## bookclerkd::node_queue::node_id

*Function*

This node's id, or `None` when the identity file cannot be read.

```rust
fn node_id(files_dir: &std::path::Path) -> Option<String>
```



## bookclerkd::node_queue::placement_wakeup

*Function*

Waits until a placement wakeup arrives.

```rust
fn placement_wakeup()
```



## bookclerkd::node_queue::requirements_wakeup

*Function*

Waits until a requirements-sweep wakeup arrives.

```rust
fn requirements_wakeup()
```



## bookclerkd::node_queue::run_node_delivery

*Function*

Handles one claimed delivery for this node's consumer: wake the matching
reconciler and ack. A body that is not a [`NodeMessage`](../bookclerk_library/control_plane/node_queue.md#bookclerk_librarycontrol_planenode_queuenodemessage) is dead-lettered
(a newer node may publish a type this one does not know; the backstop
pass still converges).

```rust
fn run_node_delivery(library: &bookclerk_library::LibraryStore, event: &bookclerk_library::DomainEventRecord, delivery: &bookclerk_library::EventDeliveryRecord)
```



## bookclerkd::node_queue::spawn_woken_loop

*Function*

Runs `pass` now, then again after each wakeup or [`RECONCILE_BACKSTOP`](node_queue.md#bookclerkdnode_queuereconcile_backstop).

```rust
fn spawn_woken_loop<F, Fut>(state: std::sync::Arc<crate::api::AppState>, wait: fn(...), pass: F)
```



## bookclerkd::node_queue::upsert_catalog_row

*Function*

Heartbeats this node's consumer row: one subscription to `node.<node_id>`.

```rust
fn upsert_catalog_row(library: &bookclerk_library::LibraryStore, files_dir: &std::path::Path)
```



## bookclerkd::node_queue::wake_for

*Function*

Wakes the reconcilers a message concerns. `notify_one` stores a permit,
so a wakeup that lands mid-pass runs one more pass afterwards.

```rust
fn wake_for(message: &bookclerk_library::control_plane::NodeMessage)
```



## bookclerkd::node_queue::wake_requirements_sweep

*Function*

Asks for a requirements sweep soon (after a deployment write that this
node made or learned of).

```rust
fn wake_requirements_sweep()
```



