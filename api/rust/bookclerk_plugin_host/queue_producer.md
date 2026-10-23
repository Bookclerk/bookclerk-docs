**bookclerk_plugin_host > queue_producer**

# Module: queue_producer

## Contents

**Structs**

- [`OutboxQueueProducer`](#outboxqueueproducer) - [`QueueProducer`](../bookclerk_plugin_abi/roles.md#bookclerk_plugin_abirolesqueueproducer) server for one binding of one invocation: validates a
- [`QueueOutbox`](#queueoutbox) - Host-owned facts a session needs before it can hand a guest its queue

---

## bookclerk_plugin_host::queue_producer::OutboxQueueProducer

*Struct*

[`QueueProducer`](../bookclerk_plugin_abi/roles.md#bookclerk_plugin_abirolesqueueproducer) server for one binding of one invocation: validates a
guest [`QueueSendMessage`](../bookclerk_plugin_abi/generated.md#bookclerk_plugin_abigeneratedqueuesendmessage) and appends it to the library outbox.

**Methods:**

- `fn queue(self: &Self) -> &str` - Queue this producer sends to.
- `fn spec(self: &Self, message: QueueSendMessage, default_delay: u32) -> std::result::Result<PublishDomainEventSpec, bookclerk_plugin_abi::PluginError>` - Validates `message` and builds the outbox row the host will insert.

**Trait Implementations:**

- **QueueProducer**
  - `fn send(self: &'life0 Self, message: QueueSendMessage) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`
  - `fn send_batch(self: &'life0 Self, messages: Vec<QueueSendMessage>, delay_seconds: u32) -> ::core::pin::Pin<Box<dyn ::core::future::Future>>`



## bookclerk_plugin_host::queue_producer::QueueOutbox

*Struct*

Host-owned facts a session needs before it can hand a guest its queue
producer bindings: the outbox store plus the granted `(binding, queue)`
pairs.

**Methods:**

- `fn new(store: LibraryStore, plugin_id: &str, manifest: &PluginManifest, grant: &PluginGrant) -> Option<Self>` - Builds the outbox hook for `plugin_id`, or `None` when no producer row
- `fn binding_names(self: &Self) -> Vec<String>` - Granted binding names in manifest order.
- `fn producers(self: &Self, invocation: &Invocation) -> Vec<(String, String, Arc<dyn QueueProducer>)>` - Producers bound to one `PluginWorker.open` invocation, shaped for

**Trait Implementations:**

- **Clone**
  - `fn clone(self: &Self) -> QueueOutbox`



