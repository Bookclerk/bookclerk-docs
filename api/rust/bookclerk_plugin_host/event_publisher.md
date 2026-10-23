**bookclerk_plugin_host > event_publisher**

# Module: event_publisher

## Contents

**Structs**

- [`EventOutbox`](#eventoutbox) - Host-owned facts a session needs before it can hand a guest an `EVENTS`
- [`OutboxEventPublisher`](#outboxeventpublisher) - [`EventPublisher`](../bookclerk_plugin_abi/roles.md#bookclerk_plugin_abiroleseventpublisher) server for one invocation: validates a guest

---

## bookclerk_plugin_host::event_publisher::EventOutbox

*Struct*

Host-owned facts a session needs before it can hand a guest an `EVENTS`
binding: the outbox store plus the producer allowlist.

**Methods:**

- `fn new<impl IntoIterator<Item = String>>(store: LibraryStore, plugin_id: &str, declared: impl Trait, granted: &BTreeSet<String>) -> Option<Self>` - Builds the outbox hook for `plugin_id`, or `None` when no producer is
- `fn publisher(self: &Self, invocation: &Invocation) -> OutboxEventPublisher` - Publisher bound to one `PluginWorker.open` invocation (account scope

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> EventOutbox`



## bookclerk_plugin_host::event_publisher::OutboxEventPublisher

*Struct*

[`EventPublisher`](../bookclerk_plugin_abi/roles.md#bookclerk_plugin_abiroleseventpublisher) server for one invocation: validates a guest
[`PluginEvent`](../bookclerk_plugin_abi/generated.md#bookclerk_plugin_abigeneratedpluginevent) and appends it to the library outbox.

**Methods:**

- `fn spec(self: &Self, event: PluginEvent) -> std::result::Result<PublishDomainEventSpec, bookclerk_plugin_abi::PluginError>` - Validates `event` and builds the outbox row the host will insert.

**Trait Implementations:**

- **EventPublisher**
  - `fn publish(self: &'life0 Self, event: PluginEvent) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



