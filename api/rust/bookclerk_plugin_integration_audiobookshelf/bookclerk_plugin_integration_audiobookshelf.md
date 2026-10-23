**bookclerk_plugin_integration_audiobookshelf**

# Module: bookclerk_plugin_integration_audiobookshelf

## Contents

**Modules**

- [`guest`](#guest) - Guest-process helpers for the external Audiobookshelf integration plugin.

**Functions**

- [`oidc_client_templates`](#oidc_client_templates) - Bookclerk-as-IdP client templates declared by this plugin (`oidcClients`).

**Constants**

- [`ID`](#id) - Integration id used in describe() and `[integrations.audiobookshelf]`.

---

## bookclerk_plugin_integration_audiobookshelf::ID

*Constant*: `&str`

Integration id used in describe() and `[integrations.audiobookshelf]`.



## Module: guest

Guest-process helpers for the external Audiobookshelf integration plugin.

Opaque config JSON (from `[integrations.audiobookshelf]` / spawn config) drives
the HTTP client. User-watch queues newly observed users for
[`guest_event_poll`](guest.md#bookclerk_plugin_integration_audiobookshelfguestguest_event_poll) — the host owns claim-ticket / portal workflows.



## bookclerk_plugin_integration_audiobookshelf::oidc_client_templates

*Function*

Bookclerk-as-IdP client templates declared by this plugin (`oidcClients`).

```rust
fn oidc_client_templates() -> Vec<bookclerk_plugin_sdk::OidcClientTemplate>
```



