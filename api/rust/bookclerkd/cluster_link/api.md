**bookclerkd > cluster_link > api**

# Module: cluster_link::api

## Contents

**Structs**

- [`ApproveBody`](#approvebody) - `POST /api/config/cluster-consents/approve` body.
- [`ClusterJobsBody`](#clusterjobsbody) - `POST /api/config/clusters/{cluster_id}/jobs` body.
- [`ClusterScanBody`](#clusterscanbody) - `POST /api/config/clusters/{cluster_id}/scan` body.
- [`RevokeBody`](#revokebody) - `POST /api/config/cluster-consents/revoke` body.

**Functions**

- [`get_cluster_consents`](#get_cluster_consents) - `GET /api/config/cluster-consents`.
- [`post_cluster_consent_approve`](#post_cluster_consent_approve) - `POST /api/config/cluster-consents/approve`.
- [`post_cluster_consent_revoke`](#post_cluster_consent_revoke) - `POST /api/config/cluster-consents/revoke`.
- [`post_cluster_jobs`](#post_cluster_jobs) - `POST /api/config/clusters/{cluster_id}/jobs`: the owner's view of jobs
- [`post_cluster_scan`](#post_cluster_scan) - `POST /api/config/clusters/{cluster_id}/scan`: asks the owner cluster

---

## bookclerkd::cluster_link::api::ApproveBody

*Struct*

`POST /api/config/cluster-consents/approve` body.

**Fields:**
- `cluster_id: String` - Owner cluster id.
- `plugin_key: String` - Canonical plugin key.
- `request_digest: String` - The `request_digest` of the request the operator reviewed, as the
- `deny_domains: std::collections::BTreeSet<String>` - Domains to take out of the request.
- `deny_bindings: std::collections::BTreeSet<String>` - Bindings to take out of the request.
- `deny_network: bool` - Deny all network access.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::cluster_link::api::ClusterJobsBody

*Struct*

`POST /api/config/clusters/{cluster_id}/jobs` body.

**Fields:**
- `job_ids: Vec<String>` - Job ids on the owner cluster.

**Trait Implementations:**

- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`



## bookclerkd::cluster_link::api::ClusterScanBody

*Struct*

`POST /api/config/clusters/{cluster_id}/scan` body.

**Fields:**
- `account: Option<String>` - Scan only this store account of the owner cluster.
- `plugin_instance_id: Option<String>` - Pin every planned job to this owner plugin instance.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Default**
  - `fn default() -> ClusterScanBody`



## bookclerkd::cluster_link::api::RevokeBody

*Struct*

`POST /api/config/cluster-consents/revoke` body.

**Fields:**
- `cluster_id: String` - Owner cluster id.
- `plugin_key: String` - Canonical plugin key.

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`



## bookclerkd::cluster_link::api::get_cluster_consents

*Function*

`GET /api/config/cluster-consents`.

```rust
fn get_cluster_consents(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::cluster_link::api::post_cluster_consent_approve

*Function*

`POST /api/config/cluster-consents/approve`.

```rust
fn post_cluster_consent_approve(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Json<ApproveBody>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::cluster_link::api::post_cluster_consent_revoke

*Function*

`POST /api/config/cluster-consents/revoke`.

```rust
fn post_cluster_consent_revoke(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::Json<RevokeBody>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::cluster_link::api::post_cluster_jobs

*Function*

`POST /api/config/clusters/{cluster_id}/jobs`: the owner's view of jobs
this node may see (those it would be trusted to claim).

```rust
fn post_cluster_jobs(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::extract::Path<String>, __arg2: axum::Json<ClusterJobsBody>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



## bookclerkd::cluster_link::api::post_cluster_scan

*Function*

`POST /api/config/clusters/{cluster_id}/scan`: asks the owner cluster
to queue a scan for this node's owner (H-5). The owner's planner decides
what that user may scan; the answer lists the queued job ids.

```rust
fn post_cluster_scan(__arg0: axum::extract::State<std::sync::Arc<crate::api::AppState>>, __arg1: axum::extract::Path<String>, body: Option<axum::Json<ClusterScanBody>>) -> Result<axum::Json<serde_json::Value>, axum::response::Response>
```



