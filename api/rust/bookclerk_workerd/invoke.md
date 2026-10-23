**bookclerk_workerd > invoke**

# Module: invoke

## Contents

**Structs**

- [`GrantCap`](#grantcap) - A capability that exists only to be described: it is never callable from
- [`InvokeClient`](#invokeclient) - Cap'n Proto client hook whose calls are `POST /invoke` requests.

**Enums**

- [`CapDescriptor`](#capdescriptor) - One entry of the `X-Bookclerk-Caps` table. Entry `i` describes capability

**Functions**

- [`method_headers`](#method_headers) - Header values for one `(interface, method)` pair.

**Constants**

- [`MAX_INVOKE_BODY_BYTES`](#max_invoke_body_bytes) - Largest `/invoke` request or reply body.

---

## bookclerk_workerd::invoke::CapDescriptor

*Enum*

One entry of the `X-Bookclerk-Caps` table. Entry `i` describes capability
index `i` of the message beside it.

**Variants:**
- `Source{ token: String }` - `Source` → granted `GET /open`.
- `Destination{ token: String }` - `Destination` → granted `PUT /put`.
- `Progress{ token: String }` - `ProgressSink` → granted `POST /progress`.
- `Cancellation{ token: String }` - `Cancellation` → granted `GET /cancel` long poll.
- `EventPublisher{ token: String }` - `EventPublisher` → granted `POST /events/publish`.
- `GuestDatabase{ token: String, binding: String }` - `GuestDatabase` → granted `POST /db/execute`.
- `AdapterSession{ id: String }` - Isolate-served `AdapterDatabaseSession`; later calls carry `id` in

**Traits:** Eq

**Trait Implementations:**

- **Debug**
  - `fn fmt(self: &Self, f: & mut $crate::fmt::Formatter) -> $crate::fmt::Result`
- **Deserialize**
  - `fn deserialize<__D>(__deserializer: __D) -> _serde::__private229::Result<Self, <__D as >::Error>`
- **Serialize**
  - `fn serialize<__S>(self: &Self, __serializer: __S) -> _serde::__private229::Result<<__S as >::Ok, <__S as >::Error>`
- **PartialEq**
  - `fn eq(self: &Self, other: &CapDescriptor) -> bool`
- **Clone**
  - `fn clone(self: &Self) -> CapDescriptor`



## bookclerk_workerd::invoke::GrantCap

*Struct*

A capability that exists only to be described: it is never callable from
this process. Placed in a `$Params` capability table it is serialized as
its [`CapDescriptor`](invoke.md#bookclerk_workerdinvokecapdescriptor) in `X-Bookclerk-Caps`, and the isolate reaches the
host object behind it through the granted channel with the token.

**Methods:**

- `fn new(descriptor: CapDescriptor) -> Self` - Registers a new grant capability.
- `fn client<C>(descriptor: CapDescriptor) -> C` - Builds a typed Cap'n client (e.g. `source::Client`) over a new grant
- `fn descriptor(self: &Self) -> &CapDescriptor` - The descriptor this capability serializes as.

**Trait Implementations:**

- **ClientHook**
  - `fn add_ref(self: &Self) -> Box<dyn ClientHook>`
  - `fn new_call(self: &Self, interface_id: u64, method_id: u16, size_hint: Option<capnp::MessageSize>) -> Request<any_pointer::Owned, any_pointer::Owned>`
  - `fn call(self: &Self, _interface_id: u64, _method_id: u16, _params: Box<dyn ParamsHook>, _results: Box<dyn ResultsHook>) -> Promise<(), Error>`
  - `fn get_brand(self: &Self) -> usize`
  - `fn get_ptr(self: &Self) -> usize`
  - `fn get_resolved(self: &Self) -> Option<Box<dyn ClientHook>>`
  - `fn when_more_resolved(self: &Self) -> Option<Promise<Box<dyn ClientHook>, Error>>`
  - `fn when_resolved(self: &Self) -> Promise<(), Error>`
- **Clone**
  - `fn clone(self: &Self) -> GrantCap`



## bookclerk_workerd::invoke::InvokeClient

*Struct*

Cap'n Proto client hook whose calls are `POST /invoke` requests.

**Methods:**

- `fn new(http: BridgeHttp, context: Option<String>) -> Self` - A hook for calls carrying `context` (the bridge JSON of one `open`), or
- `fn with_keepalive(http: BridgeHttp, context: Option<String>, keepalive: Option<Rc<dyn std::any::Any>>) -> Self` - Like `new`, also pinning `keepalive` for as long as any
- `fn typed<C>(self: &Self) -> C` - Typed client (`content_source::Client`, `job_runner::Client`, …) over

**Trait Implementations:**

- **ClientHook**
  - `fn add_ref(self: &Self) -> Box<dyn ClientHook>`
  - `fn new_call(self: &Self, interface_id: u64, method_id: u16, _size_hint: Option<capnp::MessageSize>) -> Request<any_pointer::Owned, any_pointer::Owned>`
  - `fn call(self: &Self, interface_id: u64, method_id: u16, params: Box<dyn ParamsHook>, results: Box<dyn ResultsHook>) -> Promise<(), Error>` - Server-side entry (a vat forwarding a call to this hook): copy the
  - `fn get_brand(self: &Self) -> usize`
  - `fn get_ptr(self: &Self) -> usize`
  - `fn get_resolved(self: &Self) -> Option<Box<dyn ClientHook>>`
  - `fn when_more_resolved(self: &Self) -> Option<Promise<Box<dyn ClientHook>, Error>>`
  - `fn when_resolved(self: &Self) -> Promise<(), Error>`
- **Clone**
  - `fn clone(self: &Self) -> InvokeClient`



## bookclerk_workerd::invoke::MAX_INVOKE_BODY_BYTES

*Constant*: `u32`

Largest `/invoke` request or reply body.

Twice the scalar ceiling so an `EventConsumer.event` batch or a
`JobRunner.job` invocation with a checkpoint still fits; equal to the
isolate codec's traversal budget (`MAX_TRAVERSAL_WORDS` words in
`db-capnp.ts`), so anything larger would be rejected there anyway.



## bookclerk_workerd::invoke::method_headers

*Function*

Header values for one `(interface, method)` pair.

Returns `None` for interfaces that are not routed over `/invoke`
(`Source`, `ByteSource`, `EventPublisher`, `GuestDatabase`, …) and for
ordinals outside the schema.

```rust
fn method_headers(interface_id: u64, method_id: u16) -> Option<(&'static str, &'static str)>
```



